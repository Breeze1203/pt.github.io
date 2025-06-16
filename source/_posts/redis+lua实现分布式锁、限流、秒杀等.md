---
title: redis+lua实现分布式锁、限流、秒杀等
date: 2025-06-16 17:58:15
tags: redis
---

## 分布式锁

```java
package org.pt;

/**
 * @ClassName DistributedLockService
 * @Author pt
 * @Description
 * @Date 2025/6/16 15:53
 **/

import jakarta.annotation.Resource;
import org.springframework.core.io.ClassPathResource;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.scripting.support.ResourceScriptSource;
import org.springframework.stereotype.Service;
import java.util.Collections;
import java.util.UUID;
import java.util.concurrent.TimeUnit;

@Service(value = "DistributedLockService")
public class DistributedLockService {

    private static final String LOCK_KEY_PREFIX = "lock:";

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    private static final DefaultRedisScript<Long> LOCK_SCRIPT;
    private static final DefaultRedisScript<Long> UNLOCK_SCRIPT;

    static {
        // 初始化加锁脚本
        LOCK_SCRIPT = new DefaultRedisScript<>();
        LOCK_SCRIPT.setScriptSource(new ResourceScriptSource(new ClassPathResource("scripts/lock.lua")));
        LOCK_SCRIPT.setResultType(Long.class);

        // 初始化解锁脚本
        UNLOCK_SCRIPT = new DefaultRedisScript<>();
        UNLOCK_SCRIPT.setScriptSource(new ResourceScriptSource(new ClassPathResource("scripts/unlock.lua")));
        UNLOCK_SCRIPT.setResultType(Long.class);
    }



    /**
     * 尝试获取锁
     * @param resourceName 锁定的资源名称
     * @param lockValue 锁的持有者标识
     * @param expireTime 过期时间
     * @param unit 时间单位
     * @return true 如果成功获取锁, false otherwise
     */
    public boolean tryLock(String resourceName, String lockValue, long expireTime, TimeUnit unit) {
        String key = LOCK_KEY_PREFIX + resourceName;
        long expireMillis = unit.toMillis(expireTime);
        Long result = stringRedisTemplate.execute(
                LOCK_SCRIPT,
                Collections.singletonList(key),
                lockValue,
                String.valueOf(expireMillis)
        );

        return result != null && result == 1L;
    }

    /**
     * 释放锁
     * @param resourceName 锁定的资源名称
     * @param lockValue 锁的持有者标识 (必须与加锁时相同)
     */
    public void unlock(String resourceName, String lockValue) {
        String key = LOCK_KEY_PREFIX + resourceName;
        stringRedisTemplate.execute(UNLOCK_SCRIPT, Collections.singletonList(key), lockValue);
    }

    public void processOrder(String orderId) {
        String lockValue = UUID.randomUUID().toString();
        // 尝试获取订单锁，最长等待30秒
        if (tryLock("order:" + orderId, lockValue, 30, TimeUnit.SECONDS)) {
            try {
                System.out.println("成功获取锁，开始处理订单：" + orderId);
                Thread.sleep(500); // 模拟业务处理
                System.out.println("订单处理完成：" + orderId);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                // 确保释放锁
                unlock("order:" + orderId, lockValue);
                System.out.println("释放锁：" + orderId);
            }
        } else {
            System.out.println("获取锁失败，请稍后重试：" + orderId);
        }
    }
}

```

```lua
-- KEYS[1]: 锁的 key
-- ARGV[1]: 锁的 value (通常是唯一的请求ID或线程ID)
-- ARGV[2]: 锁的过期时间（毫秒）

-- 尝试获取锁，使用 set 命令的 NX 和 PX 选项
-- 如果 key 不存在(NX)，则设置 key 和 value，并设置过期时间(PX)
local result = redis.call('set', KEYS[1], ARGV[1], 'NX', 'PX', ARGV[2])

if result then
    return 1
else
    return 0
end
```

```lua
-- KEYS[1]: 锁的 key
-- ARGV[1]: 锁的 value (用于验证是否是自己的锁)

-- 先获取锁的 value
local lockValue = redis.call('get', KEYS[1])
-- 检查锁是否存在，并且 value 与期望的 value 是否一致
if lockValue == ARGV[1] then
    -- 如果是自己的锁，则删除，释放锁
    return redis.call('del', KEYS[1])
else
    -- 不是自己的锁，或者锁已不存在，不做任何操作
    return 0
end
```

## 限流

### 方式一(string)

```lua
package org.pt;

/**
 * @ClassName RateLimiterService
 * @Author pt
 * @Description
 * @Date 2025/6/16 16:40
 **/

import jakarta.annotation.Resource;
import org.springframework.core.io.ClassPathResource;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.scripting.support.ResourceScriptSource;
import org.springframework.stereotype.Service;
import java.util.Collections;

@Service
public class RateLimiterService {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    private final static DefaultRedisScript<Long> rateLimitScript;

    static {
        rateLimitScript = new DefaultRedisScript<>();
        rateLimitScript.setScriptSource(new ResourceScriptSource(new ClassPathResource("scripts/ratelimit.lua")));
        rateLimitScript.setResultType(Long.class);
    }

    /**
     * 检查某个操作是否被允许
     * @param key 限流的唯一标识
     * @param windowSeconds 时间窗口（秒）
     * @param maxRequests 最大请求数
     * @return true 如果允许, false 如果被限流
     */
    public boolean isAllowed(String key, int windowSeconds, int maxRequests) {
        Long result = stringRedisTemplate.execute(
                rateLimitScript,
                Collections.singletonList(key),
                String.valueOf(windowSeconds),
                String.valueOf(maxRequests)
        );
        return result != null && result == 1L;
    }

    public void handleApiRequest(String userId) {
        String key = "ratelimit:api:user:" + userId;
        // 每60秒内，只允许用户访问10次
        if (isAllowed(key, 60, 10)) {
            System.out.println("用户 " + userId + " 访问成功。");
        } else {
            System.out.println("用户 " + userId + " 访问过于频繁，已被限流。");
        }
    }
}

```

```lua
-- KEYS[1]: 限流的 key (例如: ratelimit:user:123)
-- ARGV[1]: 时间窗口（秒）
-- ARGV[2]: 窗口内的最大请求数

local current_requests = tonumber(redis.call('get', KEYS[1]) or "0")

if current_requests < tonumber(ARGV[2]) then
    -- 未达到阈值，计数器加1
    local new_val = redis.call('incr', KEYS[1])
    -- 如果是第一次设置，需要设置过期时间
    if new_val == 1 then
        redis.call('expire', KEYS[1], ARGV[1])
    end
    return 1 -- 允许访问
else
    -- 已达到阈值
    return 0 -- 拒绝访问
end
```

### 方式二(sorted set)

```java
package org.pt;

/**
 * @ClassName SlidingWindowRateLimiterService
 * @Author pt
 * @Description
 * @Date 2025/6/16 16:58
 **/
import jakarta.annotation.Resource;
import org.springframework.core.io.ClassPathResource;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.scripting.support.ResourceScriptSource;
import org.springframework.stereotype.Service;
import java.util.Collections;

/**
 * 基于 Redis 有序集合 (Sorted Set) 实现的滑动窗口限流服务。
 */
@Service
public class SlidingWindowRateLimiterService {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    private final static DefaultRedisScript<Long> slidingWindowScript;

    static  {
        slidingWindowScript = new DefaultRedisScript<>();
        slidingWindowScript.setResultType(Long.class);
        slidingWindowScript.setScriptSource(new ResourceScriptSource(new ClassPathResource("scripts/sliding_window.lua")));
    }

    /**
     * 检查某个操作在滑动窗口内是否被允许。
     *
     * @param key           要限流的资源的唯一键 (e.g., "ratelimit:user:pengtao")
     * @param windowSeconds 时间窗口的大小，单位为秒 (e.g., 10)
     * @param maxRequests   在时间窗口内允许的最大请求数 (e.g., 3)
     * @return true 如果请求被允许, false 如果请求被拒绝
     */
    public boolean isAllowed(String key, int windowSeconds, int maxRequests) {
        // 调用 Redis 执行 Lua 脚本
        Long result = stringRedisTemplate.execute(
                slidingWindowScript,
                Collections.singletonList(key),   // KEYS[1]
                String.valueOf(windowSeconds),    // ARGV[1]
                String.valueOf(maxRequests)       // ARGV[2]
        );

        // 如果脚本返回 1，则表示允许
        return result != null && result == 1L;
    }

    /**
     * 使用示例：模拟一个需要限流的 API 请求。
     * @param userId 用户ID
     */
    public void handleApiRequest(String userId) {
        String key = "ratelimit:api:sliding:" + userId;
        // 设置规则：每 10 秒最多允许 3 次请求
        if (isAllowed(key, 10, 3)) {
            System.out.println("用户 " + userId + " 的请求被允许。");
            // 在这里执行核心业务逻辑...
        } else {
            System.out.println("用户 " + userId + " 的请求被拒绝，访问过于频繁！");
            // 在这里可以抛出异常或返回错误信息
        }
    }
}

```

```lua
--[[
  基于 Redis 有序集合实现的滑动窗口限流器。

  KEYS[1]: 需要被限流的资源的唯一键 (例如: "rate_limit:user:123")。

  ARGV[1]: 时间窗口的大小，单位为秒 (例如: 60)。
  ARGV[2]: 在时间窗口内允许的最大请求数 (例如: 100)。
--]]
-- 从 Redis 服务器获取当前时间。TIME 命令返回一个表: {秒, 微秒}。
local now = redis.call('TIME')
local current_seconds = tonumber(now[1])
local current_microseconds = tonumber(now[2])

-- 将秒和微秒组合成一个高精度的时间戳分数。
-- 这个值也将作为本次请求的唯一成员。
local current_timestamp_score = current_seconds * 1000000 + current_microseconds
-- 从参数中获取窗口大小和最大请求数。
local window_size_seconds = tonumber(ARGV[1])
local max_requests = tonumber(ARGV[2])
-- 计算出有效窗口的最小分数（即窗口的起始时间）。
-- 任何分数小于此值的请求都被认为是“过期的”，将会被移除。
local window_start_score = current_timestamp_score - (window_size_seconds * 1000000)
-- 1. 清理旧记录: 原子性地从有序集合中移除所有过期的成员。
--    这些都是在当前滑动窗口开始之前发生的请求。
redis.call('ZREMRANGEBYSCORE', KEYS[1], '-inf', window_start_score)
-- 2. 统计数量: 获取当前窗口内的请求总数。
local request_count = redis.call('ZCARD', KEYS[1])

-- 3. 检查阈值: 将当前计数与允许的最大请求数进行比较。
if request_count < max_requests then
  -- 未达到限流阈值。
  -- 4. 添加新请求: 将当前请求的时间戳添加到有序集合中。
  --    我们使用高精度的时间戳同时作为分数(score)和成员(member)。
  redis.call('ZADD', KEYS[1], current_timestamp_score, current_timestamp_score)

  -- 5. 设置过期时间: 给这个 Key 本身设置一个过期时间。这是一个良好实践，
  --    用于自动清理非活跃用户的键，以节省内存。
  --    过期时间设置为窗口大小。
  redis.call('EXPIRE', KEYS[1], window_size_seconds)

  -- 返回 1 表示请求被允许。
  return 1
else
  -- 已经达到限流阈值。
  -- 返回 0 表示请求应该被拒绝。
  return 0
end
```

## 秒杀

```java
package org.pt;

/**
 * @ClassName SeckillService
 * @Author pt
 * @Description
 * @Date 2025/6/16 17:22
 **/
import jakarta.annotation.Resource;
import org.springframework.core.io.ClassPathResource;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.scripting.support.ResourceScriptSource;
import org.springframework.stereotype.Service;
import java.util.Collections;

@Service
public class SeckillService {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    private final static DefaultRedisScript<Long> stockDeductScript;

    static  {
        stockDeductScript = new DefaultRedisScript<>();
        stockDeductScript.setScriptSource(new ResourceScriptSource(new ClassPathResource("scripts/stock_deduct.lua")));
        stockDeductScript.setResultType(Long.class);
    }

    /**
     * 扣减库存
     * @param productId 商品ID
     * @return true 如果扣减成功, false otherwise
     */
    public boolean deductStock(String productId) {
        String key = "stock:product:" + productId;
        // 每次扣减 1 个库存
        Long result = stringRedisTemplate.execute(
                stockDeductScript,
                Collections.singletonList(key),
                "1"
        );
        if (result == null) {
            return false;
        }
        if (result >= 0) {
            System.out.println("商品 " + productId + " 库存扣减成功，剩余库存：" + result);
            return true;
        } else if (result == -1) {
            System.out.println("商品 " + productId + " 库存不足！");
            return false;
        } else { // result == -2
            System.out.println("商品 " + productId + " 不存在或已售罄！");
            return false;
        }
    }

    // 初始化库存以供测试
    public void setStock(String productId, int stock) {
        String key = "stock:product:" + productId;
        stringRedisTemplate.opsForValue().set(key, String.valueOf(stock));
    }
}

```

```lua
-- scripts/stock_deduct.lua
-- KEYS[1]: 商品库存的 key (例如: stock:product:1001)
-- ARGV[1]: 本次要扣减的数量 (通常是 1)

local stock = tonumber(redis.call('get', KEYS[1]))
-- 检查库存是否存在且大于0
if stock and stock > 0 then
    -- 检查库存是否足够本次扣减
    local requested_amount = tonumber(ARGV[1])
    if stock >= requested_amount then
        -- 库存充足，执行扣减操作
        return redis.call('decrby', KEYS[1], requested_amount)
    else
        -- 库存不足
        return -1
    end
else
    -- 库存不存在或已售罄
    return -2
end
```


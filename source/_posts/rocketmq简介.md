---
title: "RocketMQ简介"
date: 2024-11-18 15:41:52
---

### RocketMQ介绍

#### 为什么要用MQ?

MQ(Message Queue)消息队列，是基础数据结构中"先讲先出"的一种数据结构，在消息的传输过程中保存消息的容器，生产者和消费者不能直接通讯，依靠队列保证消息的可靠性，避免了系统间的相互影响；

#### <img src="/upload/截屏2024-05-17%2010.35.09.png" style="display: inline-block;width:100.0%;height:100.0%" />应用场景

##### 1.异步解耦

##### <img src="/upload/截屏2024-05-17%2010.37.03.png" style="display: inline-block;width:100.0%;height:100.0%" />2.削峰填谷

##### <img src="/upload/截屏2024-05-17%2010.38.16.png" style="display: inline-block;width:100.0%;height:100.0%" alt="截屏2024-05-17 10.38.16.png" />3.其它

- 顺序收发

- 分布式事务一致性

- 大数据分析

- 分布式缓存同步

##### 4.MQ的缺点

- 不能完全代替RPC

- 系统可用性降低

- 系统的复杂度提高

- 一致性问题

##### 5.提到了RPC，什么是RPC?

RPC（Remote Procedure Call，远程过程调用）是一种计算机通信协议，<span color="rgb(220, 38, 38)" fontsize="" style="color: rgb(220, 38, 38)">它允许程序在不同的计算机或进程之间通过网络调用另一个程序的子程序或函数，就像调用本地函数一样</span>。RPC 的基本原理是客户端调用远程服务器上的一个函数，传递一些参数，然后服务器执行该函数并将结果返回给客户端。

RPC 的实现通常包括定义远程调用接口（接口描述语言），生成客户端和服务器端的存根（Stub）和桩（Skeleton）代码以便进行通信，以及使用一种网络传输协议来实际传输数据（如TCP或HTTP）。

RPC 的优点包括简化分布式系统的开发，使得远程调用看起来像是本地调用，提高了系统的模块化和可维护性。但是，RPC 也有一些挑战，例如需要处理网络延迟和错误，以及处理不同操作系统和编程语言之间的兼容性问题

##### 6.各种消息中间件性能对比

<span color="rgb(51, 51, 51)" fontsize="" style="color: rgb(51, 51, 51)">RocketMQ则是结合了Kafka，ActiveMQ、RabbitMQ的特性。在性能上，可以与Kafka抗衡; 而在企业级MQ的特性上，则具备了很多ActiveMQ、RabbitMQ提供的特性。因此，企业在选择消息 中间件选型时，RocketMQ是非常值得考虑的一款产品</span>

<div style="overflow-x: auto; overflow-y: hidden;">

<table style="width: 500px">
<tbody>
<tr style="height: 60px;">
<th data-colwidth="100"><p>特性</p></th>
<th data-colwidth="100"><p>ActiveMQ</p></th>
<th data-colwidth="100"><p>RabbitMQ</p></th>
<th data-colwidth="100"><p>RocketMQ</p></th>
<th data-colwidth="100"><p>Kafka</p></th>
</tr>
&#10;<tr style="height: 60px;">
<td data-colwidth="100"><p>开发语言</p></td>
<td data-colwidth="100"><p>java</p></td>
<td data-colwidth="100"><p>erlang</p></td>
<td data-colwidth="100"><p>java</p></td>
<td data-colwidth="100"><p>scala</p></td>
</tr>
<tr style="height: 60px;">
<td data-colwidth="100"><p>单机吞吐量</p></td>
<td data-colwidth="100"><p><span color="rgb(51, 51, 51)" data-fontsize="" style="color: rgb(51, 51, 51)">万级，吞 吐量比 RocketMQ 和Kafka要 低一个数 量级</span></p></td>
<td data-colwidth="100"><p><span color="rgb(51, 51, 51)" data-fontsize="" style="color: rgb(51, 51, 51)">万级，吞吐量比 RocketMQ和 Kafka要低一个 数量级</span></p></td>
<td data-colwidth="100"><p><span color="rgb(51, 51, 51)" data-fontsize="" style="color: rgb(51, 51, 51)">十万级，RocketMQ也 是可以支撑高吞吐的 一种MQ</span></p></td>
<td data-colwidth="100"><div class="columns" cols="2" style="display: flex;width: 100%;grid-gap: 8px;gap: 8px;">
<div class="column" data-index="0" style="min-width: 0;padding: 12px;flex: 1 1;box-sizing: border-box;">
<p><span color="rgb(51, 51, 51)" data-fontsize="" style="color: rgb(51, 51, 51)">十万级别，Kafka最大 优点就是吞吐量大，一 般配合大数据类的系统 来进行实时数据计算、 日志采集等场景</span></p>
</div>
</div></td>
</tr>
</tbody>
</table>

</div>

---
title: kafka生产经验
date: 2025-01-21 13:55:35
categories: kafka
tags: 
 - kafka
---

#### kafka生产经验
1. 生产经验----->生产者提高吞吐量
![截屏2025-01-21 13.56.31.png](/upload/截屏2025-01-21%2013.56.31.png)
```
Properties properties = new Properties();
// 2. 给kafka 配置对象添加配置信息：bootstrap.servers
properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,
"hadoop102:9092");
// key,value 序列化（必须）：key.serializer，value.serializer
properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
"org.apache.kafka.common.serialization.StringSerializer");
properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
"org.apache.kafka.common.serialization.StringSerializer");
// batch.size：批次大小，默认16K
properties.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
// linger.ms：等待时间，默认0
properties.put(ProducerConfig.LINGER_MS_CONFIG, 1);
// RecordAccumulator：缓冲区大小，默认32M：buffer.memory
properties.put(ProducerConfig.BUFFER_MEMORY_CONFIG,33554432);
// compression.type：压缩，默认 none，可配置值 gzip、snappy、
lz4和 zstd
properties.put(ProducerConfig.COMPRESSION_TYPE_CONFIG,"snappy");
// 3. 创建 kafka生产者对象
KafkaProducer<String, String> kafkaProducer = new
KafkaProducer<String, String>(properties);
// 4. 调用 send方法,发送消息
for (int i = 0; i < 5; i++) {
kafkaProducer.send(new
ProducerRecord<>("first","atguigu " + i));
}
// 5. 关闭资源
kafkaProducer.close();
}
}
```
2. 生产经验----->数据可靠性
- 发送流程
![截屏2025-01-21 14.00.35.png](/upload/截屏2025-01-21%2014.00.35.png)
- ACK应答级别
![截屏2025-01-21 14.01.47.png](/upload/截屏2025-01-21%2014.01.47.png)
![截屏2025-01-21 14.02.22.png](/upload/截屏2025-01-21%2014.02.22.png)
![截屏2025-01-21 14.02.55.png](/upload/截屏2025-01-21%2014.02.55.png)
```
// 1. 创建 kafka生产者的配置对象
Properties properties = new Properties();
// 2. 给kafka 配置对象添加配置信息：bootstrap.servers
properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,
"hadoop102:9092");
// key,value 序列化（必须）：key.serializer，value.serializer
properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
StringSerializer.class.getName());
properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
StringSerializer.class.getName());
// 设置acks
properties.put(ProducerConfig.ACKS_CONFIG, "all");
// 重试次数retries，默认是int最大值，2147483647
properties.put(ProducerConfig.RETRIES_CONFIG, 3);
// 3. 创建 kafka生产者对象
KafkaProducer<String, String> kafkaProducer = new
KafkaProducer<String, String>(properties);
// 4. 调用 send方法,发送消息
for (int i = 0; i < 5; i++) {
kafkaProducer.send(new
ProducerRecord<>("first","atguigu " + i));
}
// 5. 关闭资源
kafkaProducer.close();
}
}
```
3. 生产经验----->数据去重
**- 数据传输语义**
![截屏2025-01-21 14.07.15.png](/upload/截屏2025-01-21%2014.07.15.png)
**- 幂等性**
![截屏2025-01-21 14.09.22.png](/upload/截屏2025-01-21%2014.09.22.png)
![截屏2025-01-21 14.09.45.png](/upload/截屏2025-01-21%2014.09.45.png)
**- 生产者事务**
![截屏2025-01-21 14.11.20.png](/upload/截屏2025-01-21%2014.11.20.png)
4. 生产经验----->数据有序
![截屏2025-01-21 14.14.38.png](/upload/截屏2025-01-21%2014.14.38.png)
5. 生产经验----->消费者事务
![截屏2025-01-21 14.18.52.png](/upload/截屏2025-01-21%2014.18.52.png)
6. 生产经验----->数据积压（消费者如何提高吞吐量）
![截屏2025-01-21 14.20.15.png](/upload/截屏2025-01-21%2014.20.15.png)
7. 生产经验----->漏消费和重复消费
重复消费：已经消费了数据，但是 offset 没提交。
漏消费：先提交offset 后消费，有可能会造成数据的漏消费
![截屏2025-01-21 14.25.10.png](/upload/截屏2025-01-21%2014.25.10.png)




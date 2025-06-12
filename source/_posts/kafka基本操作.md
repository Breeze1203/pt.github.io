---
title: kafka基本操作
date: 2025-01-21 11:10:00
categories: kafka
tags: 
 - kafka
---

##### kafka基本操作
**1. 安装Kafka ,可以使用本地脚本和下载的文件或 docker 镜像以 KRaft 模式运行**
下载文件安装不多说，以 KRaft 模式运行如下
```
docker pull apache/kafka:3.9.0
docker run -p 9092:9092 apache/kafka:3.9.0
```
**2. 启动zookper**
运行以下命令以便以正确的顺序启动所有服务：
```
# Start the ZooKeeper service
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```
打开另一个终端会话并运行：
```
# Start the Kafka broker service
$ bin/kafka-server-start.sh config/server.properties
```
所有服务成功启动后，您将拥有一个正在运行并可供使用的基本 Kafka 环境。
但我习惯直接进入docker容器进行kafka操作
![截屏2025-01-21 11.24.28.png](/upload/截屏2025-01-21%2011.24.28.png)
config是kafka配置文件，bin目录下是操作命令
![截屏2025-01-21 11.26.49.png](/upload/截屏2025-01-21%2011.26.49.png)
包含一些命令，消费者命令，生产者命令，主题操作命令，zookeeper命令等
**3.常用命令**
文档链接 https://kafka.apache.org/documentation/#operations
**4.java操作**
1. 生产者异步发送
```
public class CustomizeProducer {
    public static void main(String[] args) {
        // 1. 创建 kafka生产者的配置对象
        Properties properties = new Properties();
        // 2. 给kafka 配置对象添加配置信息：bootstrap.servers
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,
                "localhost:9092");
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringSerializer");
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringSerializer");
       // 3. 创建 kafka生产者对象
        KafkaProducer<String, String> kafkaProducer = new
                KafkaProducer<String, String>(properties);
      // 4. 调用 send方法,发送消息
        for (int i = 200; i < 210; i++) {
            kafkaProducer.send(new
                    ProducerRecord<>("my_topic",2,"","分区二上传数据:----atguigu_pt " + i));
        }
      // 5. 关闭资源
        kafkaProducer.close();
    }
}
```
2. 生产者同步发送，调用get
```
// 同步发送
kafkaProducer.send(new ProducerRecord<>("three", "同步发送----kafka" + i)).get();
```
3. 事务
```
Kafka 的事务一共有如下 5 个API
// 1 初始化事务
void initTransactions();
// 2 开启事务
void beginTransaction() throws ProducerFencedException;
// 3 在事务内提交已经消费的偏移量（主要用于消费者）
void sendOffsetsToTransaction(Map<TopicPartition, OffsetAndMetadata> offsets,
String consumerGroupId) throws
ProducerFencedException;
// 4 提交事务
void commitTransaction() throws ProducerFencedException;
// 5 放弃事务（类似于回滚事务的操作）
void abortTransaction() throws ProducerFencedException;
```
4. 消费者
- 同步提交offset
```
public class CustomConsumerByHandSync {
    public static void main(String[] args) {
        // 1. 创建 kafka消费者配置类
        Properties properties = new Properties();
// 2. 添加配置参数
// 添加连接
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,
                "localhost:9092");
// 配置序列化 必须
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringDeserializer");
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringDeserializer");
// 配置消费者组
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
// 是否自动提交 offset
        properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG,
                false);
//3. 创建kafka消费者
        KafkaConsumer<String, String> consumer = new
                KafkaConsumer<>(properties);
//4. 设置消费主题 形参是列表
        consumer.subscribe(Arrays.asList("__consumer_offsets"));
//5. 消费数据
        while (true){
// 读取消息
            ConsumerRecords<String, String> consumerRecords =
                    consumer.poll(Duration.ofSeconds(1));
// 输出消息
            for (ConsumerRecord<String, String> consumerRecord :
                    consumerRecords) {
                System.out.println(consumerRecord.value());
            }
// 同步提交offset
            consumer.commitSync();
        }
    }
}
```
- 异步提交offset
```
public class CustomConsumerByHandAsync {
    public static void main(String[] args) {
        // 1. 创建 kafka消费者配置类
        Properties properties = new Properties();
// 2. 添加配置参数
        // 添加连接
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,
                "localhost:9092");
// 配置序列化 必须
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringDeserializer");
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringDeserializer");
// 配置消费者组
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
// 是否自动提交 offset
        properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG,
                "false");
//3. 创建Kafka消费者
        KafkaConsumer<String, String> consumer = new
                KafkaConsumer<>(properties);
//4. 设置消费主题 形参是列表
        consumer.subscribe(Arrays.asList("my_topic"));
//5. 消费数据
        while (true){
// 读取消息
            ConsumerRecords<String, String> consumerRecords =
                    consumer.poll(Duration.ofSeconds(1));
// 输出消息
            for (ConsumerRecord<String, String> consumerRecord :
                    consumerRecords) {
                System.out.println(consumerRecord.value());
            }
// 异步提交offset
            consumer.commitAsync();
        }
    }
}
```
- 指定消费时间消费
```
public class CustomConsumerForTime {
    public static void main(String[] args) {
        Properties properties = new Properties();
// 连接
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,
                "localhost:9092");
// key value 反序列化
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
                StringDeserializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
                StringDeserializer.class.getName());
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test2");
// 1 创建一个消费者
        KafkaConsumer<String, String> kafkaConsumer = new
                KafkaConsumer<>(properties);
// 2 订阅一个主题
        ArrayList<String> topics = new ArrayList<>();
        topics.add("my_topic");
        kafkaConsumer.subscribe(topics);
        Set<TopicPartition> assignment = new HashSet<>();
        while (assignment.size() == 0) {
            kafkaConsumer.poll(Duration.ofSeconds(1));
// 获取消费者分区分配信息（有了分区分配信息才能开始消费）
            assignment = kafkaConsumer.assignment();
        }
        Map<TopicPartition, Long> timestampToSearch = new
                HashMap<>();
// 封装集合存储，每个分区对应一天前的数据
        for (TopicPartition topicPartition : assignment) {
            timestampToSearch.put(topicPartition,
                    System.currentTimeMillis() - 1 * 24 * 3600 * 1000);
        }
// 获取从1天前开始消费的每个分区的 offset
        Map<TopicPartition, OffsetAndTimestamp> offsets =
                kafkaConsumer.offsetsForTimes(timestampToSearch);
// 遍历每个分区，对每个分区设置消费时间。
        for (TopicPartition topicPartition : assignment) {
            OffsetAndTimestamp offsetAndTimestamp =
                    offsets.get(topicPartition);
// 根据时间指定开始消费的位置
            if (offsetAndTimestamp != null){
                kafkaConsumer.seek(topicPartition,
                        offsetAndTimestamp.offset());
            }
        }// 3 消费该主题数据
        while (true) {
            ConsumerRecords<String, String> consumerRecords =
                    kafkaConsumer.poll(Duration.ofSeconds(1));
            for (ConsumerRecord<String, String> consumerRecord :
                    consumerRecords) {
                System.out.println(consumerRecord);
            }
        }
    }
}
```
- 指定offset消费
```
  public class CustomConsumerSeek {
    public static void main(String[] args) {
        // 0 配置信息
        Properties properties = new Properties();
// 连接
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,
                "localhost:9092");
// key value 反序列化
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
                StringDeserializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
                StringDeserializer.class.getName());
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test2");
// 1 创建一个消费者
        KafkaConsumer<String, String> kafkaConsumer = new
                KafkaConsumer<>(properties);
        // 2 订阅一个主题  可订阅多个主题
// 指定要消费的主题和分区
        <!-- String topic = "my_topic";
        List<TopicPartition> partitions = new ArrayList<>();
        partitions.add(new TopicPartition(topic, 0)); // 消费分区 0
        partitions.add(new TopicPartition(topic, 1)); // 消费分区 1
        // 手动分配分区
        kafkaConsumer.assign(partitions);
 -->
        ArrayList<String> topics = new ArrayList<>();
        topics.add("my_topic");
        kafkaConsumer.subscribe(topics);
        Set<TopicPartition> assignment= new HashSet<>();
        while (assignment.size() == 0) {
            kafkaConsumer.poll(Duration.ofSeconds(1));
       // 获取消费者分区分配信息（有了分区分配信息才能开始消费）
            assignment = kafkaConsumer.assignment();
        }
// 遍历所有分区，并指定 offset从1700 的位置开始消费
        for (TopicPartition tp: assignment) {
            kafkaConsumer.seek(tp, 1700);
        }
// 3 消费该主题数据
        while (true) {
            ConsumerRecords<String, String> consumerRecords =
                    kafkaConsumer.poll(Duration.ofSeconds(1));
            for (ConsumerRecord<String, String> consumerRecord :
                    consumerRecords) {
                System.out.println(consumerRecord);
            }
        }
    }
}
```

  


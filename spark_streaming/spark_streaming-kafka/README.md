# spark_streaming-kafka

这个模块实现了spark streaming 连接kafka组件，并且从中获取数据，手动维护offset到内部或者外部存储介质。
本模块提供了基本offset存储方案，已经预定义了接口，能够适应不同的场景。


> [!TIP]
> 1. 手动维护offset，可以手动调整消费的进度，跳过指定位置或者重复处理指定位置的数据。
> 2. 最好不要启用checkpoint机制，可能会出现重复消费的问题。


## 如何使用

仅仅20行左右代码就能够完成Spark Streaming 和kafka组件的最简单使用，大大减少了开发量。

### 使用说明
1. sparkConfMap：储存了基本SparkConf信息和自定义配置信息。具体信息查看com.opensharing.bigdata.conf.TemplateConf和SparkConf官方配置。
2. kafkaConfMap：存储了Kafka的基本配置，key是取于kafka官方的配置key值，与官方的完全一致
3. SparkStreamingKafka.setHandler：用于配置取出数据之后如何处理，由用户自定义实现，默认实现为ConsoleKafkaRDDHandler，可以配置多个，多个串行执行，无需用户持久化，会根据用户设置的Handler数量来判断是否启用持久化。
4. offsetTemplate：offset的存储模板，使用获取/存储offset的值，默认有三个实现OffsetInKafkaTemplate，OffsetInMysqlTemplate，OffsetInZookeeperTemplate。不满足用户需求可以自定义实现OffsetTemplate接口。
   + OffsetInMysqlTemplate：启用了Hutools的数据连接配置，用户可以按照这个类实现自己的数据库连接，数据存储读取。
5. stopFilePath：用于配置优雅停止的信号文件地址(不为空就表示启用优雅停止)。Spark Streaming采用的是判断信号文件是否存在来调用stop函数来达到优雅停止的目的，**暴力停止可能会导致消息的重复消费**。
6. hdfsUrl：优雅停止时必填，信号文件默认放到hdfs上(因为以集群模式启动spark application的时候不知道，driver端不知道在哪台机器，所以最好是使用公共文件存储系统)。
7. stopSecond：优雅停止时，轮询检测信号文件的时间间隔
8. checkPointPath：checkPoint地址最好是使用hdfs地址，如果不为空则判断启用checkPoint，手动维护偏移量的时候最好不要使用checkPoint。
9. hiveSupport：是否启用hive支持，默认是关闭的，可以调用startHiveSupport启用Hive支持

#### <a id='zk' href='zk' /> Offset In Zookeeper

``` java
	@Test
	public void testZk() {
		String topic = "spider-task";
		//如果kafkaConfMap设置了group_id,SparkStreamingKafka可不设置group_id
		String groupId = "spark-template";
		Map<Object, Object> sparkConfMap = new HashMap<>();
		sparkConfMap.put(TemplateConfEnum.APP_NAME, "testZk");
		sparkConfMap.put(TemplateConfEnum.MASTER, "local[4]");
		sparkConfMap.put(TemplateConfEnum.DURATION, Durations.seconds(10));
		sparkConfMap.put("spark.streaming.kafka.maxRatePerPartition", "10");
		Map<String, Object> kafkaConfMap = new HashMap<>();
		kafkaConfMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.2.58:9092,192.168.2.58:10092,192.168.2.58:11092");
		kafkaConfMap.put(ConsumerConfig.GROUP_ID_CONFIG, "spark-template");
		kafkaConfMap.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
		kafkaConfMap.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
		kafkaConfMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
		kafkaConfMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
		Map<ZkConfEnum, Object> zkConfMap = new HashMap<>();
		zkConfMap.put(ZkConfEnum.URL, "127.0.0.1:2181");
		zkConfMap.put(ZkConfEnum.CONNECTION_TIMEOUT, "3000");
		SparkStreamingKafka spark = SparkStreamingKafka.create(sparkConfMap, kafkaConfMap);
        //SparkStreamingKafka spark = SparkStreamingKafka.create(sparkConfMap, kafkaConfMap,"./checkpointStreamingZk");
		spark.setTopicName(topic);
		spark.setOffsetTemplate(new OffsetInZookeeperTemplate(zkConfMap, "/ldk"));
		spark.start();
	}
```
在传入SparkConfMap和KafkaConfMap之后，set offset的存储模板为kafka模板(OffsetInKafkaTemplate),传入KafkaConfMap以用于创建kafka连接

#### <a id='mysql' href='mysql' /> Offset In Mysql

``` java
	@Test
	public void testMysql() {
		String topic = "spider-task";
		//如果kafkaConfMap设置了group_id,SparkStreamingKafka可不设置group_id
		String groupId = "spark-template";
		Map<Object, Object> sparkConfMap = new HashMap<>();
		sparkConfMap.put(TemplateConfEnum.APP_NAME, "testMysql");
		sparkConfMap.put(TemplateConfEnum.MASTER, "local[4]");
		sparkConfMap.put(TemplateConfEnum.DURATION, Durations.seconds(10));
		Map<String, Object> kafkaConfMap = new HashMap<>();
		kafkaConfMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.2.58:9092,192.168.2.58:10092,192.168.2.58:11092");
		kafkaConfMap.put(ConsumerConfig.GROUP_ID_CONFIG, "spark-template");
		kafkaConfMap.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
		kafkaConfMap.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
		kafkaConfMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
		kafkaConfMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
		SparkStreamingKafka spark = SparkStreamingKafka.create(sparkConfMap, kafkaConfMap);
//        SparkStreamingKafka spark = SparkStreamingKafka.create(sparkConfMap, kafkaConfMap,"./checkpointStreamingMysql");
		spark.setTopicName(topic);
		spark.setOffsetTemplate(new OffsetInMysqlTemplate("kafka_offset"));
		spark.start();
	}
```

##### 使用步骤

1. 在resource下创建db.setting，用于创建数据库连接池，具体的内容可以查看源码resource，或者hutools的db.setting
2. 在pom中引入数据库连接池，任选(创建连接池的过程由hutools完成)
3. 在数据库执行sql/kafka_offset.sql，生成数据表。
3. 调用SparkStreamingKafka对象的setOffsetTemplate方法，将OffsetInMysqlTemplate对象传入。

##### 扩展数据库

还可以自定义实现其他数据库的存储方式，只要是实现了OffsetTemplate接口。
建议：start之前最好是将数据库连接池生成，如果不使用连接池，需要在实现类中维护数据库连接的获取和释放，破坏了代码结构。
1. 实现OffsetTemplate接口。
2. 调用SparkStreamingKafka对象的setOffsetTemplate方法，将OffsetTemplate接口实现类传入。

#### <a id='kafka' href='kafka' /> Offset In Kafka
``` java
	@Test
	public void testKafka() {
		String topic = "spider-task";
		//如果kafkaConfMap设置了group_id,SparkStreamingKafka可不设置group_id
		String groupId = "spark-template";
		Map<Object, Object> sparkConfMap = new HashMap<>();
		sparkConfMap.put(TemplateConfEnum.APP_NAME, "testKafka");
		sparkConfMap.put(TemplateConfEnum.MASTER, "local[4]");
		sparkConfMap.put(TemplateConfEnum.DURATION, Durations.seconds(10));
		Map<String, Object> kafkaConfMap = new HashMap<>();
		kafkaConfMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.2.58:9092,192.168.2.58:10092,192.168.2.58:11092");
		kafkaConfMap.put(ConsumerConfig.GROUP_ID_CONFIG, "spark-template");
		kafkaConfMap.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
		kafkaConfMap.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
		kafkaConfMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
		kafkaConfMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
		SparkStreamingKafka spark = SparkStreamingKafka.create(sparkConfMap, kafkaConfMap);
//        SparkStreamingKafka spark = SparkStreamingKafka.create(sparkConfMap, kafkaConfMap,"./checkpointStreamingKafka");
		spark.setTopicName(topic);
		spark.setOffsetTemplate(new OffsetInKafkaTemplate(kafkaConfMap));
		spark.start();
	}
```

需要在OffsetInZookeeperTemplate模板中传入zk配置和存储信息节点位置(在zk上可以没有，会自动创建)。


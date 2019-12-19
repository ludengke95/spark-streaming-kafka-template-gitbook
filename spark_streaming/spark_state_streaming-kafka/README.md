# spark_state_streaming-kafka

本模块结合了Spark State Streaming与kafka，从kafka中取数据组装成key，value的形式放到Streaming的持久化区，用于实时更新key，value，根据集合完成任务。

## 如何使用

### 使用说明
以offset存储在zk为示例，sparkConfMap，kafkaConfMap与Spark Streaming Kafka使用一致。
不同点：
+ StateStreaming增加了泛型，`<Long,String>`对应的State中的key的类型和value的类型，需要在声明的时候指定
+ 增加了一个必填参数，checkpointPath，StateStreaming要求必须要启用checkpoint机制，**切记：启用checkpoint的时候一定要使用优雅停止，不可使用暴力停止，可能会导致消息的重复消费**
+ 重写了addHandler方法，参数修改为PairRDDHandler(com.opensharing.bigdata.handler.PairRDDHandler)，是用于处理全部数据接口，默认有一个ConsoleKafkaPairRDDHandler实现，输出key，value的内容。如果不满足需求，仿照ConsoleKafkaPairRDDHandler实现一个自定义类即可，并且设置到SparkStateStreamingKafka中。
+ 增加了setUpdateStateHandler方法（参数KafkaUpdateStateHandler），用于设置数据重复时的更新规则以及kafka数据流转化为key，value结构的转换。默认有一个NullUpdateStateHandler实现。如果不满足需求，仿照NullUpdateStateHandler实现一个自定义类即可，并且设置到SparkStateStreamingKafka中。
   - 更新规则：更新为不为空的值，如果都不为空则更新为新值
   - 结构转化：ConsumerRecord<String, String>转化为Tuple2(value.length(),value)，以value的长度为key，value作为value。
+ 增加了timeOut对象，作用：用于控制Key的过期时间如果，key过期了，使用key对应的新value作为新值。
+ 默认实现可以再配置SparkStateStreamingKafka对象的时候不设置，会自动配置。

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
		sparkConfMap.put("spark.streaming.kafka.maxRatePerPartition", "1");
		Map<String, Object> kafkaConfMap = new HashMap<>();
		kafkaConfMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.2.58:9092,192.168.2.58:10092,192.168.2.58:11092");
		kafkaConfMap.put(ConsumerConfig.GROUP_ID_CONFIG, "spark-state-template");
		kafkaConfMap.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
		kafkaConfMap.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
		kafkaConfMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
		kafkaConfMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
		Map<ZkConfEnum, Object> zkConfMap = new HashMap<>();
		zkConfMap.put(ZkConfEnum.URL, "127.0.0.1:2181");
		zkConfMap.put(ZkConfEnum.CONNECTION_TIMEOUT, "3000");
		SparkStateStreamingKafka spark = new SparkStateStreamingKafka<Long, String>(sparkConfMap, kafkaConfMap, "./checkpointStateStreamingZk");
		spark.setTopicName(topic);
		spark.setOffsetTemplate(new OffsetInZookeeperTemplate(zkConfMap, "/ldk"));
		spark.start();
	}
```

## 功能

+ [x] 自定义更新规则/自定义流转化规则
+ [x] 全部数据快照处理
+ [x] offset auto自定义存储
+ [ ] 初始化key，value
+ [ ] 新增数据部分的快照处理

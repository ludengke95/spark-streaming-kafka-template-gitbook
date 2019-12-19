# offset in zookeeper

## 调用实例
``` java
    @Test
    public void testZk(){
        String topic = "spider-task";
        /*
        如果kafkaConfMap设置了group_id,SparkStreamingKafka可不设置group_id
         */
//        String groupId = "spark-template";
        Map<Object,Object> sparkConfMap = new HashMap<>();
        sparkConfMap.put(TemplateConf.APP_NAME,"testZk");
        sparkConfMap.put(TemplateConf.MASTER,"local[4]");
        sparkConfMap.put(TemplateConf.DURATION, Durations.seconds(10));
        Map<String,Object> kafkaConfMap = new HashMap<>();
        kafkaConfMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.2.58:9092,192.168.2.58:10092,192.168.2.58:11092");
        kafkaConfMap.put(ConsumerConfig.GROUP_ID_CONFIG, "spark-template");
        kafkaConfMap.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        kafkaConfMap.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
        kafkaConfMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        kafkaConfMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        Map<ZkConf,Object> zkConfMap = new HashMap<>();
        zkConfMap.put(ZkConf.URL,"127.0.0.1:2181");
        zkConfMap.put(ZkConf.CONNECTION_TIMEOUT,"3000");
        SparkStreamingKafka spark = SparkStreamingKafka.create(sparkConfMap, kafkaConfMap)
                .setTopicName(topic)
                .setOffsetTemplate(new OffsetInZookeeperTemplate(zkConfMap,"/ldk"));
        spark.start();
    }
```

需要在OffsetInZookeeperTemplate模板中传入zk配置和存储信息节点位置(在zk上可以没有，会自动创建)。

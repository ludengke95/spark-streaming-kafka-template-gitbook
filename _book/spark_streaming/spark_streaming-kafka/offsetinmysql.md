# offset in mysql/TiDB

## 调用实例

``` java
    @Test
    public void testMysql(){
        String topic = "spider-task";
        /*
        如果kafkaConfMap设置了group_id,SparkStreamingKafka可不设置group_id
         */
//        String groupId = "spark-template";
        Map<Object,Object> sparkConfMap = new HashMap<>();
        sparkConfMap.put(TemplateConf.APP_NAME,"testMysql");
        sparkConfMap.put(TemplateConf.MASTER,"local[4]");
        sparkConfMap.put(TemplateConf.DURATION, Durations.seconds(10));
        Map<String,Object> kafkaConfMap = new HashMap<>();
        kafkaConfMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.2.58:9092,192.168.2.58:10092,192.168.2.58:11092");
        kafkaConfMap.put(ConsumerConfig.GROUP_ID_CONFIG, "spark-template");
        kafkaConfMap.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        kafkaConfMap.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
        kafkaConfMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        kafkaConfMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        SparkStreamingKafka spark = SparkStreamingKafka.create(sparkConfMap, kafkaConfMap)
                .setTopicName(topic)
                .setOffsetTemplate(new OffsetInMysqlTemplate("kafka_offset"));
        spark.start();
    }
```

## 使用步骤

1. 在resource下创建db.setting，用于创建数据库连接池，具体的内容可以查看源码resource，或者hutools的db.setting
2. 在pom中引入数据库连接池，任选(创建连接池的过程由hutools完成)
3. 在数据库执行sql/kafka_offset.sql，生成数据表。
3. 调用SparkStreamingKafka对象的setOffsetTemplate方法，将OffsetInMysqlTemplate对象传入。

## 扩展数据库

还可以自定义实现其他数据库的存储方式，只要是实现了OffsetTemplate接口。
建议：start之前最好是将数据库连接池生成，如果不使用连接池，需要在实现类中维护数据库连接的获取和释放，破坏了代码结构。
1. 实现OffsetTemplate接口。
2. 调用SparkStreamingKafka对象的setOffsetTemplate方法，将OffsetTemplate接口实现类传入。

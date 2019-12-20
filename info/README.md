# spark-streaming-kafka-template 

这个模块包含spark_streaming与其他组件结合的模板。

## <a id='introduction' href='introduction' />简介
+ 是不是还在头疼 SparkStreaming 如何组合kafka，SparkStateStreaming怎么实现数据去重
+ Kafka 引入了，但是 Offset 怎么存储又是个问题，是由 kafka 自动管理还是，存储到 zk ，还是写到 mysql。
+ spark-streaming-kafka-template 就是为了解决诸如此类的问题应运而生的，希望能够帮助你简化开发。

这个项目的初衷就是为了简化 SparkStreaming 对接 Kafka，至于这个轮子圆不圆，走不走的远就要靠大家来检验了。

## <a id='doc' href='doc' />文档
1. [spark-streaming-kafka-template 文档](http://106.12.51.176)

## <a id='install' href='install' />安装
1. 由于没有向maven中央仓库提交，请使用手动编译(可调整组件版本)或者下载target下的jar包放到lib中。
2. 还可以放到本地的maven仓中，或者上传到所使用maven私有仓。

## <a id='join' href='join' />添砖加瓦
+ 项目还在进行中，只有我一个人，如果你觉得可以动动你的小手，点一点 fork，star。
+ 如果你也对这个项目有想法，可以加入我们(一个人可以说我们嘛？)  
联系方式：ludengke95@gmail.com/ludengke95@163.com


# spark_streaming

这个模块包含spark_streaming与其他组件结合的模板。

## 使用建议
1. 代码内部已经自动维护偏移量，最好不要启用checkpoint(StateStreaming要求必须启用checkpoint，没办法)
2. 本地测试可以暴力停止，线上环境禁止暴力停止(可能会造成数据重复消费等问题)，需使用优雅停止。
3. 实现接口或者抽象类的时候记得加上`Serializable`。

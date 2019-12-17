# spark_streaming-kafka

这个模块实现了spark streaming 连接kafka组件，并且从中获取数据，手动维护offset到内部或者外部存储介质。

强烈建议手动维护offset，可以手动调整消费的进度，跳过指定位置或者重复处理指定位置的数据。

本模块提供了基本offset存储方案，已经预定义了接口，能够适应不同的场景。



腾讯云消息服务（Cloud Message Queue）是分布式消息队列服务，能够为分布式部署的不同应用之间或者一个应用的不同组件之间，提供基于消息的可靠的异步通信机制，消息被存储在高可靠、高可用的 CMQ 队列中，多进程可以同时读写，互不干扰。

CMQ 提供了四种 SDK，这里仅用 Python 简介。

## 1. Python SDK 简介

为了方便使用， CMQ 将用户操作、队列操作、主题操作等抽象成了几个类：

- Account：封装账户 secretId、secretKey，用户可以创建删除队列、主题及订阅，并查看这些对象；
- queue：收发消息，查看设置队列属性；
- topic：发布消息，查看设置主题属性，查看订阅者；
- cmq_client：可以设置一些客户端的与服务器端连接属性，如设置是否写日志、连接超时时间、是否长连接等。

这里需要注意的是，所有的类均为非线程安全的，如果需要多线程使用，最好每个线程实例化自己的对象。


**[点击下载SDK>>](https://cloud.tencent.com/document/product/406/6107)**


## 2. 队列模型

这里的队列与数据结构中定义的 Queue 有一定区别。数据结构中的队列严格按照 FIFO 操作，这里的分布式队列不会有严格的FIFO（后续会推出专属的FIFO产品）。这里的队列相当于一个高性能、高容量、高可靠的容器，可以生产消息投递进去，也可以把消息取出来消费。队列在初始化的时候有自己的属性设置。

下面简述一下这些属性以及含义：

| 属性 | 描述 |
|---------|---------|
| maxMsgHeapNum | 最大堆积消息数。队列中可以存储的消息条数，该属性表示了队列的存储和堆积能力。 |
| pollingWaitSeconds | 消息接收长轮询等待时间。取值范围 0-30 秒，该时间设置表示的是消费消息时默认等待接收消息的时间。<br>例如设置为10，那么在消费消息时，如果没有消息， 默认会等待10s返回；如果有消息则会立即返回。<br>注：也可以在接收消息的时候设置自定义的等待时间，不使用队列的属性值。 | 
| visibilityTimeout | 消息可见性超时。<br>消息被消费者获取到，会有一个不可见时间，即在这个时间之内别的消费者无法获取这条消息。取值范围 1-43200 秒（即12小时内），默认值为30。 | 
| maxMsgSize | 消息最大长度。取值范围 1024-65536 Byte（即1K-64K），默认值为65536。 | 
| msgRetentionSeconds | 消息保留周期，即消息在队列中的保存时间，取值范围 60-1296000 秒（1min-15天），默认值为 345600 (4 天)。 | 
| createTime | 队列的创建时间。返回 Unix 时间戳，精确到秒。 | 
| lastModifyTime | 最后一次修改队列属性的时间。返回 Unix 时间戳，精确到秒。 | 
| activeMsgNum | 在队列中处于 Active 状态（不处于被消费状态）的消息总数，为近似值。 |  
| inactiveMsgNum | 在队列中处于 Inactive 状态（正处于被消费状态）的消息总数，为近似值。 | 
| rewindSeconds | 回溯队列的消息回溯时间最大值，取值范围 0-43200 秒，0 表示不开启消息回溯。 | 
| rewindmsgNum | 已调用 DelMsg 接口删除但还在回溯保留时间内的消息数量。 | 
| minMsgTime | 消息最小未消费时间，单位为秒。 | 
| delayMsgNum | 延时消息数量。| 

[**查看队列模型快速入门 >>**](/document/product/406/8436)

## 3. 主题模型

主题模型类似设计模式中的发布订阅模式，Topic 相当于发布消息的单位，Topic 下面的订阅者相当于观察者模式。Topic 会把发布的消息主动推送给订阅者：
    

| 属性 | 描述 |
|---------|---------|
| msgCount | 当前该主题中堆积的消息数目（消息堆积数)。 |
| maxMsgSize | 消息最大长度。取值范围 1024-65536 Byte（即1-64K），默认值为 65536。 |
| msgRetentionSeconds | 消息在主题中最长存活时间，从发送到该主题开始经过此参数指定的时间后，不论消息是否被成功推送给用户都将被删除，该参数单位为秒。固定为一天(86400秒)，该属性不能修改。 |
| createTime | 主题的创建时间。返回 Unix 时间戳，精确到秒。 |
| lastModifyTime | 最后一次修改主题属性的时间。返回 Unix 时间戳，精确到秒。 |
| filterType | 描述用户创建订阅时选择的过滤策略：<br> filterType =1 表示用户使用 filterTag 标签过滤；<br>filterType =2 表示用户使用 bindingKey 过滤。 |

[**查看主题模型快速入门 >>**](/document/product/406/8437)
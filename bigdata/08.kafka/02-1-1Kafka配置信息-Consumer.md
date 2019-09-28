##  Consumer配置信息

| **属性**                        | **默认值**  | **描述**                                                     |
| ------------------------------- | ----------- | ------------------------------------------------------------ |
| group.id                        |             | Consumer的组ID，相同goup.id的consumer属于同一个组。          |
| zookeeper.connect               |             | Consumer的zookeeper连接串，要和broker的配置一致。            |
| consumer.id                     | null        | 如果不设置会自动生成。                                       |
| socket.timeout.ms               | 30 * 1000   | 网络请求的socket超时时间。实际超时时间由max.fetch.wait + socket.timeout.ms 确定。 |
| socket.receive.buffer.bytes     | 64 * 1024   | The socket receive buffer for network requests.              |
| fetch.message.max.bytes         | 1024 * 1024 | 查询topic-partition时允许的最大消息大小。consumer会为每个partition缓存此大小的消息到内存，因此，这个参数可以控制consumer的内存使用量。这个值应该至少比server允许的最大消息大小大，以免producer发送的消息大于consumer允许的消息。 |
| num.consumer.fetchers           | 1           | The number fetcher threads used to fetch data.               |
| auto.commit.enable              | true        | 如果此值设置为true，consumer会周期性的把当前消费的offset值保存到zookeeper。当consumer失败重启之后将会使用此值作为新开始消费的值。 |
| auto.commit.interval.ms         | 60 * 1000   | Consumer提交offset值到zookeeper的周期。                      |
| queued.max.message.chunks       | 2           | 用来被consumer消费的message chunks 数量， 每个chunk可以缓存fetch.message.max.bytes大小的数据量。 |
| auto.commit.interval.ms         | 60 * 1000   | Consumer提交offset值到zookeeper的周期。                      |
| queued.max.message.chunks       | 2           | 用来被consumer消费的message chunks 数量， 每个chunk可以缓存fetch.message.max.bytes大小的数据量。 |
| fetch.min.bytes                 | 1           | The minimum amount of data the server should return for a fetch request. If insufficient data is available the request will wait for that much data to accumulate before answering the request. |
| fetch.wait.max.ms               | 100         | The maximum amount of time the server will block before answering the fetch request if there isn’t sufficient data to immediately satisfy fetch.min.bytes. |
| rebalance.backoff.ms            | 2000        | Backoff time between retries during rebalance.               |
| refresh.leader.backoff.ms       | 200         | Backoff time to wait before trying to determine the leader of a partition that has just lost its leader. |
| auto.offset.reset               | largest     | What to do when there is no initial offset in ZooKeeper or if an offset is out of range ;smallest : automatically reset the offset to the smallest offset; largest : automatically reset the offset to the largest offset;anything else: throw exception to the consumer |
| consumer.timeout.ms             | -1          | 若在指定时间内没有消息消费，consumer将会抛出异常。           |
| exclude.internal.topics         | true        | Whether messages from internal topics (such as offsets) should be exposed to the consumer. |
| zookeeper.session.timeout.ms    | 6000        | ZooKeeper session timeout. If the consumer fails to heartbeat to ZooKeeper for this period of time it is considered dead and a rebalance will occur. |
| zookeeper.connection.timeout.ms | 6000        | The max time that the client waits while establishing a connection to zookeeper. |
| zookeeper.sync.time.ms          | 2000        | How far a ZK follower can be behind a ZK leader              |
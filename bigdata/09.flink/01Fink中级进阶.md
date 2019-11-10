### 1中级进阶
#### 01.课程内容介绍	
#### 02.source讲解-java

source是程序的数据源输入，你可以通过StreamExecutionEnvironment.addSource(sourceFunction)来为你的程序添加一个source。
flink提供了大量的已经实现好的source方法，你也可以自定义source
通过实现sourceFunction接口来自定义无并行度的source，
或者你也可以通过实现ParallelSourceFunction 接口 or 继承RichParallelSourceFunction 来自定义有并行度的source。

基于文件
readTextFile(path)
读取文本文件，文件遵循TextInputFormat 读取规则，逐行读取并返回。
基于socket
socketTextStream从socker中读取数据，元素可以通过一个分隔符切开。
基于集合
fromCollection(Collection)
通过java 的collection集合创建一个数据流，集合中的所有元素必须是相同类型的。
自定义输入
addSource 可以实现读取第三方数据源的数据
系统内置提供了一批connectors，连接器会提供对应的source支持【kafka】		

```java

```
#### 03.自定义source-1	

实现并行度为1的自定义source
实现SourceFunction 
一般不需要实现容错性保证
处理好cancel方法(cancel应用的时候，这个方法会被调用)
实现并行化的自定义source
实现ParallelSourceFunction 
或者继承RichParallelSourceFunction 	

```java
/**
 * 把collection集合作为数据源
 *
 */
public class StreamingFromCollection {

    public static void main(String[] args) throws Exception {
        //获取Flink的运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        ArrayList<Integer> data = new ArrayList<>();
        data.add(10);
        data.add(15);
        data.add(20);
        


        //指定数据源
        DataStreamSource<Integer> collectionData = env.fromCollection(data);

        //通map对数据进行处理
        DataStream<Integer> num = collectionData.map(new MapFunction<Integer, Integer>() {
            @Override
            public Integer map(Integer value) throws Exception {
                return value + 1;
            }
        });

        //直接打印
        num.print().setParallelism(1);

        env.execute("StreamingFromCollection");


    }
}
```
#### 04.自定义source-2		

**NoParalleSource**

```java
/**
 * 自定义实现并行度为1的source
 *
 * 模拟产生从1开始的递增数字
 *
 *
 * 注意：
 * SourceFunction 和 SourceContext 都需要指定数据类型，如果不指定，代码运行的时候会报错

 */
public class MyNoParalleSource implements SourceFunction<Long>{

    private long count = 1L;

    private boolean isRunning = true;

    /**
     * 主要的方法
     * 启动一个source
     * 大部分情况下，都需要在这个run方法中实现一个循环，这样就可以循环产生数据了
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void run(SourceContext<Long> ctx) throws Exception {
        while(isRunning){
            ctx.collect(count);
            count++;
            //每秒产生一条数据
            Thread.sleep(1000);
        }
    }

    /**
     * 取消一个cancel的时候会调用的方法
     *
     */
    @Override
    public void cancel() {
        isRunning = false;
    }
}


/**
 * 使用并行度为1的source
 *
 * Created by xuwei.tech on 2018/10/23.
 */
public class StreamingDemoWithMyNoPralalleSource {

    public static void main(String[] args) throws Exception {
        //获取Flink的运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //获取数据源
        DataStreamSource<Long> text = env.addSource(new MyNoParalleSource()).setParallelism(1);//注意：针对此source，并行度只能设置为1

        DataStream<Long> num = text.map(new MapFunction<Long, Long>() {
            @Override
            public Long map(Long value) throws Exception {
                System.out.println("接收到数据：" + value);
                return value;
            }
        });

        //每2秒钟处理一次数据
        DataStream<Long> sum = num.timeWindowAll(Time.seconds(2)).sum(0);

        //打印结果
        sum.print().setParallelism(1);

        String jobName = StreamingDemoWithMyNoPralalleSource.class.getSimpleName();
        env.execute(jobName);
    }
}

```
### ParalleSource

```java
/**
 * 自定义实现一个支持并行度的source
 */
public class MyParalleSource implements ParallelSourceFunction<Long> {

    private long count = 1L;

    private boolean isRunning = true;

    /**
     * 主要的方法
     * 启动一个source
     * 大部分情况下，都需要在这个run方法中实现一个循环，这样就可以循环产生数据了
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void run(SourceContext<Long> ctx) throws Exception {
        while(isRunning){
            ctx.collect(count);
            count++;
            //每秒产生一条数据
            Thread.sleep(1000);
        }
    }

    /**
     * 取消一个cancel的时候会调用的方法
     *
     */
    @Override
    public void cancel() {
        isRunning = false;
    }
}

/**
 * 使用多并行度的source
 *
 * Created by xuwei.tech on 2018/10/23.
 */
public class StreamingDemoWithMyPralalleSource {

    public static void main(String[] args) throws Exception {
        //获取Flink的运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //获取数据源
        DataStreamSource<Long> text = env.addSource(new MyParalleSource()).setParallelism(2);

        DataStream<Long> num = text.map(new MapFunction<Long, Long>() {
            @Override
            public Long map(Long value) throws Exception {
                System.out.println("接收到数据：" + value);
                return value;
            }
        });

        //每2秒钟处理一次数据
        DataStream<Long> sum = num.timeWindowAll(Time.seconds(2)).sum(0);

        //打印结果
        sum.print().setParallelism(1);

        String jobName = StreamingDemoWithMyPralalleSource.class.getSimpleName();
        env.execute(jobName);
    }
}

```

**RichParalleSource**

```java
/**
 * 自定义实现一个支持并行度的source
 *
 * RichParallelSourceFunction 会额外提供open和close方法
 * 针对source中如果需要获取其他链接资源，那么可以在open方法中获取资源链接，在close中关闭资源链接
 *
 */
public class MyRichParalleSource extends RichParallelSourceFunction<Long> {

    private long count = 1L;

    private boolean isRunning = true;

    /**
     * 主要的方法
     * 启动一个source
     * 大部分情况下，都需要在这个run方法中实现一个循环，这样就可以循环产生数据了
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void run(SourceContext<Long> ctx) throws Exception {
        while(isRunning){
            ctx.collect(count);
            count++;
            //每秒产生一条数据
            Thread.sleep(1000);
        }
    }

    /**
     * 取消一个cancel的时候会调用的方法
     *
     */
    @Override
    public void cancel() {
        isRunning = false;
    }


    /**
     * 这个方法只会在最开始的时候被调用一次
     * 实现获取链接的代码
     * @param parameters
     * @throws Exception
     */
    @Override
    public void open(Configuration parameters) throws Exception {
        System.out.println("open.............");
        super.open(parameters);
    }

    /**
     * 实现关闭链接的代码
     * @throws Exception
     */
    @Override
    public void close() throws Exception {
        super.close();
    }
}


/**
 * 使用多并行度的source
 *
 * Created by xuwei.tech on 2018/10/23.
 */
public class StreamingDemoWithMyRichPralalleSource {

    public static void main(String[] args) throws Exception {
        //获取Flink的运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //获取数据源
        DataStreamSource<Long> text = env.addSource(new MyRichParalleSource()).setParallelism(2);

        DataStream<Long> num = text.map(new MapFunction<Long, Long>() {
            @Override
            public Long map(Long value) throws Exception {
                System.out.println("接收到数据：" + value);
                return value;
            }
        });

        //每2秒钟处理一次数据
        DataStream<Long> sum = num.timeWindowAll(Time.seconds(2)).sum(0);

        //打印结果
        sum.print().setParallelism(1);

        String jobName = StreamingDemoWithMyRichPralalleSource.class.getSimpleName();
        env.execute(jobName);
    }
}
```



#### 05.算子操作-java

map：输入一个元素，然后返回一个元素，中间可以做一些清洗转换等操作
flatmap：输入一个元素，可以返回零个，一个或者多个元素
filter：过滤函数，对传入的数据进行判断，符合条件的数据会被留下
keyBy：根据指定的key进行分组，相同key的数据会进入同一个分区【典型用法见备注】
reduce：对数据进行聚合操作，结合当前元素和上一次reduce返回的值进行聚合操作，然后返回一个新的值
aggregations：sum(),min(),max()等
window：在后面单独详解

Union：合并多个流，新的流会包含所有流中的数据，但是union是一个限制，就是所有合并的流类型必须是一致的。
Connect：和union类似，但是只能连接两个流，两个流的数据类型可以不同，会对两个流中的数据应用不同的处理方法。
CoMap, CoFlatMap：在ConnectedStreams中需要使用这种函数，类似于map和flatmap
Split：根据规则把一个数据流切分为多个流
Select：和split配合使用，选择切分后的流		

```java
/**
 * Filter演示
 *
 * Created by xuwei.tech on 2018/10/23.
 */
public class StreamingDemoFilter {

    public static void main(String[] args) throws Exception {
        //获取Flink的运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //获取数据源
        DataStreamSource<Long> text = env.addSource(new MyNoParalleSource()).setParallelism(1);//注意：针对此source，并行度只能设置为1

        DataStream<Long> num = text.map(new MapFunction<Long, Long>() {
            @Override
            public Long map(Long value) throws Exception {
                System.out.println("原始接收到数据：" + value);
                return value;
            }
        });

        //执行filter过滤，满足条件的数据会被留下
        DataStream<Long> filterData = num.filter(new FilterFunction<Long>() {
            //把所有的奇数过滤掉
            @Override
            public boolean filter(Long value) throws Exception {
                return value % 2 == 0;
            }
        });

        DataStream<Long> resultData = filterData.map(new MapFunction<Long, Long>() {
            @Override
            public Long map(Long value) throws Exception {
                System.out.println("过滤之后的数据：" + value);
                return value;
            }
        });


        //每2秒钟处理一次数据
        DataStream<Long> sum = resultData.timeWindowAll(Time.seconds(2)).sum(0);

        //打印结果
        sum.print().setParallelism(1);

        String jobName = StreamingDemoFilter.class.getSimpleName();
        env.execute(jobName);
    }
}

/**
 * union
 * 合并多个流，新的流会包含所有流中的数据，但是union是一个限制，就是所有合并的流类型必须是一致的
 *
 * Created by xuwei.tech on 2018/10/23.
 */
public class StreamingDemoUnion {

    public static void main(String[] args) throws Exception {
        //获取Flink的运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //获取数据源
        DataStreamSource<Long> text1 = env.addSource(new MyNoParalleSource()).setParallelism(1);//注意：针对此source，并行度只能设置为1

        DataStreamSource<Long> text2 = env.addSource(new MyNoParalleSource()).setParallelism(1);

        //把text1和text2组装到一起
        DataStream<Long> text = text1.union(text2);

        DataStream<Long> num = text.map(new MapFunction<Long, Long>() {
            @Override
            public Long map(Long value) throws Exception {
                System.out.println("原始接收到数据：" + value);
                return value;
            }
        });



        //每2秒钟处理一次数据
        DataStream<Long> sum = num.timeWindowAll(Time.seconds(2)).sum(0);

        //打印结果
        sum.print().setParallelism(1);

        String jobName = StreamingDemoUnion.class.getSimpleName();
        env.execute(jobName);
    }
}


/**
 * connect
 * 和union类似，但是只能连接两个流，两个流的数据类型可以不同，会对两个流中的数据应用不同的处理方法
 *
 * Created by xuwei.tech on 2018/10/23.
 */
public class StreamingDemoConnect {

    public static void main(String[] args) throws Exception {
        //获取Flink的运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //获取数据源
        DataStreamSource<Long> text1 = env.addSource(new MyNoParalleSource()).setParallelism(1);//注意：针对此source，并行度只能设置为1

        DataStreamSource<Long> text2 = env.addSource(new MyNoParalleSource()).setParallelism(1);
        SingleOutputStreamOperator<String> text2_str = text2.map(new MapFunction<Long, String>() {
            @Override
            public String map(Long value) throws Exception {
                return "str_" + value;
            }
        });

        ConnectedStreams<Long, String> connectStream = text1.connect(text2_str);

        SingleOutputStreamOperator<Object> result = connectStream.map(new CoMapFunction<Long, String, Object>() {
            @Override
            public Object map1(Long value) throws Exception {
                return value;
            }

            @Override
            public Object map2(String value) throws Exception {
                return value;
            }
        });


        //打印结果
        result.print().setParallelism(1);

        String jobName = StreamingDemoConnect.class.getSimpleName();
        env.execute(jobName);
    }
}


/**
 * split
 *
 * 根据规则把一个数据流切分为多个流
 *
 * 应用场景：
 * 可能在实际工作中，源数据流中混合了多种类似的数据，多种类型的数据处理规则不一样，所以就可以在根据一定的规则，
 * 把一个数据流切分成多个数据流，这样每个数据流就可以使用不用的处理逻辑了
 *
 * Created by xuwei.tech on 2018/10/23.
 */
public class StreamingDemoSplit {

    public static void main(String[] args) throws Exception {
        //获取Flink的运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //获取数据源
        DataStreamSource<Long> text = env.addSource(new MyNoParalleSource()).setParallelism(1);//注意：针对此source，并行度只能设置为1

        //对流进行切分，按照数据的奇偶性进行区分
        SplitStream<Long> splitStream = text.split(new OutputSelector<Long>() {
            @Override
            public Iterable<String> select(Long value) {
                ArrayList<String> outPut = new ArrayList<>();
                if (value % 2 == 0) {
                    outPut.add("even");//偶数
                } else {
                    outPut.add("odd");//奇数
                }
                return outPut;
            }
        });
        
        //选择一个或者多个切分后的流
        DataStream<Long> evenStream = splitStream.select("even");
        DataStream<Long> oddStream = splitStream.select("odd");

        DataStream<Long> moreStream = splitStream.select("odd","even");


        //打印结果
        moreStream.print().setParallelism(1);

        String jobName = StreamingDemoSplit.class.getSimpleName();
        env.execute(jobName);
    }
}

```
#### 06.partition-java

Random partitioning：随机分区
dataStream.shuffle()
Rebalancing：对数据集进行再平衡，重分区，消除数据倾斜
dataStream.rebalance()
Rescaling：解释见备注
dataStream.rescale()
Custom partitioning：自定义分区
自定义分区需要实现Partitioner接口
dataStream.partitionCustom(partitioner, "someKey")
或者dataStream.partitionCustom(partitioner, 0);
Broadcasting：在后面单独详解		

```java
public class MyPartition implements Partitioner<Long> {
    @Override
    public int partition(Long key, int numPartitions) {
        System.out.println("分区总数："+numPartitions);
        if(key % 2 == 0){
            return 0;
        }else{
            return 1;
        }
    }
}

/**
 *
 * 使用自定义分析
 * 根据数字的奇偶性来分区
 *
 * Created by xuwei.tech on 2018/10/23.
 */
public class SteamingDemoWithMyParitition {

    public static void main(String[] args) throws Exception{

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(2);
        DataStreamSource<Long> text = env.addSource(new MyNoParalleSource());

        //对数据进行转换，把long类型转成tuple1类型
        DataStream<Tuple1<Long>> tupleData = text.map(new MapFunction<Long, Tuple1<Long>>() {
            @Override
            public Tuple1<Long> map(Long value) throws Exception {
                return new Tuple1<>(value);
            }
        });
        //分区之后的数据
        DataStream<Tuple1<Long>> partitionData= tupleData.partitionCustom(new MyPartition(), 0);

        DataStream<Long> result = partitionData.map(new MapFunction<Tuple1<Long>, Long>() {
            @Override
            public Long map(Tuple1<Long> value) throws Exception {
                System.out.println("当前线程id：" + Thread.currentThread().getId() + ",value: " + value);
                return value.getField(0);
            }
        });

        result.print().setParallelism(1);

        env.execute("SteamingDemoWithMyParitition");

    }
}
```
#### 07.sink-java

writeAsText()：将元素以字符串形式逐行写入，这些字符串通过调用每个元素的toString()方法来获取
print() / printToErr()：打印每个元素的toString()方法的值到标准输出或者标准错误输出流中
自定义输出addSink【kafka、redis】		

```xml
 		<dependency>
            <groupId>org.apache.bahir</groupId>
            <artifactId>flink-connector-redis_2.11</artifactId>
            <version>1.0</version>
        </dependency>
```



```java
/**
 * 接收socket数据，把数据保存到redis中
 *
 * list
 *
 * lpush list_key value
 *
 * Created by xuwei.tech on 2018/10/23.
 */
public class StreamingDemoToRedis {

    public static void main(String[] args) throws Exception{
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<String> text = env.socketTextStream("hadoop100", 9000, "\n");

        //lpsuh l_words word

        //对数据进行组装,把string转化为tuple2<String,String>
        DataStream<Tuple2<String, String>> l_wordsData = text.map(new MapFunction<String, Tuple2<String, String>>() {
            @Override
            public Tuple2<String, String> map(String value) throws Exception {
                return new Tuple2<>("l_words", value);
            }
        });

        //创建redis的配置
        FlinkJedisPoolConfig conf = new FlinkJedisPoolConfig.Builder().setHost("hadoop110").setPort(6379).build();

        //创建redissink
        RedisSink<Tuple2<String, String>> redisSink = new RedisSink<>(conf, new MyRedisMapper());

        l_wordsData.addSink(redisSink);

        env.execute("StreamingDemoToRedis");
    }

    public static class MyRedisMapper implements RedisMapper<Tuple2<String, String>>{
        //表示从接收的数据中获取需要操作的redis key
        @Override
        public String getKeyFromData(Tuple2<String, String> data) {
            return data.f0;
        }
        //表示从接收的数据中获取需要操作的redis value
        @Override
        public String getValueFromData(Tuple2<String, String> data) {
            return data.f1;
        }

        @Override
        public RedisCommandDescription getCommandDescription() {
            return new RedisCommandDescription(RedisCommand.LPUSH);
        }
    }

}
```
#### 08.source-scala		

**NoParallelSource**

```java
/**
  * 创建自定义并行度为1的source
  *
  * 实现从1开始产生递增数字
  *
  * Created by xuwei.tech on 2018/10/23.
  */
class MyNoParallelSourceScala extends SourceFunction[Long]{

  var count = 1L
  var isRunning = true

  override def run(ctx: SourceContext[Long]) = {
    while(isRunning){
      ctx.collect(count)
      count+=1
      Thread.sleep(1000)
    }

  }

  override def cancel() = {
    isRunning = false
  }
}

object StreamingDemoWithMyNoParallelSourceScala {

  def main(args: Array[String]): Unit = {

    val env = StreamExecutionEnvironment.getExecutionEnvironment

    //隐式转换
    import org.apache.flink.api.scala._

    val text = env.addSource(new MyNoParallelSourceScala)

    val mapData = text.map(line=>{
      println("接收到的数据："+line)
      line
    })

    val sum = mapData.timeWindowAll(Time.seconds(2)).sum(0)


    sum.print().setParallelism(1)

    env.execute("StreamingDemoWithMyNoParallelSourceScala")



  }

}

```
**ParallelSource**

```java
/**
  * 创建自定义并行度为1的source
  *
  * 实现从1开始产生递增数字
  *
  * Created by xuwei.tech on 2018/10/23.
  */
class MyParallelSourceScala extends ParallelSourceFunction[Long]{

  var count = 1L
  var isRunning = true

  override def run(ctx: SourceContext[Long]) = {
    while(isRunning){
      ctx.collect(count)
      count+=1
      Thread.sleep(1000)
    }

  }

  override def cancel() = {
    isRunning = false
  }
}

object StreamingDemoWithMyParallelSourceScala {

  def main(args: Array[String]): Unit = {

    val env = StreamExecutionEnvironment.getExecutionEnvironment

    //隐式转换
    import org.apache.flink.api.scala._

    val text = env.addSource(new MyParallelSourceScala).setParallelism(2)

    val mapData = text.map(line=>{
      println("接收到的数据："+line)
      line
    })

    val sum = mapData.timeWindowAll(Time.seconds(2)).sum(0)


    sum.print().setParallelism(1)

    env.execute("StreamingDemoWithMyNoParallelSourceScala")



  }

}

```

**RichParallelSource**

```java
/**
  * 创建自定义并行度为1的source
  *
  * 实现从1开始产生递增数字
  *
  * Created by xuwei.tech on 2018/10/23.
  */
class MyRichParallelSourceScala extends RichParallelSourceFunction[Long]{

  var count = 1L
  var isRunning = true

  override def run(ctx: SourceContext[Long]) = {
    while(isRunning){
      ctx.collect(count)
      count+=1
      Thread.sleep(1000)
    }

  }

  override def cancel() = {
    isRunning = false
  }

  override def open(parameters: Configuration): Unit = super.open(parameters)

  override def close(): Unit = super.close()
}

object StreamingDemoWithMyRichParallelSourceScala {

  def main(args: Array[String]): Unit = {

    val env = StreamExecutionEnvironment.getExecutionEnvironment

    //隐式转换
    import org.apache.flink.api.scala._

    val text = env.addSource(new MyRichParallelSourceScala).setParallelism(2)

    val mapData = text.map(line=>{
      println("接收到的数据："+line)
      line
    })

    val sum = mapData.timeWindowAll(Time.seconds(2)).sum(0)


    sum.print().setParallelism(1)

    env.execute("StreamingDemoWithMyNoParallelSourceScala")

  }

}
```



#### 09.算子操作-scala		

```java
object StreamingDemoFilterScala {

  def main(args: Array[String]): Unit = {

    val env = StreamExecutionEnvironment.getExecutionEnvironment

    //隐式转换
    import org.apache.flink.api.scala._

    val text = env.addSource(new MyNoParallelSourceScala)

    val mapData = text.map(line=>{
      println("原始接收到的数据："+line)
      line
    }).filter(_ % 2 == 0)

    val sum = mapData.map(line=>{
      println("过滤之后的数据："+line)
      line
    }).timeWindowAll(Time.seconds(2)).sum(0)


    sum.print().setParallelism(1)

    env.execute("StreamingDemoWithMyNoParallelSourceScala")



  }

}

object StreamingDemoSplitScala {

  def main(args: Array[String]): Unit = {

    val env = StreamExecutionEnvironment.getExecutionEnvironment

    //隐式转换
    import org.apache.flink.api.scala._

    val text = env.addSource(new MyNoParallelSourceScala)

    val splitStream = text.split(new OutputSelector[Long] {
      override def select(value: Long) = {
        val list = new util.ArrayList[String]()
        if(value%2 == 0){
          list.add("even")// 偶数
        }else{
          list.add("odd")// 奇数
        }
        list
      }
    })

    val evenStream = splitStream.select("even")

    evenStream.print().setParallelism(1)

    env.execute("StreamingDemoWithMyNoParallelSourceScala")



  }

}

object StreamingDemoConnectScala {

  def main(args: Array[String]): Unit = {

    val env = StreamExecutionEnvironment.getExecutionEnvironment

    //隐式转换
    import org.apache.flink.api.scala._

    val text1 = env.addSource(new MyNoParallelSourceScala)
    val text2 = env.addSource(new MyNoParallelSourceScala)

    val text2_str = text2.map("str" + _)

    val connectedStreams = text1.connect(text2_str)

    val result = connectedStreams.map(line1=>{line1},line2=>{line2})

    result.print().setParallelism(1)

    env.execute("StreamingDemoWithMyNoParallelSourceScala")



  }

}

object StreamingDemoUnionScala {

  def main(args: Array[String]): Unit = {

    val env = StreamExecutionEnvironment.getExecutionEnvironment

    //隐式转换
    import org.apache.flink.api.scala._

    val text1 = env.addSource(new MyNoParallelSourceScala)
    val text2 = env.addSource(new MyNoParallelSourceScala)


    val unionall = text1.union(text2)

    val sum = unionall.map(line=>{
      println("接收到的数据："+line)
      line
    }).timeWindowAll(Time.seconds(2)).sum(0)

    sum.print().setParallelism(1)

    env.execute("StreamingDemoWithMyNoParallelSourceScala")



  }

}
```
#### 10.partition-scala		
```java
class MyPartitionerScala extends Partitioner[Long]{

  override def partition(key: Long, numPartitions: Int) = {
    println("分区总数："+numPartitions)
    if(key % 2 ==0){
      0
    }else{
      1
    }

  }

}

object StreamingDemoMyPartitionerScala {

  def main(args: Array[String]): Unit = {

    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(2)

    //隐式转换
    import org.apache.flink.api.scala._

    val text = env.addSource(new MyNoParallelSourceScala)

    //把long类型的数据转成tuple类型
    val tupleData = text.map(line=>{
      Tuple1(line)// 注意tuple1的实现方式
    })

    val partitionData = tupleData.partitionCustom(new MyPartitionerScala,0)

    val result = partitionData.map(line=>{
      println("当前线程id："+Thread.currentThread().getId+",value: "+line)
      line._1
    })

    result.print().setParallelism(1)

    env.execute("StreamingDemoWithMyNoParallelSourceScala")



  }

}
```
#### 11.sink-scala		
```java
object StreamingDataToRedisScala {

  def main(args: Array[String]): Unit = {

    //获取socket端口号
    val port = 9000

    //获取运行环境
    val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment

    //链接socket获取输入数据
    val text = env.socketTextStream("hadoop100",port,'\n')

    //注意：必须要添加这一行隐式转行，否则下面的flatmap方法执行会报错
    import org.apache.flink.api.scala._

    val l_wordsData = text.map(line=>("l_words_scala",line))

    val conf = new FlinkJedisPoolConfig.Builder().setHost("hadoop110").setPort(6379).build()

    val redisSink = new RedisSink[Tuple2[String,String]](conf,new MyRedisMapper)

    l_wordsData.addSink(redisSink)

    //执行任务
    env.execute("Socket window count");


  }

  class MyRedisMapper extends RedisMapper[Tuple2[String,String]]{

    override def getKeyFromData(data: (String, String)) = {
      data._1
    }

    override def getValueFromData(data: (String, String)) = {
      data._2
    }

    override def getCommandDescription = {
      new RedisCommandDescription(RedisCommand.LPUSH)
    }
  }


}
```
#### 12.DataSet之算子操作-java-1

基于文件
readTextFile(path)
基于集合
fromCollection(Collection)

Map：输入一个元素，然后返回一个元素，中间可以做一些清洗转换等操作
FlatMap：输入一个元素，可以返回零个，一个或者多个元素
MapPartition：类似map，一次处理一个分区的数据【如果在进行map处理的时候需要获取第三方资源链接，建议使用MapPartition】
Filter：过滤函数，对传入的数据进行判断，符合条件的数据会被留下
Reduce：对数据进行聚合操作，结合当前元素和上一次reduce返回的值进行聚合操作，然后返回一个新的值
Aggregate：sum、max、min等
Distinct：返回一个数据集中去重之后的元素，data.distinct()
Join：内连接
OuterJoin：外链接		

```java
public class BatchDemoDistinct {

    public static void main(String[] args) throws Exception{

        //获取运行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        ArrayList<String> data = new ArrayList<>();
        data.add("hello you");
        data.add("hello me");

        DataSource<String> text = env.fromCollection(data);

        FlatMapOperator<String, String> flatMapData = text.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
                String[] split = value.toLowerCase().split("\\W+");
                for (String word : split) {
                    System.out.println("单词："+word);
                    out.collect(word);
                }
            }
        });

        flatMapData.distinct()// 对数据进行整体去重
                .print();


    }



}

public class BatchDemoJoin {

    public static void main(String[] args) throws Exception{

        //获取运行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        //tuple2<用户id，用户姓名>
        ArrayList<Tuple2<Integer, String>> data1 = new ArrayList<>();
        data1.add(new Tuple2<>(1,"zs"));
        data1.add(new Tuple2<>(2,"ls"));
        data1.add(new Tuple2<>(3,"ww"));


        //tuple2<用户id，用户所在城市>
        ArrayList<Tuple2<Integer, String>> data2 = new ArrayList<>();
        data2.add(new Tuple2<>(1,"beijing"));
        data2.add(new Tuple2<>(2,"shanghai"));
        data2.add(new Tuple2<>(3,"guangzhou"));


        DataSource<Tuple2<Integer, String>> text1 = env.fromCollection(data1);
        DataSource<Tuple2<Integer, String>> text2 = env.fromCollection(data2);


        text1.join(text2).where(0)//指定第一个数据集中需要进行比较的元素角标
                        .equalTo(0)//指定第二个数据集中需要进行比较的元素角标
                        .with(new JoinFunction<Tuple2<Integer,String>, Tuple2<Integer,String>, Tuple3<Integer,String,String>>() {
                            @Override
                            public Tuple3<Integer, String, String> join(Tuple2<Integer, String> first, Tuple2<Integer, String> second)
                                    throws Exception {
                                return new Tuple3<>(first.f0,first.f1,second.f1);
                            }
                        }).print();

        System.out.println("==================================");

        //注意，这里用map和上面使用的with最终效果是一致的。
        /*text1.join(text2).where(0)//指定第一个数据集中需要进行比较的元素角标
                .equalTo(0)//指定第二个数据集中需要进行比较的元素角标
                .map(new MapFunction<Tuple2<Tuple2<Integer,String>,Tuple2<Integer,String>>, Tuple3<Integer,String,String>>() {
                    @Override
                    public Tuple3<Integer, String, String> map(Tuple2<Tuple2<Integer, String>, Tuple2<Integer, String>> value) throws Exception {
                        return new Tuple3<>(value.f0.f0,value.f0.f1,value.f1.f1);
                    }
                }).print();*/



    }



}

public class BatchDemoUnion {

    public static void main(String[] args) throws Exception{

        //获取运行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        ArrayList<Tuple2<Integer, String>> data1 = new ArrayList<>();
        data1.add(new Tuple2<>(1,"zs"));
        data1.add(new Tuple2<>(2,"ls"));
        data1.add(new Tuple2<>(3,"ww"));


        ArrayList<Tuple2<Integer, String>> data2 = new ArrayList<>();
        data2.add(new Tuple2<>(1,"lili"));
        data2.add(new Tuple2<>(2,"jack"));
        data2.add(new Tuple2<>(3,"jessic"));


        DataSource<Tuple2<Integer, String>> text1 = env.fromCollection(data1);
        DataSource<Tuple2<Integer, String>> text2 = env.fromCollection(data2);

        UnionOperator<Tuple2<Integer, String>> union = text1.union(text2);

        union.print();



    }



}

```
#### 13.DataSet之算子操作-java-2

Cross：获取两个数据集的笛卡尔积
Union：返回两个数据集的总和，数据类型需要一致
First-n：获取集合中的前N个元素
Sort Partition：在本地对数据集的所有分区进行排序，通过sortPartition()的链接调用来完成对多个字段的排序		

```java
public class BatchDemoOuterJoin {

    public static void main(String[] args) throws Exception{

        //获取运行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        //tuple2<用户id，用户姓名>
        ArrayList<Tuple2<Integer, String>> data1 = new ArrayList<>();
        data1.add(new Tuple2<>(1,"zs"));
        data1.add(new Tuple2<>(2,"ls"));
        data1.add(new Tuple2<>(3,"ww"));


        //tuple2<用户id，用户所在城市>
        ArrayList<Tuple2<Integer, String>> data2 = new ArrayList<>();
        data2.add(new Tuple2<>(1,"beijing"));
        data2.add(new Tuple2<>(2,"shanghai"));
        data2.add(new Tuple2<>(4,"guangzhou"));


        DataSource<Tuple2<Integer, String>> text1 = env.fromCollection(data1);
        DataSource<Tuple2<Integer, String>> text2 = env.fromCollection(data2);

        /**
         * 左外连接
         *
         * 注意：second这个tuple中的元素可能为null
         *
         */
        text1.leftOuterJoin(text2)
                .where(0)
                .equalTo(0)
                .with(new JoinFunction<Tuple2<Integer,String>, Tuple2<Integer,String>, Tuple3<Integer,String,String>>() {
                    @Override
                    public Tuple3<Integer, String, String> join(Tuple2<Integer, String> first, Tuple2<Integer, String> second) throws Exception {
                        if(second==null){
                            return new Tuple3<>(first.f0,first.f1,"null");
                        }else{
                            return new Tuple3<>(first.f0,first.f1,second.f1);
                        }

                    }
                }).print();

        System.out.println("=============================================================================");

        /**
         * 右外连接
         *
         * 注意：first这个tuple中的数据可能为null
         *
         */
        text1.rightOuterJoin(text2)
                .where(0)
                .equalTo(0)
                .with(new JoinFunction<Tuple2<Integer,String>, Tuple2<Integer,String>, Tuple3<Integer,String,String>>() {
                    @Override
                    public Tuple3<Integer, String, String> join(Tuple2<Integer, String> first, Tuple2<Integer, String> second) throws Exception {
                        if(first==null){
                            return new Tuple3<>(second.f0,"null",second.f1);
                        }
                        return new Tuple3<>(first.f0,first.f1,second.f1);
                    }
                }).print();



        System.out.println("=============================================================================");

        /**
         * 全外连接
         *
         * 注意：first和second这两个tuple都有可能为null
         *
         */

        text1.fullOuterJoin(text2)
                .where(0)
                .equalTo(0)
                .with(new JoinFunction<Tuple2<Integer,String>, Tuple2<Integer,String>, Tuple3<Integer,String,String>>() {
                    @Override
                    public Tuple3<Integer, String, String> join(Tuple2<Integer, String> first, Tuple2<Integer, String> second) throws Exception {
                        if(first==null){
                            return new Tuple3<>(second.f0,"null",second.f1);
                        }else if(second == null){
                            return new Tuple3<>(first.f0,first.f1,"null");
                        }else{
                            return new Tuple3<>(first.f0,first.f1,second.f1);
                        }
                    }
                }).print();


    }

}

public class BatchDemoCross {

    public static void main(String[] args) throws Exception{

        //获取运行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        //tuple2<用户id，用户姓名>
        ArrayList<String> data1 = new ArrayList<>();
        data1.add("zs");
        data1.add("ww");

        //tuple2<用户id，用户所在城市>
        ArrayList<Integer> data2 = new ArrayList<>();
        data2.add(1);
        data2.add(2);

        DataSource<String> text1 = env.fromCollection(data1);
        DataSource<Integer> text2 = env.fromCollection(data2);

        CrossOperator.DefaultCross<String, Integer> cross = text1.cross(text2);

        cross.print();


    }

}

public class BatchDemoFirstN {

    public static void main(String[] args) throws Exception{

        //获取运行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        ArrayList<Tuple2<Integer, String>> data = new ArrayList<>();
        data.add(new Tuple2<>(2,"zs"));
        data.add(new Tuple2<>(4,"ls"));
        data.add(new Tuple2<>(3,"ww"));
        data.add(new Tuple2<>(1,"xw"));
        data.add(new Tuple2<>(1,"aw"));
        data.add(new Tuple2<>(1,"mw"));


        DataSource<Tuple2<Integer, String>> text = env.fromCollection(data);


        //获取前3条数据，按照数据插入的顺序
        text.first(3).print();
        System.out.println("==============================");

        //根据数据中的第一列进行分组，获取每组的前2个元素
        text.groupBy(0).first(2).print();
        System.out.println("==============================");

        //根据数据中的第一列分组，再根据第二列进行组内排序[升序]，获取每组的前2个元素
        text.groupBy(0).sortGroup(1, Order.ASCENDING).first(2).print();
        System.out.println("==============================");

        //不分组，全局排序获取集合中的前3个元素，针对第一个元素升序，第二个元素倒序
        text.sortPartition(0,Order.ASCENDING).sortPartition(1,Order.DESCENDING).first(3).print();

    }



}
```
#### 14.DataSet之partition-java

Rebalance：对数据集进行再平衡，重分区，消除数据倾斜
Hash-Partition：根据指定key的哈希值对数据集进行分区
partitionByHash()
Range-Partition：根据指定的key对数据集进行范围分区
.partitionByRange()
Custom Partitioning：自定义分区规则
自定义分区需要实现Partitioner接口
partitionCustom(partitioner, "someKey")
或者partitionCustom(partitioner, 0)		

```java
public class BatchDemoMapPartition {

    public static void main(String[] args) throws Exception{

        //获取运行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        ArrayList<String> data = new ArrayList<>();
        data.add("hello you");
        data.add("hello me");

        DataSource<String> text = env.fromCollection(data);

        /*text.map(new MapFunction<String, String>() {
            @Override
            public String map(String value) throws Exception {
                //获取数据库连接--注意，此时是每过来一条数据就获取一次链接
                //处理数据
                //关闭连接
                return value;
            }
        });*/


        DataSet<String> mapPartitionData = text.mapPartition(new MapPartitionFunction<String, String>() {
            @Override
            public void mapPartition(Iterable<String> values, Collector<String> out) throws Exception {
                //获取数据库连接--注意，此时是一个分区的数据获取一次连接【优点，每个分区获取一次链接】
                //values中保存了一个分区的数据
                //处理数据
                Iterator<String> it = values.iterator();
                while (it.hasNext()) {
                    String next = it.next();
                    String[] split = next.split("\\W+");
                    for (String word : split) {
                        out.collect(word);
                    }
                }
                //关闭链接
            }
        });

        mapPartitionData.print();


    }



}

public class BatchDemoHashRangePartition {

    public static void main(String[] args) throws Exception{

        //获取运行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        ArrayList<Tuple2<Integer, String>> data = new ArrayList<>();
        data.add(new Tuple2<>(1,"hello1"));
        data.add(new Tuple2<>(2,"hello2"));
        data.add(new Tuple2<>(2,"hello3"));
        data.add(new Tuple2<>(3,"hello4"));
        data.add(new Tuple2<>(3,"hello5"));
        data.add(new Tuple2<>(3,"hello6"));
        data.add(new Tuple2<>(4,"hello7"));
        data.add(new Tuple2<>(4,"hello8"));
        data.add(new Tuple2<>(4,"hello9"));
        data.add(new Tuple2<>(4,"hello10"));
        data.add(new Tuple2<>(5,"hello11"));
        data.add(new Tuple2<>(5,"hello12"));
        data.add(new Tuple2<>(5,"hello13"));
        data.add(new Tuple2<>(5,"hello14"));
        data.add(new Tuple2<>(5,"hello15"));
        data.add(new Tuple2<>(6,"hello16"));
        data.add(new Tuple2<>(6,"hello17"));
        data.add(new Tuple2<>(6,"hello18"));
        data.add(new Tuple2<>(6,"hello19"));
        data.add(new Tuple2<>(6,"hello20"));
        data.add(new Tuple2<>(6,"hello21"));


        DataSource<Tuple2<Integer, String>> text = env.fromCollection(data);

        /*text.partitionByHash(0).mapPartition(new MapPartitionFunction<Tuple2<Integer,String>, Tuple2<Integer,String>>() {
            @Override
            public void mapPartition(Iterable<Tuple2<Integer, String>> values, Collector<Tuple2<Integer, String>> out) throws Exception {
                Iterator<Tuple2<Integer, String>> it = values.iterator();
                while (it.hasNext()){
                    Tuple2<Integer, String> next = it.next();
                    System.out.println("当前线程id："+Thread.currentThread().getId()+","+next);
                }

            }
        }).print();*/


        text.partitionByRange(0).mapPartition(new MapPartitionFunction<Tuple2<Integer,String>, Tuple2<Integer,String>>() {
            @Override
            public void mapPartition(Iterable<Tuple2<Integer, String>> values, Collector<Tuple2<Integer, String>> out) throws Exception {
                Iterator<Tuple2<Integer, String>> it = values.iterator();
                while (it.hasNext()){
                    Tuple2<Integer, String> next = it.next();
                    System.out.println("当前线程id："+Thread.currentThread().getId()+","+next);
                }

            }
        }).print();



    }



}
```
#### 15.DataSet之算子操作-scala-1		
```scala
object BatchDemoCounterScala {

  def main(args: Array[String]): Unit = {

    val env = ExecutionEnvironment.getExecutionEnvironment

    import org.apache.flink.api.scala._

    val data = env.fromElements("a","b","c","d")

    val res = data.map(new RichMapFunction[String,String] {
      //1：定义累加器
      val numLines = new IntCounter

      override def open(parameters: Configuration): Unit = {
        super.open(parameters)
        //2:注册累加器
        getRuntimeContext.addAccumulator("num-lines",this.numLines)
      }

      override def map(value: String) = {
        this.numLines.add(1)
        value
      }

    }).setParallelism(4)


    res.writeAsText("d:\\data\\count21")
    val jobResult = env.execute("BatchDemoCounterScala")
    //3：获取累加器
    val num = jobResult.getAccumulatorResult[Int]("num-lines")
    println("num:"+num)

  }

}

object BatchDemoCrossScala {

  def main(args: Array[String]): Unit = {

    val env = ExecutionEnvironment.getExecutionEnvironment
    import org.apache.flink.api.scala._

    val data1 = List("zs","ww")


    val data2 = List(1,2)

    val text1 = env.fromCollection(data1)
    val text2 = env.fromCollection(data2)

    text1.cross(text2).print()




  }

}
object BatchDemoDistinctScala {

  def main(args: Array[String]): Unit = {

    val env = ExecutionEnvironment.getExecutionEnvironment
    import org.apache.flink.api.scala._

    val data = ListBuffer[String]()

    data.append("hello you")
    data.append("hello me")

    val text = env.fromCollection(data)

    val flatMapData = text.flatMap(line=>{
      val words = line.split("\\W+")
      for(word <- words){
        println("单词："+word)
      }
      words
    })

    flatMapData.distinct().print()


  }

}
object BatchDemoFirstNScala {

  def main(args: Array[String]): Unit = {

    val env = ExecutionEnvironment.getExecutionEnvironment
    import org.apache.flink.api.scala._

    val data = ListBuffer[Tuple2[Int,String]]()
    data.append((2,"zs"))
    data.append((4,"ls"))
    data.append((3,"ww"))
    data.append((1,"xw"))
    data.append((1,"aw"))
    data.append((1,"mw"))

    val text = env.fromCollection(data)

    //获取前3条数据，按照数据插入的顺序
    text.first(3).print()
    println("==============================")

    //根据数据中的第一列进行分组，获取每组的前2个元素
    text.groupBy(0).first(2).print()
    println("==============================")


    //根据数据中的第一列分组，再根据第二列进行组内排序[升序]，获取每组的前2个元素
    text.groupBy(0).sortGroup(1,Order.ASCENDING).first(2).print()
    println("==============================")


    //不分组，全局排序获取集合中的前3个元素，
    text.sortPartition(0,Order.ASCENDING).sortPartition(1,Order.DESCENDING).first(3).print()





  }

}
```
#### 16.DataSet之算子操作-scala-2		
```scala
object BatchDemoJoinScala {

  def main(args: Array[String]): Unit = {

    val env = ExecutionEnvironment.getExecutionEnvironment
    import org.apache.flink.api.scala._

    val data1 = ListBuffer[Tuple2[Int,String]]()
    data1.append((1,"zs"))
    data1.append((2,"ls"))
    data1.append((3,"ww"))


    val data2 = ListBuffer[Tuple2[Int,String]]()
    data2.append((1,"beijing"))
    data2.append((2,"shanghai"))
    data2.append((3,"guangzhou"))

    val text1 = env.fromCollection(data1)
    val text2 = env.fromCollection(data2)

    text1.join(text2).where(0).equalTo(0).apply((first,second)=>{
      (first._1,first._2,second._2)
    }).print()
  }

}

object BatchDemoOuterJoinScala {

  def main(args: Array[String]): Unit = {

    val env = ExecutionEnvironment.getExecutionEnvironment
    import org.apache.flink.api.scala._

    val data1 = ListBuffer[Tuple2[Int,String]]()
    data1.append((1,"zs"))
    data1.append((2,"ls"))
    data1.append((3,"ww"))


    val data2 = ListBuffer[Tuple2[Int,String]]()
    data2.append((1,"beijing"))
    data2.append((2,"shanghai"))
    data2.append((4,"guangzhou"))

    val text1 = env.fromCollection(data1)
    val text2 = env.fromCollection(data2)

    text1.leftOuterJoin(text2).where(0).equalTo(0).apply((first,second)=>{
      if(second==null){
        (first._1,first._2,"null")
      }else{
        (first._1,first._2,second._2)
      }
    }).print()

    println("===============================")

    text1.rightOuterJoin(text2).where(0).equalTo(0).apply((first,second)=>{
      if(first==null){
        (second._1,"null",second._2)
      }else{
        (first._1,first._2,second._2)
      }
    }).print()


    println("===============================")

    text1.fullOuterJoin(text2).where(0).equalTo(0).apply((first,second)=>{
      if(first==null){
        (second._1,"null",second._2)
      }else if(second==null){
        (first._1,first._2,"null")
      }else{
        (first._1,first._2,second._2)
      }
    }).print()

  }

}
object BatchDemoUnionScala {

  def main(args: Array[String]): Unit = {

    val env = ExecutionEnvironment.getExecutionEnvironment
    import org.apache.flink.api.scala._

    val data1 = ListBuffer[Tuple2[Int,String]]()
    data1.append((1,"zs"))
    data1.append((2,"ls"))
    data1.append((3,"ww"))


    val data2 = ListBuffer[Tuple2[Int,String]]()
    data2.append((1,"jack"))
    data2.append((2,"lili"))
    data2.append((3,"jessic"))

    val text1 = env.fromCollection(data1)
    val text2 = env.fromCollection(data2)

    text1.union(text2).print()

  }

}
```
**patition**

```scala
object BatchDemoMapPartitionScala {

  def main(args: Array[String]): Unit = {

    val env = ExecutionEnvironment.getExecutionEnvironment
    import org.apache.flink.api.scala._

    val data = ListBuffer[String]()

    data.append("hello you")
    data.append("hello me")

    val text = env.fromCollection(data)

    text.mapPartition(it=>{
      //创建数据库连接，建议吧这块代码放到try-catch代码块中
      val res = ListBuffer[String]()
      while(it.hasNext){
        val line = it.next()
        val words = line.split("\\W+")
        for(word <- words){
          res.append(word)
        }
      }
      res
      //关闭连接
    }).print()


  }

}

object BatchDemoHashRangePartitionScala {

  def main(args: Array[String]): Unit = {

    val env = ExecutionEnvironment.getExecutionEnvironment
    import org.apache.flink.api.scala._

    val data1 = ListBuffer[Tuple2[Int,String]]()
    data1.append((1,"hello1"))
    data1.append((2,"hello2"))
    data1.append((2,"hello3"))
    data1.append((3,"hello4"))
    data1.append((3,"hello5"))
    data1.append((3,"hello6"))
    data1.append((4,"hello7"))
    data1.append((4,"hello8"))
    data1.append((4,"hello9"))
    data1.append((4,"hello10"))
    data1.append((5,"hello11"))
    data1.append((5,"hello12"))
    data1.append((5,"hello13"))
    data1.append((5,"hello14"))
    data1.append((5,"hello15"))
    data1.append((6,"hello16"))
    data1.append((6,"hello17"))
    data1.append((6,"hello18"))
    data1.append((6,"hello19"))
    data1.append((6,"hello20"))
    data1.append((6,"hello21"))

    val text = env.fromCollection(data1)

    /*text.partitionByHash(0).mapPartition(it=>{
      while (it.hasNext){
        val tu = it.next()
        println("当前线程id："+Thread.currentThread().getId+","+tu)
      }
      it
    }).print()*/

    text.partitionByRange(0).mapPartition(it=>{
      while (it.hasNext){
        val tu = it.next()
        println("当前线程id："+Thread.currentThread().getId+","+tu)
      }
      it
    }).print()


  }

}
```



#### 17.Flink支持的dataType和序列化		

```java

```
#### 18.Flink Broadcast广播变量-(java代码)		
```java

```
#### 19.Flink Broadcast广播变量-(scala代码)		
```java

```
#### 20.Flink Counters-java代码		
```java

```
#### 21.Flink Counters-scala代码		
```java

```
#### 22.Flink Distributed Cache		
```java

```
#### 23.state之keyedState分析		
```java

```
#### 24.state之operatorState分析		
```java

```
#### 25.Flink checkPoint分析		
```java

```
#### 26.Flink state backend详细分析		
```java

```
#### 27.Flink state backend实战演示		
```java

```
#### 28.Flink 重启策略分析		
```java

```
#### 29.Flink 从checkpoint恢复数据		
```java

```
#### 30.Flink savePoint的使用详解1		
```java

```
#### 31.Flink window详解		
```java

```
#### 32.Flink time介绍		
```java

```
#### 33.Flink watermark介绍		
```java

```
#### 34.Flink watermark解决乱序数据-1		
```java

```
#### 35.Flink watermark解决乱序数据-2		
```java

```
#### 36.Flink parallelism并行度分析		
```java

```
#### 37.Flink UI界面介绍		
```java

```
#### 38.Flink kafka-connector分析		
```java

```
#### 39.kafka-connector代码操作-java		
```java

```
#### 40.kafka-connector代码操作scala		
```java

```
#### 41.Flink 生产环境配置介绍		
```java

```
#### 42.实战需求分析(数据清洗[实时ETL])		
```java

```
#### 43.数据清洗[实时ETL]-java代码实现-1		
```java

```
#### 44.数据清洗[实时ETL]-java代码实现-2		
```java

```
#### 45.数据清洗[实时ETL]-java代码提交集群运行		
```java

```
#### 46.数据清洗[实时ETL]-把任务提交命令封装成脚本		
```java

```
#### 47.数据清洗[实时ETL]-scala代码实现		
```java

```
#### 48.实战需求分析(数据报表)		
```java

```
#### 49.数据报表-java代码实现-1		
```java

```
#### 50.数据报表-java代码实现-2		
```java

```
#### 51.数据报表-es和kibana的安装		
```java

```
#### 52.数据报表-运行任务		
```java

```
#### 53.数据报表-执行脚本封装		
```java

```
#### 54.数据报表-scala代码实现	
```java

```

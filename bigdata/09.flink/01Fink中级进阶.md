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

writeAsText()：将元素以字符串形式逐行写入，这些字符串通过调用每个元素的toString()方法来获取
writeAsCsv()：将元组以逗号分隔写入文件中，行及字段之间的分隔是可配置的。每个字段的值来自对象的toString()方法
print()：打印每个元素的toString()方法的值到标准输出或者标准错误输出流中

------------------

Java Tuple 和 Scala case class
Java POJOs：java实体类
Primitive Types
默认支持java和scala基本数据类型
General Class Types
默认支持大多数java和scala class
Hadoop Writables
支持hadoop中实现了org.apache.hadoop.Writable的数据类型
Special Types
例如scala中的Either Option 和Try

---------------

Flink自带了针对诸如int，long，String等标准类型的序列化器
针对Flink无法实现序列化的数据类型，我们可以交给Avro和Kryo
使用方法：ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
使用avro序列化：env.getConfig().enableForceAvro();
使用kryo序列化：env.getConfig().enableForceKryo();
使用自定义序列化：env.getConfig().addDefaultKryoSerializer(Class<?> type, Class<? extends Serializer<?>> serializerClass)
https://ci.apache.org/projects/flink/flink-docs-release-1.6/dev/custom_serializers.html

```java

```
#### 18.Flink Broadcast广播变量-(java代码)

DataStreaming 中的Broadcast

把元素广播给所有的分区，数据会被重复处理
类似于storm中的allGrouping
dataStream.broadcast()

广播变量允许编程人员在每台机器上保持1个只读的缓存变量，而不是传送变量的副本给tasks
广播变量创建后，它可以运行在集群中的任何function上，而不需要多次传递给集群节点。另外需要记住，不应该修改广播变量，这样才能确保每个节点获取到的值都是一致的
一句话解释，可以理解为是一个公共的共享变量，我们可以把一个dataset 数据集广播出去，然后不同的task在节点上都能够获取到，这个数据在每个节点上只会存在一份。如果不使用broadcast，则在每个节点中的每个task中都需要拷贝一份dataset数据集，比较浪费内存(也就是一个节点中可能会存在多份dataset数据)。
用法
1：初始化数据
DataSet<Integer> toBroadcast = env.fromElements(1, 2, 3)
2：广播数据
.withBroadcastSet(toBroadcast, "broadcastSetName");
3：获取数据
Collection<Integer> broadcastSet = getRuntimeContext().getBroadcastVariable("broadcastSetName");
注意：
1：广播出去的变量存在于每个节点的内存中，所以这个数据集不能太大。因为广播出去的数据，会常驻内存，除非程序执行结束
2：广播变量在初始化广播出去以后不支持修改，这样才能保证每个节点的数据都是一致的。

​		

```java
/**
 * broadcast广播变量
 *
 *
 *
 * 需求：
 * flink会从数据源中获取到用户的姓名
 *
 * 最终需要把用户的姓名和年龄信息打印出来
 *
 * 分析：
 * 所以就需要在中间的map处理的时候获取用户的年龄信息
 *
 * 建议吧用户的关系数据集使用广播变量进行处理
 *
 *
 *
 *
 * 注意：如果多个算子需要使用同一份数据集，那么需要在对应的多个算子后面分别注册广播变量
 *
 *
 *
 * Created by xuwei.tech on 2018/10/8.
 */
public class BatchDemoBroadcast {

    public static void main(String[] args) throws Exception{

        //获取运行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        //1：准备需要广播的数据
        ArrayList<Tuple2<String, Integer>> broadData = new ArrayList<>();
        broadData.add(new Tuple2<>("zs",18));
        broadData.add(new Tuple2<>("ls",20));
        broadData.add(new Tuple2<>("ww",17));
        DataSet<Tuple2<String, Integer>> tupleData = env.fromCollection(broadData);


        //1.1:处理需要广播的数据,把数据集转换成map类型，map中的key就是用户姓名，value就是用户年龄
        DataSet<HashMap<String, Integer>> toBroadcast = tupleData.map(new MapFunction<Tuple2<String, Integer>, HashMap<String, Integer>>() {
            @Override
            public HashMap<String, Integer> map(Tuple2<String, Integer> value) throws Exception {
                HashMap<String, Integer> res = new HashMap<>();
                res.put(value.f0, value.f1);
                return res;
            }
        });

        //源数据
        DataSource<String> data = env.fromElements("zs", "ls", "ww");

        //注意：在这里需要使用到RichMapFunction获取广播变量
        DataSet<String> result = data.map(new RichMapFunction<String, String>() {

            List<HashMap<String, Integer>> broadCastMap = new ArrayList<HashMap<String, Integer>>();
            HashMap<String, Integer> allMap = new HashMap<String, Integer>();

            /**
             * 这个方法只会执行一次
             * 可以在这里实现一些初始化的功能
             *
             * 所以，就可以在open方法中获取广播变量数据
             *
             */
            @Override
            public void open(Configuration parameters) throws Exception {
                super.open(parameters);
                //3:获取广播数据
                this.broadCastMap = getRuntimeContext().getBroadcastVariable("broadCastMapName");
                for (HashMap map : broadCastMap) {
                    allMap.putAll(map);
                }

            }

            @Override
            public String map(String value) throws Exception {
                Integer age = allMap.get(value);
                return value + "," + age;
            }
        }).withBroadcastSet(toBroadcast, "broadCastMapName");//2：执行广播数据的操作



        result.print();


    }



}


/**
 *  broadcast分区规则
 */
public class StreamingDemoWithMyNoPralalleSourceBroadcast {

    public static void main(String[] args) throws Exception {
        //获取Flink的运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(4);

        //获取数据源
        DataStreamSource<Long> text = env.addSource(new MyNoParalleSource()).setParallelism(1);//注意：针对此source，并行度只能设置为1

        DataStream<Long> num = text.broadcast().map(new MapFunction<Long, Long>() {
            @Override
            public Long map(Long value) throws Exception {
                long id = Thread.currentThread().getId();
                System.out.println("线程id："+id+",接收到数据：" + value);
                return value;
            }
        });

        //每2秒钟处理一次数据
        DataStream<Long> sum = num.timeWindowAll(Time.seconds(2)).sum(0);

        //打印结果
        sum.print().setParallelism(1);

        String jobName = StreamingDemoWithMyNoPralalleSourceBroadcast.class.getSimpleName();
        env.execute(jobName);
    }
}
```
#### 19.Flink Broadcast广播变量-(scala代码)		
```scala
/**
  * broadcast 广播变量
  */
object BatchDemoBroadcastScala {

  def main(args: Array[String]): Unit = {

    val env = ExecutionEnvironment.getExecutionEnvironment

    import org.apache.flink.api.scala._

    //1: 准备需要广播的数据
    val broadData = ListBuffer[Tuple2[String,Int]]()
    broadData.append(("zs",18))
    broadData.append(("ls",20))
    broadData.append(("ww",17))

    //1.1处理需要广播的数据
    val tupleData = env.fromCollection(broadData)
    val toBroadcastData = tupleData.map(tup=>{
      Map(tup._1->tup._2)
    })


    val text = env.fromElements("zs","ls","ww")

    val result = text.map(new RichMapFunction[String,String] {


      var listData: java.util.List[Map[String,Int]] = null
      var allMap  = Map[String,Int]()

      override def open(parameters: Configuration): Unit = {
        super.open(parameters)
        this.listData = getRuntimeContext.getBroadcastVariable[Map[String,Int]]("broadcastMapName")
        val it = listData.iterator()
        while (it.hasNext){
          val next = it.next()
          allMap = allMap.++(next)
        }
      }

      override def map(value: String) = {
        val age = allMap.get(value).get
        value+","+age
      }
    }).withBroadcastSet(toBroadcastData,"broadcastMapName")


  result.print()

  }

}
```
#### 20.Flink Counters-java代码

Accumulator即累加器，与Mapreduce counter的应用场景差不多，都能很好地观察task在运行期间的数据变化
可以在Flink job任务中的算子函数中操作累加器，但是只能在任务执行结束之后才能获得累加器的最终结果。
Counter是一个具体的累加器(Accumulator)实现
IntCounter, LongCounter 和 DoubleCounter
用法
1：创建累加器
private IntCounter numLines = new IntCounter(); 
2：注册累加器
getRuntimeContext().addAccumulator("num-lines", this.numLines);
3：使用累加器
this.numLines.add(1); 
4：获取累加器的结果
myJobExecutionResult.getAccumulatorResult("num-lines")		

```java
/**
 * 全局累加器
 *
 * counter 计数器
 *
 * 需求：
 * 计算map函数中处理了多少数据
 *
 *
 * 注意：只有在任务执行结束后，才能获取到累加器的值
 *
 *
 *
 * Created by xuwei.tech on 2018/10/8.
 */
public class BatchDemoCounter {

    public static void main(String[] args) throws Exception{

        //获取运行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        DataSource<String> data = env.fromElements("a", "b", "c", "d");

        DataSet<String> result = data.map(new RichMapFunction<String, String>() {

            //1:创建累加器
           private IntCounter numLines = new IntCounter();

            @Override
            public void open(Configuration parameters) throws Exception {
                super.open(parameters);
                //2:注册累加器
                getRuntimeContext().addAccumulator("num-lines",this.numLines);

            }

            //int sum = 0;
            @Override
            public String map(String value) throws Exception {
                //如果并行度为1，使用普通的累加求和即可，但是设置多个并行度，则普通的累加求和结果就不准了
                //sum++;
                //System.out.println("sum："+sum);
                this.numLines.add(1);
                return value;
            }
        }).setParallelism(8);

        //result.print();

        result.writeAsText("d:\\data\\count10");

        JobExecutionResult jobResult = env.execute("counter");
        //3：获取累加器
        int num = jobResult.getAccumulatorResult("num-lines");
        System.out.println("num:"+num);

    }



}
```
#### 21.Flink Counters-scala代码		
```scala
/**
  * counter 累加器
  */
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
```
Broadcast(广播变量)允许程序员将一个只读的变量缓存在每台机器上，而不用在任务之间传递变量。广播变量可以进行共享，但是不可以进行修改
Accumulators(累加器)是可以在不同任务中对同一个变量进行累加操作。

#### 22.Flink Distributed Cache

Flink提供了一个分布式缓存，类似于hadoop，可以使用户在并行函数中很方便的读取本地文件
此缓存的工作机制如下：程序注册一个文件或者目录(本地或者远程文件系统，例如hdfs或者s3)，通过ExecutionEnvironment注册缓存文件并为它起一个名称。当程序执行，Flink自动将文件或者目录复制到所有taskmanager节点的本地文件系统，用户可以通过这个指定的名称查找文件或者目录，然后从taskmanager节点的本地文件系统访问它
用法
1：注册一个文件
env.registerCachedFile("hdfs:///path/to/your/file", "hdfsFile")  
2：访问数据
File myFile = getRuntimeContext().getDistributedCache().getFile("hdfsFile");		

```java
/**
 * Distributed Cache
 */
public class BatchDemoDisCache {

    public static void main(String[] args) throws Exception{

        //获取运行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        //1：注册一个文件,可以使用hdfs或者s3上的文件
        env.registerCachedFile("d:\\data\\file\\a.txt","a.txt");

        DataSource<String> data = env.fromElements("a", "b", "c", "d");

        DataSet<String> result = data.map(new RichMapFunction<String, String>() {
            private ArrayList<String> dataList = new ArrayList<String>();

            @Override
            public void open(Configuration parameters) throws Exception {
                super.open(parameters);
                //2：使用文件
                File myFile = getRuntimeContext().getDistributedCache().getFile("a.txt");
                List<String> lines = FileUtils.readLines(myFile);
                for (String line : lines) {
                    this.dataList.add(line);
                    System.out.println("line:" + line);
                }
            }

            @Override
            public String map(String value) throws Exception {
                //在这里就可以使用dataList
                return value;
            }
        });

        result.print();


    }



}

/**
  * Distributed Cache
  * Created by xuwei.tech on 2018/10/30.
  */
object BatchDemoDisCacheScala {

  def main(args: Array[String]): Unit = {

    val env = ExecutionEnvironment.getExecutionEnvironment

    import org.apache.flink.api.scala._


    //1:注册文件
    env.registerCachedFile("d:\\data\\file\\a.txt","b.txt")

    val data = env.fromElements("a","b","c","d")

    val result = data.map(new RichMapFunction[String,String] {

      override def open(parameters: Configuration): Unit = {
        super.open(parameters)
        val myFile = getRuntimeContext.getDistributedCache.getFile("b.txt")
        val lines = FileUtils.readLines(myFile)
        val it = lines.iterator()
        while (it.hasNext){
          val line = it.next();
          println("line:"+line)
        }
      }
      override def map(value: String) = {
        value
      }
    })

    result.print()

  }

}
```
#### 23.state之keyedState分析	

我们前面写的word count的例子，没有包含状态管理。如果一个task在处理过程中挂掉了，那么它在内存中的状态都会丢失，所有的数据都需要重新计算。从容错和消息处理的语义上(at least once, exactly once)，Flink引入了state和checkpoint。
首先区分一下两个概念
state一般指一个具体的task/operator的状态【state数据默认保存在java的堆内存中】
而checkpoint【可以理解为checkpoint是把state数据持久化存储了】，则表示了一个Flink Job在一个特定时刻的一份全局状态快照，即包含了所有task/operator的状态
注意：task是Flink中执行的基本单位。operator指算子(transformation)。
State可以被记录，在失败的情况下数据还可以恢复
Flink中有两种基本类型的State
Keyed State
Operator State	



Keyed State和Operator State，可以以两种形式存在：
原始状态(raw state)
托管状态(managed state)
托管状态是由Flink框架管理的状态
而原始状态，由用户自行管理状态具体的数据结构，框架在做checkpoint的时候，使用byte[]来读写状态内容，对其内部数据结构一无所知。
通常在DataStream上的状态推荐使用托管的状态，当实现一个用户自定义的operator时，会使用到原始状态。

**Keyed State**

顾名思义，就是基于KeyedStream上的状态。这个状态是跟特定的key绑定的，对KeyedStream流上的每一个key，都对应一个state。
stream.keyBy(…)
保存state的数据结构
ValueState<T>:即类型为T的单值状态。这个状态与对应的key绑定，是最简单的状态了。它可以通过update方法更新状态值，通过value()方法获取状态值
ListState<T>:即key上的状态值为一个列表。可以通过add方法往列表中附加值；也可以通过get()方法返回一个Iterable<T>来遍历状态值
ReducingState<T>:这种状态通过用户传入的reduceFunction，每次调用add方法添加值的时候，会调用reduceFunction，最后合并到一个单一的状态值
MapState<UK, UV>:即状态值为一个map。用户通过put或putAll方法添加元素
需要注意的是，以上所述的State对象，仅仅用于与状态进行交互（更新、删除、清空等），而真正的状态值，有可能是存在内存、磁盘、或者其他分布式存储系统中。相当于我们只是持有了这个状态的句柄

```java

```
#### 24.state之operatorState分析

与Key无关的State，与Operator绑定的state，整个operator只对应一个state
保存state的数据结构
ListState<T>
举例来说，Flink中的Kafka Connector，就使用了operator state。它会在每个connector实例中，保存该实例中消费topic的所有(partition, offset)映射		

```java

```
#### 25.Flink checkPoint分析

依靠checkPoint机制
保证exactly-once
只能保证Flink系统内的exactly-once
对于source和sink需要依赖外部的组件一同保证

​	为了保证state的容错性，Flink需要对state进行checkpoint。
Checkpoint是Flink实现容错机制最核心的功能，它能够根据配置周期性地基于Stream中各个Operator/task的状态来生成快照，从而将这些状态数据定期持久化存储下来，当Flink程序一旦意外崩溃时，重新运行程序时可以有选择地从这些快照进行恢复，从而修正因为故障带来的程序数据异常
Flink的checkpoint机制可以与(stream和state)的持久化存储交互的前提：
持久化的source，它需要支持在一定时间内重放事件。这种sources的典型例子是持久化的消息队列（比如Apache Kafka，RabbitMQ等）或文件系统（比如HDFS，S3，GFS等）
用于state的持久化存储，例如分布式文件系统（比如HDFS，S3，GFS等）

默认checkpoint功能是disabled的，想要使用的时候需要先启用
checkpoint开启之后，默认的checkPointMode是Exactly-once
checkpoint的checkPointMode有两种，Exactly-once和At-least-once
Exactly-once对于大多数应用来说是最合适的。At-least-once可能用在某些延迟超低的应用程序（始终延迟为几毫秒）	

```java
public class SocketWindowWordCountJavaCheckPoint {

    public static void main(String[] args) throws Exception{
        //获取需要的端口号
        int port;
        try {
            ParameterTool parameterTool = ParameterTool.fromArgs(args);
            port = parameterTool.getInt("port");
        }catch (Exception e){
            System.err.println("No port set. use default port 9000--java");
            port = 9000;
        }

        //获取flink的运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // 每隔1000 ms进行启动一个检查点【设置checkpoint的周期】
        env.enableCheckpointing(1000);
        // 高级选项：
        // 设置模式为exactly-once （这是默认值）
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
        // 确保检查点之间有至少500 ms的间隔【checkpoint最小间隔】
        env.getCheckpointConfig().setMinPauseBetweenCheckpoints(500);
        // 检查点必须在一分钟内完成，或者被丢弃【checkpoint的超时时间】
        env.getCheckpointConfig().setCheckpointTimeout(60000);
        // 同一时间只允许进行一个检查点
        env.getCheckpointConfig().setMaxConcurrentCheckpoints(1);
        // 表示一旦Flink处理程序被cancel后，会保留Checkpoint数据，以便根据实际需要恢复到指定的Checkpoint【详细解释见备注】
        //ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION:表示一旦Flink处理程序被cancel后，会保留Checkpoint数据，以便根据实际需要恢复到指定的Checkpoint
        //ExternalizedCheckpointCleanup.DELETE_ON_CANCELLATION: 表示一旦Flink处理程序被cancel后，会删除Checkpoint数据，只有job执行失败的时候才会保存checkpoint
        env.getCheckpointConfig().enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);


        //设置statebackend

        //env.setStateBackend(new MemoryStateBackend());
        //env.setStateBackend(new FsStateBackend("hdfs://hadoop100:9000/flink/checkpoints"));
        //env.setStateBackend(new RocksDBStateBackend("hdfs://hadoop100:9000/flink/checkpoints",true));

        String hostname = "hadoop100";
        String delimiter = "\n";
        //连接socket获取输入的数据
        DataStreamSource<String> text = env.socketTextStream(hostname, port, delimiter);

        // a a c

        // a 1
        // a 1
        // c 1
        DataStream<WordWithCount> windowCounts = text.flatMap(new FlatMapFunction<String, WordWithCount>() {
            public void flatMap(String value, Collector<WordWithCount> out) throws Exception {
                String[] splits = value.split("\\s");
                for (String word : splits) {
                    out.collect(new WordWithCount(word, 1L));
                }
            }
        }).keyBy("word")
                .timeWindow(Time.seconds(2), Time.seconds(1))//指定时间窗口大小为2秒，指定时间间隔为1秒
                .sum("count");//在这里使用sum或者reduce都可以
                /*.reduce(new ReduceFunction<WordWithCount>() {
                                    public WordWithCount reduce(WordWithCount a, WordWithCount b) throws Exception {

                                        return new WordWithCount(a.word,a.count+b.count);
                                    }
                                })*/
        //把数据打印到控制台并且设置并行度
        windowCounts.print().setParallelism(1);

        //这一行代码一定要实现，否则程序不执行
        env.execute("Socket window count");

    }

    public static class WordWithCount{
        public String word;
        public long count;
        public  WordWithCount(){}
        public WordWithCount(String word,long count){
            this.word = word;
            this.count = count;
        }
        @Override
        public String toString() {
            return "WordWithCount{" +
                    "word='" + word + '\'' +
                    ", count=" + count +
                    '}';
        }
    }



}
```
#### 26.Flink state backend详细分析

默认情况下，state会保存在taskmanager的内存中，checkpoint会存储在JobManager的内存中。
state 的store和checkpoint的位置取决于State Backend的配置
env.setStateBackend(…)
一共有三种State Backend
MemoryStateBackend
FsStateBackend
RocksDBStateBackend

------------

MemoryStateBackend
state数据保存在java堆内存中，执行checkpoint的时候，会把state的快照数据保存到jobmanager的内存中
基于内存的state backend在生产环境下不建议使用
FsStateBackend
state数据保存在taskmanager的内存中，执行checkpoint的时候，会把state的快照数据保存到配置的文件系统中
可以使用hdfs等分布式文件系统
RocksDBStateBackend
RocksDB跟上面的都略有不同，它会在本地文件系统中维护状态，state会直接写入本地rocksdb中。同时它需要配置一个远端的filesystem uri（一般是HDFS），在做checkpoint的时候，会把本地的数据直接复制到filesystem中。fail over的时候从filesystem中恢复到本地
RocksDB克服了state受内存限制的缺点，同时又能够持久化到远端文件系统中，比较适合在生产中使用	

-------------

修改State Backend的两种方式
第一种：单任务调整
修改当前任务代码
env.setStateBackend(new FsStateBackend("hdfs://namenode:9000/flink/checkpoints"));
或者new MemoryStateBackend()
或者new RocksDBStateBackend(filebackend, true);【需要添加第三方依赖】
第二种：全局调整
修改flink-conf.yaml
state.backend: filesystem
state.checkpoints.dir: hdfs://namenode:9000/flink/checkpoints
注意：state.backend的值可以是下面几种：jobmanager(MemoryStateBackend), filesystem(FsStateBackend), rocksdb(RocksDBStateBackend)	

```java

```
#### 27.Flink state backend实战演示		
```java
        //设置statebackend

        //env.setStateBackend(new MemoryStateBackend());
        //env.setStateBackend(new FsStateBackend("hdfs://hadoop100:9000/flink/checkpoints"));
        //env.setStateBackend(new RocksDBStateBackend("hdfs://hadoop100:9000/flink/checkpoints",true));
```
#### 28.Flink 重启策略分析

Flink支持不同的重启策略，以在故障发生时控制作业如何重启
集群在启动时会伴随一个默认的重启策略，在没有定义具体重启策略时会使用该默认策略。 如果在工作提交时指定了一个重启策略，该策略会覆盖集群的默认策略
默认的重启策略可以通过 Flink 的配置文件 flink-conf.yaml 指定。配置参数 restart-strategy 定义了哪个策略被使用。
常用的重启策略(配置依次如下)
固定间隔 (Fixed delay)
失败率 (Failure rate)
无重启 (No restart)
如果没有启用 checkpointing，则使用无重启 (no restart) 策略。 
如果启用了 checkpointing，但没有配置重启策略，则使用固定间隔 (fixed-delay) 策略，其中 Integer.MAX_VALUE 参数是尝试重启次数
重启策略可以在flink-conf.yaml中配置，表示全局的配置。也可以在应用代码中动态指定，会覆盖全局配置		

```java

```
第一种：全局配置 flink-conf.yaml
restart-strategy: fixed-delay
restart-strategy.fixed-delay.attempts: 3
restart-strategy.fixed-delay.delay: 10 s
第二种：应用代码设置
env.setRestartStrategy(RestartStrategies.fixedDelayRestart(
  3, // 尝试重启的次数
  Time.of(10, TimeUnit.SECONDS) // 间隔
));

------------

第一种：全局配置 flink-conf.yaml
restart-strategy: failure-rate
restart-strategy.failure-rate.max-failures-per-interval: 3
restart-strategy.failure-rate.failure-rate-interval: 5 min
restart-strategy.failure-rate.delay: 10 s
第二种：应用代码设置
env.setRestartStrategy(RestartStrategies.failureRateRestart(
  3, // 一个时间段内的最大失败次数
  Time.of(5, TimeUnit.MINUTES), // 衡量失败次数的是时间段
  Time.of(10, TimeUnit.SECONDS) // 间隔
));

-----------

第一种：全局配置 flink-conf.yaml
restart-strategy: none
第二种：应用代码设置
env.setRestartStrategy(RestartStrategies.noRestart());

#### 29.Flink 从checkpoint恢复数据	

默认情况下，如果设置了Checkpoint选项，则Flink只保留最近成功生成的1个Checkpoint，而当Flink程序失败时，可以从最近的这个Checkpoint来进行恢复。但是，如果我们希望保留多个Checkpoint，并能够根据实际需要选择其中一个进行恢复，这样会更加灵活，比如，我们发现最近4个小时数据记录处理有问题，希望将整个状态还原到4小时之前
Flink可以支持保留多个Checkpoint，需要在Flink的配置文件conf/flink-conf.yaml中，添加如下配置，指定最多需要保存Checkpoint的个数
state.checkpoints.num-retained: 20
这样设置以后就查看对应的Checkpoint在HDFS上存储的文件目录
hdfs dfs -ls hdfs://namenode:9000/flink/checkpoints
如果希望回退到某个Checkpoint点，只需要指定对应的某个Checkpoint路径即可实现	

-------------

如果Flink程序异常失败，或者最近一段时间内数据处理错误，我们可以将程序从某一个Checkpoint点进行恢复
bin/flink run -s hdfs://namenode:9000/flink/checkpoints/467e17d2cc343e6c56255d222bae3421/chk-56/_metadata flink-job.jar
程序正常运行后，还会按照Checkpoint配置进行运行，继续生成Checkpoint数据

```java

```
#### 30.Flink savePoint的使用详解1	

Flink通过Savepoint功能可以做到程序升级后，继续从升级前的那个点开始执行计算，保证数据不中断
全局，一致性快照。可以保存数据源offset，operator操作状态等信息
可以从应用在过去任意做了savepoint的时刻开始继续消费

-----------------

checkPoint
应用定时触发，用于保存状态，会过期
内部应用失败重启的时候使用
savePoint
用户手动执行，是指向Checkpoint的指针，不会过期
在升级的情况下使用
注意：为了能够在作业的不同版本之间以及 Flink 的不同版本之间顺利升级，强烈推荐程序员通过 uid(String) 方法手动的给算子赋予 ID，这些 ID 将用于确定每一个算子的状态范围。如果不手动给各算子指定 ID，则会由 Flink 自动给每个算子生成一个 ID。只要这些 ID 没有改变就能从保存点（savepoint）将程序恢复回来。而这些自动生成的 ID 依赖于程序的结构，并且对代码的更改是很敏感的。因此，强烈建议用户手动的设置 ID。

----------------

1：在flink-conf.yaml中配置Savepoint存储位置
不是必须设置，但是设置后，后面创建指定Job的Savepoint时，可以不用在手动执行命令时指定Savepoint的位置
state.savepoints.dir: hdfs://namenode:9000/flink/savepoints
2：触发一个savepoint【直接触发或者在cancel的时候触发】
bin/flink savepoint jobId [targetDirectory] [-yid yarnAppId]【针对on yarn模式需要指定-yid参数】
bin/flink cancel -s [targetDirectory] jobId [-yid yarnAppId]【针对on yarn模式需要指定-yid参数】
3：从指定的savepoint启动job
bin/flink run -s savepointPath [runArgs]

```java

```
#### 31.Flink window详解

聚合事件（比如计数、求和）在流上的工作方式与批处理不同。
比如，对流中的所有元素进行计数是不可能的，因为通常流是无限的（无界的）。所以，流上的聚合需要由 window 来划定范围，比如 “计算过去的5分钟” ，或者 “最后100个元素的和” 。
window是一种可以把无限数据切割为有限数据块的手段
窗口可以是 时间驱动的 【Time Window】（比如：每30秒）或者 数据驱动的【Count Window】 （比如：每100个元素）。

------

窗口通常被区分为不同的类型:
tumbling windows：滚动窗口 【没有重叠】 
sliding windows：滑动窗口 【有重叠】
session windows：会话窗口 

----

增量聚合
全量聚合

------

增量聚合
窗口中每进入一条数据，就进行一次计算
reduce(reduceFunction)
aggregate(aggregateFunction)
sum(),min(),max()

---

全量聚合
等属于窗口的数据到齐，才开始进行聚合计算【可以实现对窗口内的数据进行排序等需求】
apply(windowFunction)
process(processWindowFunction)
processWindowFunction比windowFunction提供了更多的上下文信息。

```java
/**
 * window 增量聚合
 */
public class SocketDemoIncrAgg {

    public static void main(String[] args) throws Exception{
        //获取需要的端口号
        int port;
        try {
            ParameterTool parameterTool = ParameterTool.fromArgs(args);
            port = parameterTool.getInt("port");
        }catch (Exception e){
            System.err.println("No port set. use default port 9000--java");
            port = 9000;
        }

        //获取flink的运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        String hostname = "hadoop100";
        String delimiter = "\n";
        //连接socket获取输入的数据
        DataStreamSource<String> text = env.socketTextStream(hostname, port, delimiter);

        DataStream<Tuple2<Integer,Integer>> intData = text.map(new MapFunction<String, Tuple2<Integer,Integer>>() {
            @Override
            public Tuple2<Integer,Integer> map(String value) throws Exception {
                return new Tuple2<>(1,Integer.parseInt(value));
            }
        });

        intData.keyBy(0)
                .timeWindow(Time.seconds(5))
                .reduce(new ReduceFunction<Tuple2<Integer, Integer>>() {
                    @Override
                    public Tuple2<Integer, Integer> reduce(Tuple2<Integer, Integer> value1, Tuple2<Integer, Integer> value2) throws Exception {
                        System.out.println("执行reduce操作："+value1+","+value2);
                        return new Tuple2<>(value1.f0,value1.f1+value2.f1);
                    }
                }).print();


        //这一行代码一定要实现，否则程序不执行
        env.execute("Socket window count");

    }

}

/**
 * window 增量聚合
 * */
public class SocketDemoFullCount {

    public static void main(String[] args) throws Exception{
        //获取需要的端口号
        int port;
        try {
            ParameterTool parameterTool = ParameterTool.fromArgs(args);
            port = parameterTool.getInt("port");
        }catch (Exception e){
            System.err.println("No port set. use default port 9000--java");
            port = 9000;
        }

        //获取flink的运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        String hostname = "hadoop100";
        String delimiter = "\n";
        //连接socket获取输入的数据
        DataStreamSource<String> text = env.socketTextStream(hostname, port, delimiter);

        DataStream<Tuple2<Integer,Integer>> intData = text.map(new MapFunction<String, Tuple2<Integer,Integer>>() {
            @Override
            public Tuple2<Integer,Integer> map(String value) throws Exception {
                return new Tuple2<>(1,Integer.parseInt(value));
            }
        });

        intData.keyBy(0)
                .timeWindow(Time.seconds(5))
                .process(new ProcessWindowFunction<Tuple2<Integer,Integer>, String, Tuple, TimeWindow>() {
                    @Override
                    public void process(Tuple key, Context context, Iterable<Tuple2<Integer, Integer>> elements, Collector<String> out)
                            throws Exception {
                        System.out.println("执行process。。。");
                        long count = 0;
                        for(Tuple2<Integer,Integer> element: elements){
                            count++;
                        }
                        out.collect("window:"+context.window()+",count:"+count);
                    }
                }).print();


        //这一行代码一定要实现，否则程序不执行
        env.execute("Socket window count");

    }
}
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

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
### 1.5soucefunction

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

```



#### 05.算子操作-java		

```java

```
#### 06.partition-java		
```java

```
#### 07.sink-java		
```java

```
#### 08.source-scala		
```java

```
#### 09.算子操作-scala		
```java

```
#### 10.partition-scala		
```java

```
#### 11.sink-scala		
```java

```
#### 12.DataSet之算子操作-java-1		
```java

```
#### 13.DataSet之算子操作-java-2		
```java

```
#### 14.DataSet之partition-java		
```java

```
#### 15.DataSet之算子操作-scala-1		
```java

```
#### 16.DataSet之算子操作-scala-2		
```java

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

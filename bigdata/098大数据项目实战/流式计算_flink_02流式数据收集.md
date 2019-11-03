#### 29基于springboot+springcloud之2.0版本构建实时数据收集服务之注册中心代码编写1

```xml
	<modelVersion>4.0.0</modelVersion>
    <artifactId>eurekaserver</artifactId>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.2.RELEASE</version>
        <relativePath/>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

```yaml
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

spring:
  application:
    name: youfanRegiterCenter
```

```java
//主启动类一定要放在package下 否则无法启动 因为componentScan无法生效
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```
浏览器访问 http://localhost:8761/ 正常

#### 30基于springboot+springcloud之2.0版本构建实时数据收集服务之注册中心补充

#### 31基于springboot+springcloud之2.0版本构建实时数据收集服务之服务搭建代码编写


```xml
    <modelVersion>4.0.0</modelVersion>
    <groupId>flinkuser</groupId>
    <artifactId>infoservice</artifactId>


    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.2.RELEASE</version>
        <relativePath/>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.47</version>
        </dependency>
        <!--<dependency>-->
            <!--<groupId>flinkuser</groupId>-->
            <!--<artifactId>common</artifactId>-->
            <!--<version>1.0</version>-->
        <!--</dependency>-->
        <!--<dependency>-->
            <!--<groupId>org.springframework.kafka</groupId>-->
            <!--<artifactId>spring-kafka</artifactId>-->
        <!--</dependency>-->
   </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
```
```properties

server.port=8762

spring.application.name=infoservice

eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/


```

```java
@SpringBootApplication
@EnableEurekaClient
@EnableAutoConfiguration
public class InfoServiceApplication {
    public static void main(String[] args) {

        SpringApplication.run( InfoServiceApplication.class, args );
    }
}

@RestController
@RequestMapping("/info")
public class InfoController {

    @RequestMapping(value="hello",method = RequestMethod.GET)
    public String hello(HttpServletRequest request){
        String ip =request.getRemoteAddr();
        return "hello:"+ip+"success";
    }
}

```

1.启动注册中心 正常访问http://localhost:8761/ 

2.启动信息服务 正常注册到eurekaServer

3.访问http://localhost:8762/info/hello 正常返回结果

s
#### 32用户画像之基于springboot+springcloud之2.0版本构建实时数据收集服务代码编写
1.抽取一个common module
```xml
    <modelVersion>4.0.0</modelVersion>
    <groupId>flinkuser</groupId>
    <artifactId>common</artifactId>
    <version>1.0</version>

    <dependencies>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.8.1</version>
        </dependency>
        <dependency>
            <groupId>com.typesafe</groupId>
            <artifactId>config</artifactId>
            <version>1.2.1</version>
        </dependency>

    </dependencies>

```
把原来的四种日志移动到当前的common当中

-----------------

```java
public class ResultMessage {
    private String status;//状态 fail 、 success
    private String message;//消息内容
}

//InfoController 定义如下 方法 同时pom。xml文件开启如下的common依赖
@RequestMapping(value = "receivelog", method = RequestMethod.POST)
public String hellowolrd(String recevicelog, HttpServletRequest req) {
	if (StringUtils.isBlank(recevicelog)) {
		return null;
	}
	String[] rearrays = recevicelog.split(":", 2);
	String classname = rearrays[0];
	String data = rearrays[1];
	String resulmesage = "";

	if ("AttentionProductLog".equals(classname)) {
		AttentionProductLog attentionProductLog = JSONObject.parseObject(data, AttentionProductLog.class);
		resulmesage = JSONObject.toJSONString(attentionProductLog);
//            kafkaTemplate.send(attentionProductLogTopic,resulmesage+"##1##"+new Date().getTime());
	} else if ("BuyCartProductLog".equals(classname)) {
		BuyCartProductLog buyCartProductLog = JSONObject.parseObject(data, BuyCartProductLog.class);
		resulmesage = JSONObject.toJSONString(buyCartProductLog);
//            kafkaTemplate.send(buyCartProductLogTopic,resulmesage+"##1##"+new Date().getTime());
	} else if ("CollectProductLog".equals(classname)) {
		CollectProductLog collectProductLog = JSONObject.parseObject(data, CollectProductLog.class);
		resulmesage = JSONObject.toJSONString(collectProductLog);
//            kafkaTemplate.send(collectProductLogTopic,resulmesage+"##1##"+new Date().getTime());
	} else if ("ScanProductLog".equals(classname)) {
		ScanProductLog scanProductLog = JSONObject.parseObject(data, ScanProductLog.class);
		resulmesage = JSONObject.toJSONString(scanProductLog);
//            kafkaTemplate.send(scanProductLogTopic,resulmesage+"##1##"+new Date().getTime());
	}
	ResultMessage resultMessage = new ResultMessage();
	resultMessage.setMessage(resulmesage);
	resultMessage.setStatus("success");
	String result = JSONObject.toJSONString(resultMessage);
	return result;
}
```





#### 33用户画像之kafka环境搭建

linux环境安装kafka

```java

```
#### 34用户画像之实时收集服务整合kafka代码编写1

```xml
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
```

```properties
kafka.consumer.zookeeper.connect=192.168.80.134:2181
kafka.consumer.servers=192.168.80.134:9092
kafka.consumer.enable.auto.commit=true
kafka.consumer.session.timeout=6000
kafka.consumer.auto.commit.interval=100
kafka.consumer.auto.offset.reset=latest
kafka.consumer.topic=test
kafka.consumer.group.id=test
kafka.consumer.concurrency=10

kafka.producer.servers=192.168.80.134:9092
kafka.producer.retries=0
kafka.producer.batch.size=4096
kafka.producer.linger=1
kafka.producer.buffer.memory=40960

```

```java
@Configuration
@EnableKafka
public class KafkaProducerConfig {

    @Value("${kafka.producer.servers}")
    private String servers;
    @Value("${kafka.producer.retries}")
    private int retries;
    @Value("${kafka.producer.batch.size}")
    private int batchSize;
    @Value("${kafka.producer.linger}")
    private int linger;
    @Value("${kafka.producer.buffer.memory}")
    private int bufferMemory;

    public Map<String, Object> producerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, servers);
        props.put(ProducerConfig.RETRIES_CONFIG, retries);
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, batchSize);
        props.put(ProducerConfig.LINGER_MS_CONFIG, linger);
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, bufferMemory);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return props;
    }

    public ProducerFactory<String,String> producerFactory(){
        return new DefaultKafkaProducerFactory<>(producerConfigs());
    }
    @Bean
    public KafkaTemplate<String,String> kafkaTemplate(){
        return new KafkaTemplate<String, String>(producerFactory());
    }
}

```
#### 35用户画像之实时收集服务整合kafka代码编写2

```xml
		<!--读取配置文件properties-->
		<dependency>
            <groupId>com.typesafe</groupId>
            <artifactId>config</artifactId>
            <version>1.2.1</version>
        </dependency>
```

```properties
attentionProductLog=attentionProductLog
buyCartProductLog=buyCartProductLog
collectProductLog=collectProductLog
scanProductLog=scanProductLog
```

```java
public class ReadProperties {
    public final static Config config = ConfigFactory.load("test.properties");
    public static String getKey(String key){
        return config.getString(key).trim();
    }
    public static String getKey(String key,String filename){
        Config config =  ConfigFactory.load(filename);
        return config.getString(key).trim();
    }
}

//修改InfoController 添加如下的代码
    private final String attentionProductLogTopic = ReadProperties.getKey("attentionProductLog");
    private final String buyCartProductLogTopic = ReadProperties.getKey("buyCartProductLog");
    private final String collectProductLogTopic = ReadProperties.getKey("collectProductLog");
    private final String scanProductLogTopic = ReadProperties.getKey("scanProductLog");

	
    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

//使用kafkaProducer生产不同类型的主题
if ("AttentionProductLog".equals(classname)) {
            AttentionProductLog attentionProductLog = JSONObject.parseObject(data, AttentionProductLog.class);
            resulmesage = JSONObject.toJSONString(attentionProductLog);
            kafkaTemplate.send(attentionProductLogTopic, resulmesage + "##1##" + System.currentTimeMillis());
        } else if ("BuyCartProductLog".equals(classname)) {
            BuyCartProductLog buyCartProductLog = JSONObject.parseObject(data, BuyCartProductLog.class);
            resulmesage = JSONObject.toJSONString(buyCartProductLog);
            kafkaTemplate.send(buyCartProductLogTopic, resulmesage + "##1##" + System.currentTimeMillis());
        } else if ("CollectProductLog".equals(classname)) {
            CollectProductLog collectProductLog = JSONObject.parseObject(data, CollectProductLog.class);
            resulmesage = JSONObject.toJSONString(collectProductLog);
            kafkaTemplate.send(collectProductLogTopic, resulmesage + "##1##" + System.currentTimeMillis());
        } else if ("ScanProductLog".equals(classname)) {
            ScanProductLog scanProductLog = JSONObject.parseObject(data, ScanProductLog.class);
            resulmesage = JSONObject.toJSONString(scanProductLog);
            kafkaTemplate.send(scanProductLogTopic, resulmesage + "##1##" + System.currentTimeMillis());
        }

```
#### 36用户画像之实时品牌偏好设计以及代码编写实现实时更新用户品牌偏好

==1.flink整合kafka==

```xml
		<dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka-0.10_${scala.binary.version}</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-scala_2.11</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
            <version>${project.version}</version>
        </dependency>
```

2.kafkaEvent支持

```java
public class KafkaEvent {
    private final static String splitword = "##";
	private String word;
	private int frequency;
	private long timestamp;
    
    public KafkaEvent() {}

	public KafkaEvent(String word, int frequency, long timestamp) {
		this.word = word;
		this.frequency = frequency;
		this.timestamp = timestamp;
	}
    
    public static KafkaEvent fromString(String eventStr) {
		String[] split = eventStr.split(splitword);
		return new KafkaEvent(split[0], Integer.valueOf(split[1]), Long.valueOf(split[2]));
	}

	@Override
	public String toString() {
		return word +splitword + frequency + splitword + timestamp;
	}
}

public class KafkaEventSchema implements DeserializationSchema<KafkaEvent>, SerializationSchema<KafkaEvent> {

	private static final long serialVersionUID = 6154188370181669758L;

	@Override
	public byte[] serialize(KafkaEvent event) {
		return event.toString().getBytes();
	}

	@Override
	public KafkaEvent deserialize(byte[] message) throws IOException {
		return KafkaEvent.fromString(new String(message));
	}

	@Override
	public boolean isEndOfStream(KafkaEvent nextElement) {
		return false;
	}

	@Override
	public TypeInformation<KafkaEvent> getProducedType() {
		return TypeInformation.of(KafkaEvent.class);
	}
}

public class Kafka010Example {

	public static void main(String[] args) throws Exception {
		// parse input arguments
		args = new String[]{"--input-topic","test1","--output-topic","test2","--bootstrap.servers","192.168.80.134:9092","--zookeeper.connect","192.168.80.134:2181","--group.id","myconsumer"};
		final ParameterTool parameterTool = ParameterTool.fromArgs(args);

		if (parameterTool.getNumberOfParameters() < 5) {
			System.out.println("Missing parameters!\n" +
					"Usage: Kafka --input-topic <topic> --output-topic <topic> " +
					"--bootstrap.servers <kafka brokers> " +
					"--zookeeper.connect <zk quorum> --group.id <some id>");
			return;
		}

		StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
		env.getConfig().disableSysoutLogging();
		env.getConfig().setRestartStrategy(RestartStrategies.fixedDelayRestart(4, 10000));
		env.enableCheckpointing(5000); // create a checkpoint every 5 seconds
		env.getConfig().setGlobalJobParameters(parameterTool); // make parameters available in the web interface
		env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

		DataStream<KafkaEvent> input = env
				.addSource(
					new FlinkKafkaConsumer010<>(
						parameterTool.getRequired("input-topic"),
						new KafkaEventSchema(),
						parameterTool.getProperties())
					.assignTimestampsAndWatermarks(new CustomWatermarkExtractor()))
				.keyBy("word")
				.map(new RollingAdditionMapper());

		input.addSink(
				new FlinkKafkaProducer010<>(
						parameterTool.getRequired("output-topic"),
						new KafkaEventSchema(),
						parameterTool.getProperties()));

		env.execute("Kafka 0.10 Example");
	}


	private static class RollingAdditionMapper extends RichMapFunction<KafkaEvent, KafkaEvent> {

		private static final long serialVersionUID = 1180234853172462378L;

		private transient ValueState<Integer> currentTotalCount;

		@Override
		public KafkaEvent map(KafkaEvent event) throws Exception {
			Integer totalCount = currentTotalCount.value();
			System.out.println("哈哈--");
			if (totalCount == null) {
				totalCount = 0;
			}
			totalCount += event.getFrequency();

			currentTotalCount.update(totalCount);

			return new KafkaEvent(event.getWord(), totalCount, event.getTimestamp());
		}

		@Override
		public void open(Configuration parameters) throws Exception {
			currentTotalCount = getRuntimeContext().getState(new ValueStateDescriptor<>("currentTotalCount", Integer.class));
		}
	}

	private static class CustomWatermarkExtractor implements AssignerWithPeriodicWatermarks<KafkaEvent> {

		private static final long serialVersionUID = -742759155861320823L;

		private long currentTimestamp = Long.MIN_VALUE;

		@Override
		public long extractTimestamp(KafkaEvent event, long previousElementTimestamp) {
			// the inputs are assumed to be of format (message,timestamp)
			this.currentTimestamp = event.getTimestamp();
			return event.getTimestamp();
		}

		@Nullable
		@Override
		public Watermark getCurrentWatermark() {
			return new Watermark(currentTimestamp == Long.MIN_VALUE ? Long.MIN_VALUE : currentTimestamp - 1);
		}
	}
}
```
==2.brandlike品牌标签偏好开发==


#### 72用户画像之接口查询服务构建
```xml
 <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.2.RELEASE</version>
        <relativePath/>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.47</version>
        </dependency>
        <dependency>
            <groupId>flinkuser</groupId>
            <artifactId>common</artifactId>
            <version>1.0</version>
        </dependency>
        <dependency>
            <groupId>org.mongodb</groupId>
            <artifactId>mongodb-driver</artifactId>
            <version>3.6.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>1.2.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-server</artifactId>
            <!--<version>1.0.0-cdh5.5.1</version>-->
            <version>1.2.3</version>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
```
```properties
server.port=8763

spring.application.name=search

eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/


```

```java
@RestController
@RequestMapping("test")
public class Test {

    @RequestMapping(value = "helloworld",method = RequestMethod.GET)
    public String hellowolrd(HttpServletRequest req){
        String ip =req.getRemoteAddr();
        String result = "hello world from "+ ip;
        return result;
    }

}

@SpringBootApplication
@EnableEurekaClient
@EnableAutoConfiguration
public class SearchServiceApplication {
    public static void main(String[] args) {

        SpringApplication.run( SearchServiceApplication.class, args );
    }
}

```

启动注册中心

启动search项目

浏览器访问localhost:8763/test/helloworld

#### 73用户画像之年代接口代码编写

```java
public class BaseMongo {
    protected static MongoClient mongoClient ;
		
		static {
			List<ServerAddress> addresses = new ArrayList<ServerAddress>();
			String[] addressList = new String[]{"192.168.80.134"};
			String[] portList = new String[]{"27017"};
			for (int i = 0; i < addressList.length; i++) {
				ServerAddress address = new ServerAddress(addressList[i], Integer.parseInt(portList[i]));
				addresses.add(address);
				
			}
			mongoClient = new MongoClient(addresses);
		}
	
}

//YearSearchBaseImpl YearSearchBaseControl 被整合到后面的统一的查询接口当中
```
#### 74用户画像之前端查询服务构建
```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.2.RELEASE</version>
        <relativePath/>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.47</version>
        </dependency>
        <dependency>
            <groupId>flinkuser</groupId>
            <artifactId>common</artifactId>
            <version>1.0</version>
        </dependency>
         <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
```

```properties
server.port=8764

spring.application.name=view

eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/


```

```java
@RestController
@RequestMapping("test")
public class Test {

    @RequestMapping(value = "helloworld",method = RequestMethod.GET)
    public String hellowolrd(HttpServletRequest req){
        String ip =req.getRemoteAddr();
        String result = "hello world from "+ ip;
        return result;
    }

}

@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
@EnableDiscoveryClient
@EnableAutoConfiguration
public class SearchServiceApplication {
    public static void main(String[] args) {

        SpringApplication.run( SearchServiceApplication.class, args );
    }
}

```

启动注册中心

启动search项目

浏览器访问localhost:8764/test/helloworld
#### 75用户画像之基于springcloud+Feign服务调用代码编写

```java
//YearBaseService 后面整合到其他的类中
```
#### 76用户画像之基于springcloud+Feign服务调用代码编写2
```java
//YearBaseController 后面整合到其他的类中
```
#### 77用户画像之vuejs整合前端查询接口代码编写
```javascript
//baseyear.vue
<template>
    <div>
      <x-chart id="high" class="high" :option="option"></x-chart>
    </div>
</template>

<script>
  // 导入chart组件
  var myvue = {};
  import XChart from './charts'
  export default {
    data() {
      return {
        option:{
          chart: {
            type: 'column'
          },
          title: {
            text: '月平均降雨量'
          },
          subtitle: {
            text: '数据来源: WorldClimate.com'
          },
          xAxis: {
            categories: [
              '一月','二月','三月','四月','五月','六月','七月','八月','九月','十月','十一月','十二月'
            ],
            crosshair: true
          },
          yAxis: {
            min: 0,
            title: {
              text: '降雨量 (mm)'
            }
          },
          series: [{
            name: '东京',
            data: [49.9, 71.5, 106.4, 129.2, 144.0, 176.0, 135.6, 148.5,500, 194.1, 95.6, 54.4]
          }]
        },
      }
    },
    beforeCreate:function(){
      myvue = this;
    },
    created() {
      //新增用户
      this.$http.post('http://127.0.0.1:8764/mongoData/resultinfoView',{
          "type": "yearBase"
        }).then((response) => {
          this.option = {
                        chart: {
                          type: 'column'
                        },
                        title: {
                          text: '年代趋势'
                        },
                        xAxis: {
                          categories: response.body.infolist,
                          crosshair: true
                        },
                        yAxis: {
                          min: 0,
                          title: {
                            text: '数量'
                          }
                        },
                        series: [{
                          name: '年代',
                          data: response.body.countlist
                        }]
          };
      });
    },
    components: {
      XChart
    }
  }
</script>

<style scoped>

</style>

```
#### 78用户画像之vuejs整合前段查询接口之跨域问题解决
```java
//@CrossOrigin
//contoller添加@CrossOrigin处理器浏览器的跨域问题
```
#### 79用户画像之前端查询接口进一步封装代码编写
```java
//动态遍历结果集 response.body.infolist
```
#### 80用户画像之接口重构代码编写
```java
public class AnalyResult {
    private String info;//分组条件
    private Long count;//总数
}

@Service
public class MongoDataServiceImpl extends BaseMongo {


    public List<AnalyResult> listMongoInfoby(String tablename) {
        List<AnalyResult> result = new ArrayList<AnalyResult>();


        MongoDatabase db = mongoClient.getDatabase("youfanPortrait");
        MongoCollection<Document> collection =  db.getCollection(tablename);


        Document groupFields = new Document();
        Document idFields = new Document();
        idFields.put("info", "$info");
        groupFields.put("_id", idFields);
        groupFields.put("count", new Document("$sum", "$count"));

        Document group = new Document("$group", groupFields);


        Document projectFields = new Document();
        projectFields.put("_id", false);
        projectFields.put("info", "$_id.info");
        projectFields.put("count", true);
        Document project = new Document("$project", projectFields);
        AggregateIterable<Document> iterater = collection.aggregate(
                (List<Document>) Arrays.asList(group, project)
        );

        MongoCursor<Document> cursor = iterater.iterator();
        while(cursor.hasNext()){
            Document document = cursor.next();
            String jsonString = JSONObject.toJSONString(document);
            AnalyResult brandUser = JSONObject.parseObject(jsonString,AnalyResult.class);
            result.add(brandUser);
        }
        return result;
    }
}

@RestController
@RequestMapping("yearBase")
public class MongodataControl {

    @Autowired
    private MongoDataServiceImpl mongoDataServiceImpl;

    @RequestMapping(value = "searchYearBase",method = RequestMethod.POST)
    public List<AnalyResult> searchYearBase(){
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        AnalyResult analyResult = new AnalyResult();
        //40年代，50年代，60年代，70年代，80年代，90年代，00年代 10后
        analyResult.setCount(50l);
        analyResult.setInfo("40年代");
        list.add(analyResult);
        analyResult = new AnalyResult();
        analyResult.setCount(60l);
        analyResult.setInfo("50年代");
        list.add(analyResult);
        analyResult = new AnalyResult();
        analyResult.setCount(100l);
        analyResult.setInfo("60年代");
        list.add(analyResult);
        analyResult = new AnalyResult();
        analyResult.setCount(90l);
        analyResult.setInfo("70年代");
        list.add(analyResult);
        analyResult = new AnalyResult();
        analyResult.setCount(500l);
        analyResult.setInfo("80年代");
        list.add(analyResult);
        analyResult = new AnalyResult();
        analyResult.setCount(600l);
        analyResult.setInfo("90年代");
        list.add(analyResult);
        analyResult = new AnalyResult();
        analyResult.setCount(300l);
        analyResult.setInfo("00年代");
        list.add(analyResult);
        analyResult = new AnalyResult();
        analyResult.setCount(70l);
        analyResult.setInfo("10后");

        list.add(analyResult);

        return list;
//        return mongoDataServiceImpl.listMongoInfoby("yearbasestatics");
    }

    @RequestMapping(value = "searchUseType",method = RequestMethod.POST)
    public List<AnalyResult> searchUseType(){
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        AnalyResult analyResult = new AnalyResult();
        //pc端，小程序端，移动端
        analyResult.setCount(50l);
        analyResult.setInfo("pc端");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(60l);
        analyResult.setInfo("小程序端");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(40l);
        analyResult.setInfo("移动端");
        list.add(analyResult);

        return list;
//        return mongoDataServiceImpl.listMongoInfoby("usetypestatics");
    }

    @RequestMapping(value = "searchEmail",method = RequestMethod.POST)
    public List<AnalyResult> searchEmail(){
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        AnalyResult analyResult = new AnalyResult();
        //qq邮箱，139邮箱，网易邮箱,阿里邮箱
        analyResult.setCount(150l);
        analyResult.setInfo("qq邮箱");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(60l);
        analyResult.setInfo("139邮箱");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(240l);
        analyResult.setInfo("网易邮箱");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(540l);
        analyResult.setInfo("阿里邮箱");
        list.add(analyResult);

        return list;

//        return mongoDataServiceImpl.listMongoInfoby("emailstatics");
    }

    @RequestMapping(value = "searchConsumptionlevel",method = RequestMethod.POST)
    public List<AnalyResult> searchConsumptionlevel(){
        //高消费 中等消费  低消费
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        AnalyResult analyResult = new AnalyResult();
        //qq邮箱，139邮箱，网易邮箱,阿里邮箱
        analyResult.setCount(50l);
        analyResult.setInfo("高消费");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(560l);
        analyResult.setInfo("中等消费");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(760l);
        analyResult.setInfo("低消费");
        list.add(analyResult);

        return list;

//        return mongoDataServiceImpl.listMongoInfoby("consumptionlevelstatics");
    }

    @RequestMapping(value = "searchChaoManAndWomen",method = RequestMethod.POST)
    public List<AnalyResult> searchChaoManAndWomen(){
        //潮男 潮女
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        AnalyResult analyResult = new AnalyResult();

        analyResult.setCount(350l);
        analyResult.setInfo("潮男");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(560l);
        analyResult.setInfo("潮女");
        list.add(analyResult);

        return list;

//        return mongoDataServiceImpl.listMongoInfoby("chaoManAndWomenstatics");
    }

    @RequestMapping(value = "searchCarrier",method = RequestMethod.POST)
    public List<AnalyResult> searchCarrier(){
        //联通 移动 电信 其他
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        AnalyResult analyResult = new AnalyResult();

        analyResult.setCount(1350l);
        analyResult.setInfo("联通");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(1560l);
        analyResult.setInfo("移动");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(560l);
        analyResult.setInfo("电信");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(4560l);
        analyResult.setInfo("其他");
        list.add(analyResult);

        return list;
//        return mongoDataServiceImpl.listMongoInfoby("carrierstatics");
    }

    @RequestMapping(value = "searchBrandlike",method = RequestMethod.POST)
    public List<AnalyResult> searchBrandlike(){
        //李宁 爱迪达斯 森马 海尔
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        AnalyResult analyResult = new AnalyResult();

        analyResult.setCount(1350l);
        analyResult.setInfo("李宁");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(1560l);
        analyResult.setInfo("爱迪达斯");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(560l);
        analyResult.setInfo("森马");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(4560l);
        analyResult.setInfo("海尔");
        list.add(analyResult);

        return list;

//        return mongoDataServiceImpl.listMongoInfoby("brandlikestatics");
    }

}

```
#### 81用户画像之前端查询接口重用改造代码编写
```java
@FeignClient(value = "youfanSearchInfo")
public interface MongoDataService {

    @RequestMapping(value = "yearBase/searchYearBase",method = RequestMethod.POST)
    public List<AnalyResult> searchYearBase();

    @RequestMapping(value = "yearBase/searchUseType",method = RequestMethod.POST)
    public List<AnalyResult> searchUseType();

    @RequestMapping(value = "yearBase/searchEmail",method = RequestMethod.POST)
    public List<AnalyResult> searchEmail();

    @RequestMapping(value = "yearBase/searchConsumptionlevel",method = RequestMethod.POST)
    public List<AnalyResult> searchConsumptionlevel();

    @RequestMapping(value = "yearBase/searchChaoManAndWomen",method = RequestMethod.POST)
    public List<AnalyResult> searchChaoManAndWomen();

    @RequestMapping(value = "yearBase/searchCarrier",method = RequestMethod.POST)
    public List<AnalyResult> searchCarrier();

    @RequestMapping(value = "yearBase/searchBrandlike",method = RequestMethod.POST)
    public List<AnalyResult> searchBrandlike();
}

@RestController
@RequestMapping("mongoData")
@CrossOrigin
public class MongoDataViewControl {

    @Autowired
    MongoDataService mongoDataService;

    @RequestMapping(value = "resultinfoView",method = RequestMethod.POST,produces = "application/json;charset=UTF-8")
    public String resultinfoView(@RequestBody AnalyForm analyForm){
        String type = analyForm.getType();
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        if("yearBase".equals(type)){
            list = mongoDataService.searchYearBase();
        }else if ("useType".equals(type)){
            list = mongoDataService.searchUseType();
        }else if ("email".equals(type)){
            list = mongoDataService.searchEmail();
        }else if ("consumptionlevel".equals(type)){
            list = mongoDataService.searchConsumptionlevel();
        }else if ("carrier".equals(type)){
            list = mongoDataService.searchCarrier();
        }else if ("chaoManAndWomen".equals(type)){
            list = mongoDataService.searchChaoManAndWomen();
        }else if ("brandlike".equals(type)){
            list = mongoDataService.searchBrandlike();
        }
        ViewResultAnaly viewResultAnaly = new ViewResultAnaly();
        List<String> infolist = new ArrayList<String>();//分组list，x轴的值
        List<Long> countlist =new ArrayList<Long>();//数量
        for(AnalyResult analyResult:list){
            infolist.add(analyResult.getInfo());
            countlist.add(analyResult.getCount());
        }
        viewResultAnaly.setInfolist(infolist);
        viewResultAnaly.setCountlist(countlist);
        String result = JSONObject.toJSONString(viewResultAnaly);
        return result;
    }
}

@RestController
@RequestMapping("mongoData")
@CrossOrigin
public class MongoDataViewControl {

    @Autowired
    MongoDataViewService mongoDataService;

    @RequestMapping(value = "resultinfoView",method = RequestMethod.POST,produces = "application/json;charset=UTF-8")
    public String resultinfoView(@RequestBody AnalyForm analyForm){
        String type = analyForm.getType();
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        if("yearBase".equals(type)){
            list = mongoDataService.searchYearBase();
        }else if ("useType".equals(type)){
            list = mongoDataService.searchUseType();
        }else if ("email".equals(type)){
            list = mongoDataService.searchEmail();
        }else if ("consumptionlevel".equals(type)){
            list = mongoDataService.searchConsumptionlevel();
        }else if ("carrier".equals(type)){
            list = mongoDataService.searchCarrier();
        }else if ("chaoManAndWomen".equals(type)){
            list = mongoDataService.searchChaoManAndWomen();
        }else if ("brandlike".equals(type)){
            list = mongoDataService.searchBrandlike();
        }
        ViewResultAnaly viewResultAnaly = new ViewResultAnaly();
        List<String> infolist = new ArrayList<String>();//分组list，x轴的值
        List<Long> countlist =new ArrayList<Long>();//数量
        for(AnalyResult analyResult:list){
            infolist.add(analyResult.getInfo());
            countlist.add(analyResult.getCount());
        }
        viewResultAnaly.setInfolist(infolist);
        viewResultAnaly.setCountlist(countlist);
        String result = JSONObject.toJSONString(viewResultAnaly);
        return result;
    }
}

public class AnalyForm {
    private String type;
    private String userid;
}

public class ViewResultAnaly {
    private List<String> infolist;//分组list，x轴的值
    private List<Long> countlist;//数量list
    private String result;
    private String typename;//标签类型名称
    private String lablevalue;//标签类型对应的值
}    
```
#### 82用户画像vuejs完善剩余图表代码编写1
```java

```
#### 83用户画像之vuejs完善剩余图表代码编写2
```java

```
#### 84用户画像之vuejs完善剩余图表代码编写3
```java

```
#### 85用户画像之vuejs配置路由代码编写
```java

```
#### 86用户画像之接口服务前端查询服务以及前端展示服务联调以及效果展示.zip
```java

```
#### 87用户画像之TF-IDF通俗讲解
```java

```
#### 88用户画像之分词工具ik讲解以及代码编写.zip
```java

```
#### 89用户画像之java实现TF-IDF代码编写1
```java

```
#### 90用户画像之java实现TF-IDF代码编写2
```java

```
#### 91用户画像之flink实现分布式TF-IDF代码编写1
```java

```
#### 92用户画像之flink实现分布式TF-IDF代码编写2、
```java

```
#### 93用户画像之fink分布式TF-IDF实现用户年度、月度，季度商品关键词代码编写1
```java

```
#### 94用户画像之fink分布式TF-IDF实现用户年度、月度，季度商品关键词代码编写2
```java

```
#### 95用户画像之fink分布式TF-IDF实现用户年度、月度，季度商品关键词代码编写3
```java

```
#### 96用户画像之fink分布式TF-IDF实现用户年度、月度，季度商品关键词代码编写4
```java

```
#### 97用户画像之标签接口之败家指数接口代码编写
```java

```
#### 98用户画像之全部标签接口代码编写
```java

```
#### 99用户画像之前端标签查询服务代码编写
```java

```
#### 100用户画像之vue.js标签显示代码编写1
```java

```
#### 101用户画像之vue.js标签显示代码编写2以及效果演示
```java

```

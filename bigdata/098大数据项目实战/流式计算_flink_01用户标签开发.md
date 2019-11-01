### 1
#### 01课程介绍

#### 02项目价值说明

#### 03项目架构讲解

#### 04数据来源说明

#### 05静态信息和动态信息说明

#### 06用户画像之还原真实场景表结构定义讲解

创建表

user_info信息表
user_detail表

用户表：用户ID、用户名、密码、性别、手机号、邮箱、年龄、注册时间、收货地址、终端类型

用户详情补充表：学历、收入、职业、婚姻、是否有小孩、是否有车有房、使用手机品牌、用户id


#### 07用户画像之flink画像分析模块项目构建

1.创建一个父类的工程flinkuser

2.创建一个module工程

3.项目加入如下的依赖

```xml
<modelVersion>4.0.0</modelVersion>
    <artifactId>service</artifactId>
    <parent>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-examples</artifactId>
        <version>1.7.0</version>
    </parent>

    <properties>
        <scala.version>2.11.12</scala.version>
        <scala.binary.version>2.11</scala.binary.version>
    </properties>


    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-scala_2.11</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
```
4.编写一个类

```java
public class Test {
    public static void main(String[] args) {
        //把main方法的参数设置为params
        final ParameterTool params = ParameterTool.fromArgs(args);
        //设置启动环境
        final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //设置参数为全局参数
        env.getConfig().setGlobalJobParameters(params);
        //文件输入
        DataSet<String> text = env.readTextFile(params.get("input"));
        //
        DataSet map = text.flatMap(null);
        DataSet reduce = map.groupBy("groupbyfield").reduce(null);
        try {
            env.execute("test");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### 08用户画像之hadoop环境搭建

1.安装jdk1.8

2.安装hadoop2.6

```java

```
#### 09用户画像之hbase环境搭建

1.安装zk3.4.5

2.安装hbase1.0.0

```java

```
#### 10用户画像之mongo环境搭建

mongodb安装

```java

```
#### 11用户画像之年代标签代码编写1
```java
public class YearBaseEntity {
    private String yeartype;//年代类型
    private Long count;//数量
    private String groupfield;//分组字段
 	//省去get/set方法   
}

public class DateUtils {
    public static String getYearbasebyAge(String age){
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(new Date());
        calendar.add(Calendar.YEAR,-Integer.valueOf(age));
        Date newdate = calendar.getTime();
        DateFormat dateFormat = new SimpleDateFormat("yyyy");
        String newdatestring = dateFormat.format(newdate);
        Integer newdateinteger = Integer.valueOf(newdatestring);
        String yearbasetype = "未知";
        if(newdateinteger >= 1940 && newdateinteger < 1950){
            yearbasetype = "40后";
        }else if (newdateinteger >= 1950 && newdateinteger < 1960){
            yearbasetype = "50后";
        }else if (newdateinteger >= 1960 && newdateinteger < 1970){
            yearbasetype = "60后";
        }else if (newdateinteger >= 1970 && newdateinteger < 1980){
            yearbasetype = "70后";
        }else if (newdateinteger >= 1980 && newdateinteger < 1990){
            yearbasetype = "80后";
        }else if (newdateinteger >= 1990 && newdateinteger < 2000){
            yearbasetype = "90后";
        }else if (newdateinteger >= 2000 && newdateinteger < 2010){
            yearbasetype = "00后";
        }else if (newdateinteger >= 2010 ){
            yearbasetype = "10后";
        }
        return yearbasetype;
    }

    public static int getDaysBetweenbyStartAndend(String starttime,String endTime,String dateFormatstring) throws ParseException {
        DateFormat dateFormat = new SimpleDateFormat(dateFormatstring);
        Date start = dateFormat.parse(starttime);
        Date end = dateFormat.parse(endTime);
        Calendar startcalendar = Calendar.getInstance();
        Calendar endcalendar = Calendar.getInstance();
        startcalendar.setTime(start);
        endcalendar.setTime(end);
        int days = 0;
        while(startcalendar.before(endcalendar)){
            startcalendar.add(Calendar.DAY_OF_YEAR,1);
            days += 1;
        }
        return days;
    }

    public static String gethoursbydate(String timevalue) throws ParseException {
        DateFormat dateFormat = new SimpleDateFormat("yyyyMMdd hhmmss");
        Date time = dateFormat.parse(timevalue);
        dateFormat = new SimpleDateFormat("hh");
        String resulthour = dateFormat.format(time);
        return resulthour;
    }
}


public class YearBaseMap implements MapFunction<String,YearBaseEntity>{

    @Override
    public YearBaseEntity map(String s) throws Exception {
        if(StringUtils.isBlank(s)){
            return null;
        }
        String[] userinfos = s.split(",");
        String userid = userinfos[0];
        String username = userinfos[1];
        String sex = userinfos[2];
        String telphone = userinfos[3];
        String email = userinfos[4];
        String age = userinfos[5];
        String registerTime = userinfos[6];
        String usetype = userinfos[7];//'终端类型：0、pc端；1、移动端；2、小程序端'
        String yearBaseType = DateUtils.getYearbasebyAge(age);
        String tableName = "userflaginfo";
        String rowKey = userid;
        String familyName ="baseInfo";
        String column = "yearbase";
        HbaseUtils.putData(tableName,rowKey,familyName,column,yearBaseType);
        HbaseUtils.putData(tableName,rowKey,familyName,"age",age);
        YearBaseEntity yearBase = new YearBaseEntity();
        String groupField = "yearbase=="+yearBaseType;
        yearBase.setYeartype(yearBaseType);
        yearBase.setCount(1L);
        yearBase.setGroupfield(groupField);
        return yearBase;
    }
}

public class YearBaseTask {
    public static void main(String[] args) {
        final ParameterTool params = ParameterTool.fromArgs(args);
        final ExecutionEnvironment env =
                ExecutionEnvironment.getExecutionEnvironment();
        env.getConfig().setGlobalJobParameters(params);

        DataSet<String> text = env.readTextFile(params.get("input"));
        DataSet<YearBaseEntity> mapResult = text.map(new YearBaseMap());
        DataSet<YearBaseEntity> reduceResult = mapResult.groupBy("groupfield")
                .reduce(new YearBaseReduce());
        try {
            List<YearBaseEntity> resultList = reduceResult.collect();
            for (YearBaseEntity yearBaseEntity : resultList) {
                String yearType = yearBaseEntity.getYeartype();
                Long count = yearBaseEntity.getCount();
                Document document = MongoUtils.findOneBy("yearbasestatics", "test", yearType);
                if (document == null) {
                    document = new Document();
                    document.put("info", yearType);
                    document.put("count", count);
                } else {
                    Long countPre = document.getLong("count");
                    Long total = countPre + count;
                    document.put("count", total);
                }
                MongoUtils.saveOrUpdateMongo("yearbasestatics", "test", document);
            }
            env.execute("year base analyse");
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}
```
#### 12用户画像之flink结合hbase保存年代标签代码编写

```xml
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
```

```java
public class HbaseUtils {
    private static Admin admin = null;
    private static Connection conn = null;

    static {
        //创建 hbase配置对象
        Configuration conf = HBaseConfiguration.create();
        conf.set("hbase.rootdir", "hdfs://192.168.80.134:9000/hbase");
        //使用eclipse时必须添加这个，否则无法定位
        conf.set("hbase.zookeeper.quorum", "192.168.80.134");
        conf.set("hbase.client.scanner.timeout.period", "600000");
        conf.set("hbase.rpc.timeout", "600000");
        try {
            conn = ConnectionFactory.createConnection(conf);
            //得到管理程序
            admin = conn.getAdmin();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /*插入数据*/
    public static void put(String tableName, String rowKey, String familyName, Map<String, String> map) throws Exception {
        Table table = conn.getTable(TableName.valueOf(tableName));
        //把字符串转换为byte[]
        byte[] rowkeyByte = Bytes.toBytes(rowKey);
        Put put = new Put(rowkeyByte);
        if (map != null) {
            Set<Map.Entry<String, String>> set = map.entrySet();
            for (Map.Entry<String, String> entry : set) {
                String key = entry.getKey();
                Object value = entry.getValue();
                put.addColumn(Bytes.toBytes(familyName), Bytes.toBytes(key), Bytes.toBytes(value + ""));
            }
        }
        table.put(put);
        table.close();
        System.out.println("ok");
    }

    public static String getData(String tableName, String rowKey, String familyName, String column) throws Exception {
        Table table = conn.getTable(TableName.valueOf(tableName));
        //将字符串转换为byte[]
        byte[] rowKeyByte = Bytes.toBytes(rowKey);
        Get get = new Get(rowKeyByte);
        Result result = table.get(get);
        byte[] resultBytes = result.getValue(familyName.getBytes(), column.getBytes());
        if (resultBytes == null) {
            return null;
        }
        return new String(resultBytes);
    }

    public static void putData(String tableName, String rowKey, String familyName, String column, String data) throws Exception {
        Table table = conn.getTable(TableName.valueOf(tableName));
        Put put = new Put(rowKey.getBytes());
        put.addColumn(familyName.getBytes(), column.getBytes(), data.getBytes());
        table.put(put);
    }

}
```
#### 13用户画像之年代群体数量统计代码编写1
```java
public class YearBaseReduce implements ReduceFunction<YearBaseEntity> {

    @Override
    public YearBaseEntity reduce(YearBaseEntity yearBaseEntity, YearBaseEntity t1) throws Exception {
        String yearType = yearBaseEntity.getYeartype();
        Long count1 = yearBaseEntity.getCount();
        Long count2 = t1.getCount();
        YearBaseEntity entity = new YearBaseEntity();
        entity.setYeartype(yearType);
        entity.setCount(count1+count2);
        return entity;
    }
}
```
#### 14用户画像之flink结合mongo保存年代群体数量

```xml
		<dependency>
            <groupId>org.mongodb</groupId>
            <artifactId>mongodb-driver</artifactId>
            <version>3.0.1</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.47</version>
        </dependency>
```

```java
public class MongoUtils {

    private static MongoClient mongoClient = new MongoClient("192.168.80.134", 27017);

    public static Document findOneBy(String tableName, String dataBase, String yearBaseType) {
        MongoDatabase mongoDatabase = mongoClient.getDatabase(dataBase);
        MongoCollection collection = mongoDatabase.getCollection(tableName);
        Document document = new Document();
        FindIterable<Document> iterable = collection.find(document);
        MongoCursor<Document> cursor = iterable.iterator();
        if (cursor.hasNext()) {
            return cursor.next();
        } else {
            return null;
        }
    }

    public static void saveOrUpdateMongo(String tableName, String dataBase, Document document) {
        MongoDatabase mongoDatabase = mongoClient.getDatabase(dataBase);
        MongoCollection<Document> mongoCollection = mongoDatabase.getCollection(tableName);
        if (!document.containsKey("_id")) {
            ObjectId objectId = new ObjectId();
            document.put("_id", objectId);
            mongoCollection.insertOne(document);
        }
        Document matchDocument = new Document();
        String objectId = document.getString("_id").toString();
        matchDocument.put("_id", new ObjectId(objectId));
        FindIterable<Document> findIterable = mongoCollection.find(matchDocument);
        if (findIterable.iterator().hasNext()) {
            mongoCollection.updateOne(matchDocument, new Document("$set", document));
            System.out.println(JSONObject.toJSONString(document));
        } else {
            mongoCollection.insertOne(document);
            System.out.println(JSONObject.toJSONString(document));
        }

    }

}
```
#### 15用户画像之手机运营商标签代码编写1
```java
public class CarrierInfoUtils {
    /**
     * 中国电信号码格式验证 手机段： 133,153,180,181,189,177,1700,173,199
     **/
    private static final String CHINA_TELECOM_PATTERN = "(^1(33|53|77|73|99|8[019])\\d{8}$)|(^1700\\d{7}$)";

    /**
     * 中国联通号码格式验证 手机段：130,131,132,155,156,185,186,145,176,1709
     **/
    private static final String CHINA_UNICOM_PATTERN = "(^1(3[0-2]|4[5]|5[56]|7[6]|8[56])\\d{8}$)|(^1709\\d{7}$)";

    /**
     * 中国移动号码格式验证
     * 手机段：134,135,136,137,138,139,150,151,152,157,158,159,182,183,184,187,188,147,178,1705
     **/
    private static final String CHINA_MOBILE_PATTERN = "(^1(3[4-9]|4[7]|5[0-27-9]|7[8]|8[2-478])\\d{8}$)|(^1705\\d{7}$)";

    /**
     * 0、未知 1、移动 2、联通 3、电信
     *
     * @param telphone
     * @return
     */
    public static int getCarrierByTel(String telphone) {
        boolean b1 = telphone == null || telphone.trim().equals("") ? false : match(CHINA_MOBILE_PATTERN, telphone);
        if (b1) {
            return 1;
        }
        b1 = telphone == null || telphone.trim().equals("") ? false : match(CHINA_UNICOM_PATTERN, telphone);
        if (b1) {
            return 2;
        }
        b1 = telphone == null || telphone.trim().equals("") ? false : match(CHINA_TELECOM_PATTERN, telphone);
        if (b1) {
            return 3;
        }
        return 0;
    }

    /**
     * 匹配函数
     *
     * @param regex
     * @param tel
     * @return
     */
    private static boolean match(String regex, String tel) {
        return Pattern.matches(regex, tel);
    }
}
```
#### 16用户画像之手机运营商标签代码编写2
```java
public class CarrierInfoEntity {
    private String carrier;//运营商
    private Long count;//数量
    private String groupfield;//分组
}

public class CarrierInfoMap implements MapFunction<String, CarrierInfoEntity> {
    @Override
    public CarrierInfoEntity map(String s) throws Exception {
        if (StringUtils.isBlank(s)) {
            return null;
        }
        String[] userinfos = s.split(",");
        String userid = userinfos[0];
        String telphone = userinfos[3];

        int carriertype = CarrierInfoUtils.getCarrierByTel(telphone);
        String carriertypestring = carriertype == 0 ? "未知运营商" : carriertype == 1 ? "移动用户" : carriertype == 2 ? "联通用户" : "电信用户";

        String tablename = "userflaginfo";
        String rowkey = userid;
        String famliyname = "baseinfo";
        String colum = "carrierinfo";//运营商
        HbaseUtils.putData(tablename, rowkey, famliyname, colum, carriertypestring);
        CarrierInfoEntity carrierInfo = new CarrierInfoEntity();
        String groupfield = "carrierInfo==" + carriertype;
        carrierInfo.setCount(1L);
        carrierInfo.setCarrier(carriertypestring);
        carrierInfo.setGroupfield(groupfield);
        return carrierInfo;
    }
}

public class CarrierInfoReduce implements ReduceFunction<CarrierInfoEntity> {

    @Override
    public CarrierInfoEntity reduce(CarrierInfoEntity carrierInfo, CarrierInfoEntity t1) throws Exception {
        String carrier = carrierInfo.getCarrier();
        Long count1 = carrierInfo.getCount();
        Long count2 = t1.getCount();

        CarrierInfoEntity carrierInfofinal = new CarrierInfoEntity();
        carrierInfofinal.setCarrier(carrier);
        carrierInfofinal.setCount(count1 + count2);
        return carrierInfofinal;
    }
}

public class CarrierInfoTask {
    public static void main(String[] args) {
        final ParameterTool params = ParameterTool.fromArgs(args);

        // set up the execution environment
        final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        // make parameters available in the web interface
        env.getConfig().setGlobalJobParameters(params);

        // get input data
        DataSet<String> text = env.readTextFile(params.get("input"));

        DataSet<CarrierInfoEntity> mapresult = text.map(new CarrierInfoMap());
        DataSet<CarrierInfoEntity> reduceresutl = mapresult.groupBy("groupfield").reduce(new CarrierInfoReduce());
        try {
            List<CarrierInfoEntity> reusltlist = reduceresutl.collect();
            for (CarrierInfoEntity carrierInfo : reusltlist) {
                String carrier = carrierInfo.getCarrier();
                Long count = carrierInfo.getCount();

                Document doc = MongoUtils.findOneBy("carrierstatics", "test", carrier);
                if (doc == null) {
                    doc = new Document();
                    doc.put("info", carrier);
                    doc.put("count", count);
                } else {
                    Long countpre = doc.getLong("count");
                    Long total = countpre + count;
                    doc.put("count", total);
                }
                MongoUtils.saveOrUpdateMongo("carrierstatics", "test", doc);
            }
            env.execute("carrier analyse");
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}
```
#### 17用户画像之邮件运营商标签代码编写1
```java
public class EmailUtils {
  
    public static String getEmailType(String email) {
        String emailtye = "其他邮箱用户";
        if (email.contains("@163.com") || email.contains("@126.com")) {
            emailtye = "网易邮箱用户";
        } else if (email.contains("@139.com")) {
            emailtye = "移动邮箱用户";
        } else if (email.contains("@sohu.com")) {
            emailtye = "搜狐邮箱用户";
        } else if (email.contains("@qq.com")) {
            emailtye = "qq邮箱用户";
        } else if (email.contains("@189.cn")) {
            emailtye = "189邮箱用户";
        } else if (email.contains("@tom.com")) {
            emailtye = "tom邮箱用户";
        } else if (email.contains("@aliyun.com")) {
            emailtye = "阿里邮箱用户";
        } else if (email.contains("@sina.com")) {
            emailtye = "新浪邮箱用户";
        }
        return emailtye;
    }
}
```
#### 18用户画像之邮件运营商标签代码编写2
```java
public class EmailEntity {
    private String emailtype;//邮箱类型
    private Long count;//数量
    private String groupfield;//分组字段
}

public class EmailMap implements MapFunction<String, EmailEntity> {
    @Override
    public EmailEntity map(String s) throws Exception {
        if(StringUtils.isBlank(s)){
            return null;
        }
        String[] userinfos = s.split(",");
        String userid = userinfos[0];
        String email = userinfos[4];

        String emailtype = EmailUtils.getEmailType(email);

        String tablename = "userflaginfo";
        String rowkey = userid;
        String famliyname = "baseinfo";
        String colum = "emailinfo";//运营商
        HbaseUtils.putData(tablename,rowkey,famliyname,colum,emailtype);
        EmailEntity emailInfo = new EmailEntity();
        String groupfield = "emailInfo=="+emailtype;
        emailInfo.setEmailtype(emailtype);
        emailInfo.setCount(1L);
        emailInfo.setGroupfield(groupfield);
        return emailInfo;
    }
}

public class EmailReduce implements ReduceFunction<EmailEntity> {

    @Override
    public EmailEntity reduce(EmailEntity emaiInfo, EmailEntity t1) throws Exception {
        String emailtype = emaiInfo.getEmailtype();
        Long count1 = emaiInfo.getCount();

        Long count2 = t1.getCount();

        EmailEntity emaiInfofinal = new EmailEntity();
        emaiInfofinal.setEmailtype(emailtype);
        emaiInfofinal.setCount(count1 + count2);

        return emaiInfofinal;
    }
}

public class EmailTask {
    public static void main(String[] args) {
        final ParameterTool params = ParameterTool.fromArgs(args);

        // set up the execution environment
        final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        // make parameters available in the web interface
        env.getConfig().setGlobalJobParameters(params);

        // get input data
        DataSet<String> text = env.readTextFile(params.get("input"));

        DataSet<EmailEntity> mapresult = text.map(new EmailMap());
        DataSet<EmailEntity> reduceresutl = mapresult.groupBy("groupfield").reduce(new EmailReduce());
        try {
            List<EmailEntity> reusltlist = reduceresutl.collect();
            for (EmailEntity emaiInfo : reusltlist) {
                String emailtype = emaiInfo.getEmailtype();
                Long count = emaiInfo.getCount();

                Document doc = MongoUtils.findOneBy("emailstatics", "test", emailtype);
                if (doc == null) {
                    doc = new Document();
                    doc.put("info", emailtype);
                    doc.put("count", count);
                } else {
                    Long countpre = doc.getLong("count");
                    Long total = countpre + count;
                    doc.put("count", total);
                }
                MongoUtils.saveOrUpdateMongo("emailstatics", "test", doc);
            }
            env.execute("email analy");
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}
```
#### 19用户画像之还原真实消费信息表结构定义

分别创建如下的三张表

product_info
product_type
order_info

用户订单表：订单id 商品id，商品类别id，订单时间、支付时间、支付方式、使用优惠劵金额、退款金额
商品表
商品类别表

```java

```
#### 20用户画像之败家指数计算规则定义
```java

```
#### 21用户画像之败家指数代码编写1
```java
public class BaiJiaEntity {
    private String baijiatype;//败家指数区段：0-20 、20-50 、50-70、70-80、80-90、90-100
    private String userid;
    private String createtime;
    private String amount;
    private String paytype;
    private String paytime;
    private String paystatus;//0、未支付 1、已支付 2、已退款
    private String couponamount;
    private String totalamount;
    private String refundamount;
    private Long count;//数量
    private String groupfield;//分组

    private List<BaiJiaEntity> list;
    
}    
```
#### 22用户画像之败家指数代码编写2
```java
public class BaiJiaMap implements MapFunction<String, BaiJiaEntity> {
    @Override
    public BaiJiaEntity map(String s) throws Exception {
        if (StringUtils.isBlank(s)) {
            return null;
        }

        String[] orderinfos = s.split(",");
        String createtime = orderinfos[3];
        String amount = orderinfos[4];
        String paytype = orderinfos[5];
        String paytime = orderinfos[6];
        String paystatus = orderinfos[7];
        String couponamount = orderinfos[8];
        String totalamount = orderinfos[9];
        String refundamount = orderinfos[10];
        String userid = orderinfos[12];

        BaiJiaEntity baiJiaInfo = new BaiJiaEntity();
        baiJiaInfo.setUserid(userid);
        baiJiaInfo.setCreatetime(createtime);
        baiJiaInfo.setAmount(amount);
        baiJiaInfo.setPaytype(paytype);
        baiJiaInfo.setPaytime(paytime);
        baiJiaInfo.setPaystatus(paystatus);
        baiJiaInfo.setCouponamount(couponamount);
        baiJiaInfo.setTotalamount(totalamount);
        baiJiaInfo.setRefundamount(refundamount);
        String groupfield = "baijia==" + userid;
        baiJiaInfo.setGroupfield(groupfield);
        List<BaiJiaEntity> list = new ArrayList<BaiJiaEntity>();
        list.add(baiJiaInfo);
        return baiJiaInfo;
    }
}
```
#### 23用户画像之败家指数代码编写3
```java
public class BaiJiaReduce implements ReduceFunction<BaiJiaEntity> {

    @Override
    public BaiJiaEntity reduce(BaiJiaEntity baiJiaInfo, BaiJiaEntity t1) throws Exception {
        String userid = baiJiaInfo.getUserid();
        List<BaiJiaEntity> baijialist1 = baiJiaInfo.getList();
        List<BaiJiaEntity> baijialist2 = t1.getList();
        List<BaiJiaEntity> finallist = new ArrayList<BaiJiaEntity>();
        finallist.addAll(baijialist1);
        finallist.addAll(baijialist2);

        BaiJiaEntity baiJiaInfofinal = new BaiJiaEntity();
        baiJiaInfofinal.setUserid(userid);
        baiJiaInfofinal.setList(finallist);
        return baiJiaInfofinal;
    }
}
```
#### 24用户画像之败家指数代码编写4
```java
public class BaiJiaTask {
    public static void main(String[] args) {
        final ParameterTool params = ParameterTool.fromArgs(args);

        // set up the execution environment
        final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        // make parameters available in the web interface
        env.getConfig().setGlobalJobParameters(params);

        // get input data
        DataSet<String> text = env.readTextFile(params.get("input"));

        DataSet<BaiJiaEntity> mapresult = text.map(new BaiJiaMap());
        DataSet<BaiJiaEntity> reduceresutl = mapresult.groupBy("groupfield").reduce(new BaiJiaReduce());
        try {
            List<BaiJiaEntity> reusltlist = reduceresutl.collect();
            for (BaiJiaEntity baiJiaInfo : reusltlist) {
                String userid = baiJiaInfo.getUserid();
                List<BaiJiaEntity> list = baiJiaInfo.getList();
                Collections.sort(list, new Comparator<BaiJiaEntity>() {
                    @Override
                    public int compare(BaiJiaEntity o1, BaiJiaEntity o2) {
                        String timeo1 = o1.getCreatetime();
                        String timeo2 = o2.getCreatetime();
                        DateFormat dateFormat = new SimpleDateFormat("yyyyMMdd hhmmss");
                        Date datenow = new Date();
                        Date time1 = datenow;
                        Date time2 = datenow;
                        try {
                            time1 = dateFormat.parse(timeo1);
                            time2 = dateFormat.parse(timeo2);
                        } catch (ParseException e) {
                            e.printStackTrace();
                        }
                        return time1.compareTo(time2);
                    }
                });
                BaiJiaEntity before = null;
                Map<Integer, Integer> frequencymap = new HashMap<Integer, Integer>();
                double maxamount = 0d;
                double sum = 0d;
                for (BaiJiaEntity baiJiaInfoinner : list) {
                    if (before == null) {
                        before = baiJiaInfoinner;
                        continue;
                    }
                    //计算购买的频率
                    String beforetime = before.getCreatetime();
                    String endstime = baiJiaInfoinner.getCreatetime();
                    int days = DateUtils.getDaysBetweenbyStartAndend(beforetime, endstime, "yyyyMMdd hhmmss");
                    int brefore = frequencymap.get(days) == null ? 0 : frequencymap.get(days);
                    frequencymap.put(days, brefore + 1);

                    //计算最大金额
                    String totalamountstring = baiJiaInfoinner.getTotalamount();
                    Double totalamout = Double.valueOf(totalamountstring);
                    if (totalamout > maxamount) {
                        maxamount = totalamout;
                    }

                    //计算平均值
                    sum += totalamout;

                    before = baiJiaInfoinner;
                }
                double avramount = sum / list.size();
                int totaldays = 0;
                Set<Map.Entry<Integer, Integer>> set = frequencymap.entrySet();
                for (Map.Entry<Integer, Integer> entry : set) {
                    Integer frequencydays = entry.getKey();
                    Integer count = entry.getValue();
                    totaldays += frequencydays * count;
                }
                int avrdays = totaldays / list.size();//平均天数

                //败家指数 = 支付金额平均值*0.3、最大支付金额*0.3、下单频率*0.4
                //支付金额平均值30分（0-20 5 20-60 10 60-100 20 100-150 30 150-200 40 200-250 60 250-350 70 350-450 80 450-600 90 600以上 100  ）
                // 最大支付金额30分（0-20 5 20-60 10 60-200 30 200-500 60 500-700 80 700 100）
                // 下单平率30分 （0-5 100 5-10 90 10-30 70 30-60 60 60-80 40 80-100 20 100以上的 10）
                int avraoumtsoce = 0;
                if (avramount >= 0 && avramount < 20) {
                    avraoumtsoce = 5;
                } else if (avramount >= 20 && avramount < 60) {
                    avraoumtsoce = 10;
                } else if (avramount >= 60 && avramount < 100) {
                    avraoumtsoce = 20;
                } else if (avramount >= 100 && avramount < 150) {
                    avraoumtsoce = 30;
                } else if (avramount >= 150 && avramount < 200) {
                    avraoumtsoce = 40;
                } else if (avramount >= 200 && avramount < 250) {
                    avraoumtsoce = 60;
                } else if (avramount >= 250 && avramount < 350) {
                    avraoumtsoce = 70;
                } else if (avramount >= 350 && avramount < 450) {
                    avraoumtsoce = 80;
                } else if (avramount >= 450 && avramount < 600) {
                    avraoumtsoce = 90;
                } else if (avramount >= 600) {
                    avraoumtsoce = 100;
                }

                int maxaoumtscore = 0;
                if (maxamount >= 0 && maxamount < 20) {
                    maxaoumtscore = 5;
                } else if (maxamount >= 20 && maxamount < 60) {
                    maxaoumtscore = 10;
                } else if (maxamount >= 60 && maxamount < 200) {
                    maxaoumtscore = 30;
                } else if (maxamount >= 200 && maxamount < 500) {
                    maxaoumtscore = 60;
                } else if (maxamount >= 500 && maxamount < 700) {
                    maxaoumtscore = 80;
                } else if (maxamount >= 700) {
                    maxaoumtscore = 100;
                }

                // 下单平率30分 （0-5 100 5-10 90 10-30 70 30-60 60 60-80 40 80-100 20 100以上的 10）
                int avrdaysscore = 0;
                if (avrdays >= 0 && avrdays < 5) {
                    avrdaysscore = 100;
                } else if (avramount >= 5 && avramount < 10) {
                    avrdaysscore = 90;
                } else if (avramount >= 10 && avramount < 30) {
                    avrdaysscore = 70;
                } else if (avramount >= 30 && avramount < 60) {
                    avrdaysscore = 60;
                } else if (avramount >= 60 && avramount < 80) {
                    avrdaysscore = 40;
                } else if (avramount >= 80 && avramount < 100) {
                    avrdaysscore = 20;
                } else if (avramount >= 100) {
                    avrdaysscore = 10;
                }
                double totalscore = (avraoumtsoce / 100) * 30 + (maxaoumtscore / 100) * 30 + (avrdaysscore / 100) * 40;

                String tablename = "userflaginfo";
                String rowkey = userid;
                String famliyname = "baseinfo";
                String colum = "baijiasoce";
                HbaseUtils.putData(tablename, rowkey, famliyname, colum, totalscore + "");
            }
            env.execute("baijiascore analy");
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

}
```
#### 25用户画像之败家指数代码编写5
```java

```
#### 26用户画像之败家指数之最终得分计算代码编写
```java

```
#### 27用户画像之败家指数之最终得分保存代码编写
```java

```
#### 28用户画像之用户行为日志结构讲解以及实体定义

浏览商品行为：商品id 商品类别id 浏览时间、停留时间、用户id 终端类别,用户ip
收藏商品行为：商品id 商品类别id 操作时间、操作类型（收藏，取消）、用户id、终端类别、用户ip
购物车行为：商品id 商品类别id 、操作时间、操作类型（加入，删除）、用户id、终端类别、用户ip
关注商品:商品id 商品类别id 操作时间、操作类型（关注，取消）、用户id、终端类别、用户ip

```java
public class AttentionProductLog implements Serializable {
    private int productid;//商品id
    private int producttypeid;//商品类别id
    private String opertortime;//操作时间
    private int operatortype;//操作类型，0、关注，1、取消
    private String staytime;//停留时间
    private int userid;//用户id
    private int usetype;//终端类型：0、pc端；1、移动端；2、小程序端'
    private String ip;// 用户ip
    private String brand;//品牌
}

public class BuyCartProductLog implements Serializable {
    private int productid;//商品id
    private int producttypeid;//商品类别id
    private String operatortime;//操作时间
    private int operatortype;//操作类型 0、加入，1、删除
    private int userid;//用户id
    private int usetype;//终端类型：0、pc端；1、移动端；2、小程序端'
    private String ip;// 用户ip

    private String brand;//品牌
}

public class CollectProductLog {
    private int productid;//商品id
    private int producttypeid;//商品类别id
    private String opertortime;//操作时间
    private int opertortype;//操作类型，0、收藏，1、取消
    private int userid;//用户id
    private int usetype;//终端类型：0、pc端；1、移动端；2、小程序端'
    private String ip;// 用户ip

    private String brand;//品牌
}

public class ScanProductLog implements Serializable {
    private int productid;//商品id
    private int producttypeid;//商品类别id
    private String scantime;//浏览时间
    private String staytime;//停留时间
    private int userid;//用户id
    private int usetype;//终端类型：0、pc端；1、移动端；2、小程序端'
    private String ip;// 用户ip

    private String brand;//品牌
}    
```
#### 29基于springboot+springcloud之2.0版本构建实时数据收集服务之注册中心代码编写1
```java

```
#### 30基于springboot+springcloud之2.0版本构建实时数据收集服务之注册中心补充
```java

```
#### 31基于springboot+springcloud之2.0版本构建实时数据收集服务之服务搭建代码编写
```java

```
#### 32用户画像之基于springboot+springcloud之2.0版本构建实时数据收集服务代码编写
```java

```
#### 33用户画像之kafka环境搭建
```java

```
#### 34用户画像之实时收集服务整合kafka代码编写1
```java

```
#### 35用户画像之实时收集服务整合kafka代码编写2
```java

```
#### 36用户画像之实时品牌偏好设计以及代码编写实现实时更新用户品牌偏好

==2.kafkaEvent支持==

```java

public class BrandLikeEntity {
    private String brand;
    private long count;
    private String groupbyfield;
}

public class BrandLikeMap implements FlatMapFunction<KafkaEvent, BrandLike>  {

    @Override
    public void flatMap(KafkaEvent kafkaEvent, Collector<BrandLike> collector) throws Exception {
            String data = kafkaEvent.getWord();
            ScanProductLog scanProductLog = JSONObject.parseObject(data,ScanProductLog.class);
            int userid = scanProductLog.getUserid();
            String brand = scanProductLog.getBrand();
            String tablename = "userflaginfo";
            String rowkey = userid+"";
            String famliyname = "userbehavior";
            String colum = "brandlist";//运营
            String mapdata = HbaseUtils.getdata(tablename,rowkey,famliyname,colum);
            Map<String,Long> map = new HashMap<String,Long>();
            if(StringUtils.isNotBlank(mapdata)){
                map = JSONObject.parseObject(mapdata,Map.class);
            }
            //获取之前的品牌偏好
            String maxprebrand = MapUtils.getmaxbyMap(map);

            long prebarnd = map.get(brand)==null?0l:map.get(brand);
            map.put(brand,prebarnd+1);
            String finalstring = JSONObject.toJSONString(map);
            HbaseUtils.putdata(tablename,rowkey,famliyname,colum,finalstring);

            String maxbrand = MapUtils.getmaxbyMap(map);
            if(StringUtils.isNotBlank(maxbrand)&&!maxprebrand.equals(maxbrand)){
                BrandLike brandLike = new BrandLike();
                brandLike.setBrand(maxprebrand);
                brandLike.setCount(-1l);
                brandLike.setGroupbyfield("==brandlik=="+maxprebrand);
                collector.collect(brandLike);
            }

            BrandLike brandLike = new BrandLike();
            brandLike.setBrand(maxbrand);
            brandLike.setCount(1l);
            collector.collect(brandLike);
            brandLike.setGroupbyfield("==brandlik=="+maxbrand);
            colum = "brandlike";
            HbaseUtils.putdata(tablename,rowkey,famliyname,colum,maxbrand);

    }

}

public class BrandlikeTask {
    public static void main(String[] args) {
        // parse input arguments
        args = new String[]{"--input-topic","scanProductLog","--bootstrap.servers","192.168.80.134:9092","--zookeeper.connect","192.168.80.134:2181","--group.id","youfan"};
        final ParameterTool parameterTool = ParameterTool.fromArgs(args);


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
                                .assignTimestampsAndWatermarks(new CustomWatermarkExtractor()));
        DataStream<BrandLike> brandLikeMap = input.flatMap(new BrandLikeMap());

        DataStream<BrandLike> brandLikeReduce = brandLikeMap.keyBy("groupbyfield").timeWindowAll(Time.seconds(2)).reduce(new BrandLikeReduce());

        brandLikeReduce.addSink(new BrandLikeSink());

        try {
            env.execute("brandLike analy");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

public class MapUtils {

    public static String getmaxbyMap(Map<String,Long> datamap){
        if(datamap.isEmpty()){
                return  null;
        }
        TreeMap<Long,String> map = new TreeMap<Long, String>(new Comparator<Long>() {
            public int compare(Long o1, Long o2) {
                return o2.compareTo(o1);
            }
        });
        Set<Map.Entry<String,Long>> set = datamap.entrySet();
        for(Map.Entry<String,Long> entry :set){
            String key = entry.getKey();
            Long value = entry.getValue();
            map.put(value,key);
        }
        return map.get(map.firstKey());
    }
}
```
#### 37用户画像之实时品牌偏好代码编写2
```java
public class BrandLikeReduce implements ReduceFunction<BrandLike> {
    @Override
    public BrandLike reduce(BrandLike brandLike, BrandLike t1) throws Exception {
        String brand = brandLike.getBrand();
        long count1 = brandLike.getCount();
        long count2 = t1.getCount();
        BrandLike brandLikefinal = new BrandLike();
        brandLikefinal.setBrand(brand);
        brandLikefinal.setCount(count1+count2);
        return brandLikefinal;
    }
}
```
#### 38用户画像之实时品牌偏好代码编写3
```java
public class BrandLikeSink implements SinkFunction<BrandLike> {
    @Override
    public void invoke(BrandLike value, Context context) throws Exception {
        String brand = value.getBrand();
        long count = value.getCount();
        Document doc = MongoUtils.findoneby("brandlikestatics","youfanPortrait",brand);
        if(doc == null){
            doc = new Document();
            doc.put("info",brand);
            doc.put("count",count);
        }else{
            Long countpre = doc.getLong("count");
            Long total = countpre+count;
            doc.put("count",total);
        }
        MongoUtils.saveorupdatemongo("brandlikestatics","youfanPortrait",doc);
    }
}
```
#### 39-41用户画像之实时终端偏好代码编写123
```java

```
#### 42用户画像之flume环境搭建
```java

```
#### 43用户画像之梯度下降法大白话讲解
```java

```
#### 44用户画像之结合数据微分以及数学公式讲解梯度下降法
```java

```
#### 45用户画像之java实现逻辑回归算法
```java

```
#### 46用户画像之flink实现分布式逻辑回归算法代码编写1
```java

```
#### 47用户画像之flink实现分布式逻辑回归算法代码编写2
```java

```
#### 48用户画像之flink逻辑回归预测性别代码编写1
```java

```
#### 49用户画像之flink逻辑回归预测性别代码编写2
```java

```
#### 50用户画像之flink逻辑回归预测性别代码编写3
```java

```
#### 51用户画像之kmeans之原理讲解
```java

```
#### 52用户画像之java实现kmeans代码编写
```java

```
#### 53用户画像之flink实现分布式kmeans代码编写
```java

```
#### 54用户画像之flink实现分布式kmeans代码编写2
```java

```
#### 55用户画像之flink实现分布式kmeans代码编写3
```java

```
#### 56用户画像之flink实现分布式kmeans代码编写4
```java

```
#### 57用户画像之flink分布式kmeans实现用户分群代码编写1
```java

```
#### 58用户画像之flink分布式kmeans实现用户分群代码编写2
```java

```
#### 59用户画像之flink分布式kmeans实现用户分群代码编写3
```java

```
#### 60用户画像之flink分布式kmeans实现用户分群代码编写4
```java

```
#### 61用户画像之flink分布式kmeans实现用户分群代码编写5
```java

```
#### 62用户画像之潮男族潮女族标签代码编写1
```java

```
#### 63用户画像之潮男组潮女族标签代码编写2
```java

```
#### 64用户画像之潮男族潮女族标签代码编写3
```java

```
#### 65用户画像之潮男族潮女族标签代码编写4
```java

```
#### 66用户画像之消费水平标签代码编写1
```java

```
#### 67用户画像之消费水平标签代码编写2
```java

```
#### 68用户画像之消费水平标签代码编写3
```java

```
#### 69用户画像之vuejs+nodejs构建前端项目讲解
```java

```
#### 70用户画像之vuejs+highcharts构建图表代码编写
```java

```
#### 71用户画像之vuejs+hightcharts构建图表效果演示
```java

```
#### 72用户画像之接口查询服务构建
```java

```
#### 73用户画像之年代接口代码编写
```java

```
#### 74用户画像之前端查询服务构建
```java

```
#### 75用户画像之基于springcloud+Feign服务调用代码编写
```java

```
#### 76用户画像之基于springcloud+Feign服务调用代码编写2
```java

```
#### 77用户画像之vuejs整合前端查询接口代码编写
```java

```
#### 78用户画像之vuejs整合前段查询接口之跨域问题解决
```java

```
#### 79用户画像之前端查询接口进一步封装代码编写
```java

```
#### 80用户画像之接口重构代码编写
```java

```
#### 81用户画像之前端查询接口重用改造代码编写
```java

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

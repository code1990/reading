#### 63、SpringBoot_数据访问-整合MyBatis（一）-基础环境搭建
```xml
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.3.1</version>
		</dependency>
```
步骤：

 1）、配置数据源相关属性（见上一节Druid）

 2）、给数据库建表

 3）、创建JavaBean

#### 64、SpringBoot_数据访问-整合MyBatis（二）-注解版MyBatis

```java
//指定这是一个操作数据库的mapper
@Mapper
public interface DepartmentMapper {

    @Select("select * from department where id=#{id}")
    public Department getDeptById(Integer id);

    @Delete("delete from department where id=#{id}")
    public int deleteDeptById(Integer id);

    @Options(useGeneratedKeys = true,keyProperty = "id")
    @Insert("insert into department(departmentName) values(#{departmentName})")
    public int insertDept(Department department);

    @Update("update department set departmentName=#{departmentName} where id=#{id}")
    public int updateDept(Department department);
}
//设置自定义的扫描规则
@org.springframework.context.annotation.Configuration
public class MyBatisConfig {

    @Bean
    public ConfigurationCustomizer configurationCustomizer(){
        return new ConfigurationCustomizer(){

            @Override
            public void customize(Configuration configuration) {
            	//是否开启下划线与驼峰命名匹配
                configuration.setMapUnderscoreToCamelCase(true);
            }
        };
    }
}
```
#### 65、SpringBoot_数据访问-整合MyBatis（二）-配置版MyBatis
```properties
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml 指定全局配置文件的位置
  mapper-locations: classpath:mybatis/mapper/*.xml  指定sql映射文件的位置
```

```java
//使用MapperScan批量扫描所有的Mapper接口；
@MapperScan(value = "com.springboot.mapper")
@SpringBootApplication
public class SpringBoot06DataMybatisApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringBoot06DataMybatisApplication.class, args);
	}
}
```


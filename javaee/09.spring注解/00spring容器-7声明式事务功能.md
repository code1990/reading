### 基于注解的事务

```java
@Repository
public class UserDao {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public void insert() {
        String sql = "INSERT INTO `tb_user`(username,age) VALUES(?,?)";
        String username = UUID.randomUUID().toString().substring(0, 5);
        jdbcTemplate.update(sql, username, 19);

    }

}

@Service
public class UserService {

    @Autowired
    private UserDao userDao;

    /*一般放在public的service层方法上 同时数据存储引擎为Innodb*/
    @Transactional
    public void insertUser(){
        userDao.insert();
        System.out.println("insert user");
        //主动抛出异常 实现业务逻辑回滚
        int i =10/0;
    }
}

/**
 * 声明式事务：
 * <p>
 * 环境搭建：
 * 1、导入相关依赖
 * 数据源、数据库驱动、Spring-jdbc模块
 * 2、配置数据源、JdbcTemplate（Spring提供的简化数据库操作的工具）操作数据
 * 3、给方法上标注 @Transactional 表示当前方法是一个事务方法；
 * 4、 @EnableTransactionManagement 开启基于注解的事务管理功能；
 *
 * 		@EnableXXX 
 	5、配置事务管理器来控制事务;
 * @Bean public PlatformTransactionManager transactionManager()
 */
/*@EnableTransactionManagement 开启基于注解的事务管理功能；*/
@EnableTransactionManagement
@ComponentScan("part06transcational")
@Configuration
public class TransactionalConfig {

    //数据源
    @Bean
    public DataSource dataSource() throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("123456");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        return dataSource;
    }

    //JdbcTemplate
    @Bean
    public JdbcTemplate jdbcTemplate() throws Exception{
        //Spring对@Configuration类会特殊处理；给容器中加组件的方法，多次调用都只是从容器中找组件
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource());
        return jdbcTemplate;
    }

    //注册事务管理器在容器中
    @Bean
    public PlatformTransactionManager transactionManager() throws Exception{
        return new DataSourceTransactionManager(dataSource());
    }


}
```


# IOC 注解配置

# IOC 注解配置

1. 创建对象Bean

   @Component

   属性：value 指定bean的id 如果不指定value,bean默认id为当前类类名首字母小写

   衍生注解 作用一样

   - @Controller 表现层
   - @Service 业务层
   - @Repository 持久层
2. 注入数据

   @Autowired 自动按照类型注入

   ---

   @Qualifier 按照类型注入的基础上按照bean id注入

   常与@Autowired一起使用，给方法注入参数可以独立使用

   属性：value 指定bean的id

   ---

   @Value

   作用：注入基本数据类型和String类型数据

   属性：value指定值

   ---

   @Scope

   指定bean作用范围

   属性

   value:指定范围

   可取值 singleton prototype request session gloabalsession

   ---

   @PostConstruct

   指定bean初始化方法

   @PreDestroy

   指定bean销毁方法
3. 半注解配置

配置数据库连接，开启扫描

```xml
<!-- 告知spring框架在，读取配置⽂件，创建容器时，扫描注解，依据注解创建对象，并存⼊容器中 -->
<context:component-scan base-package="com.zzxx"/>

<!-- 配置 queryRunner 此处我们只注⼊了数据源，表明每条语句独⽴事务-->
<bean id="queryRunner" class="org.apache.commons.dbutils.QueryRunner">
	<constructor-arg name="ds" ref="dataSource"/>
</bean>
<!-- 配置数据源 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
	<property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
	<property name="jdbcUrl" value="jdbc:mysql:///spring"/>
	<property name="user" value="root"/>
	<property name="password" value="root"/>
</bean>
```

1. 全注解配置

   @Configuration 配置类

   ```java
   /**
   * spring的配置类，相当于beans.xml⽂件
   * @author bonnie
   **/
   @Configuration
   	public class SpringConfiguration {
   }
   ```

   @ComponentScan 扫描

   ```java
   /**
   * spring的配置类，相当于beans.xml⽂件
   * @author bonnie
   **/
   @Configuration
   @ComponentScan("com.zzxx")
   	public class SpringConfiguration {
   }
   ```

   @Bean

   写在方法上，表明使用此方法创建一个对象

   属性name:指定名称

   ```java
   /**
   * 连接数据库的配置类
   * @author bonnie
   **/
   public class JdbcConfig {
   /**
   * 创建⼀个数据源，并存⼊ spring 容器中
   * @return
   */
   	@Bean(name = "dataSource")
   	public DataSource createDataSource() {
   		try {
   			ComboPooledDataSource ds = new ComboPooledDataSource();
   			ds.setUser("root");
   			ds.setPassword("root");
   			ds.setDriverClass("com.mysql.cj.jdbc.Driver");
   			ds.setJdbcUrl("jdbc:mysql:///spring");
   			return ds;
   		} catch (Exception e) {
   			throw new RuntimeException(e);
   		}
   	}
   	/**
   	* 创建⼀个 QueryRunner，并且也存⼊ spring 容器中
   	* @param dataSource
   	* @return
   	*/
   	@Bean(name = "queryRunner")
   	public QueryRunner createQueryRunner(DataSource dataSource) {
   			return new QueryRunner(dataSource);
   	}
   }
   ```

   @PropertySource

   加载.properties文件中的配置

   属性value[]:指定文件位置

   ```java
   /**
   * 连接数据库的配置类
   * @author bonnie
   **/
   @PropertySource("classpath:jdbc.properties")
   public class JdbcConfig {
   	@Value("${jdbc.driver}")
   	private String driver;
   	@Value("${jdbc.url}")
   	private String url;
   	@Value("${jdbc.username}")
   	private String username;
   	@Value("${jdbc.password}")
   	private String password;
   	/**
   	* 创建⼀个数据源，并存⼊ spring 容器中
   	* @return
   	*/
   	@Bean(name = "dataSource")
   	public DataSource createDataSource() {
   		try {
   			ComboPooledDataSource ds = new ComboPooledDataSource();
   			ds.setUser(username);
   			ds.setPassword(password);
   			ds.setDriverClass(driver);
   			ds.setJdbcUrl(url);
   			return ds;
   		} catch (Exception e) {
   			throw new RuntimeException(e);
   		}
     }
   }
   ```

   ```java
   jdbc.driver=com.mysql.cj.jdbc.Driver
   jdbc.url=jdbc:mysql:///spring
   jdbc.username=root
   jdbc.password=root
   ```

   @Import

   导入其他配置类

   属性value[]：指定其他配置类字节码

   ```java
   @Configuration
   @ComponentScan("com.zzxx")
   @Import({JdbcConfig.class})
   	public class SpringConfiguration {
   }
   ```

   ```java
   @Configuration
   @PropertySource("classpath:jdbc.properties")
   	public class JdbcConfig {
   }
   ```

   注解类获得容器

   ```java
   ApplicationContext ac = new AnnotationConfigApplicationContext(SpringConfiguration.class);
   ```

Spring整合Junit

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:beans.xml"})
	public class AccountServiceTest {
}
```

locations属性：指定配置文件，类路径下加classpath

classes：指定注解的类

‍

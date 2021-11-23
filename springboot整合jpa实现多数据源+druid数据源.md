# springboot整合jpa实现多数据源+druid数据源

## 一、引入坐标

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.22</version>
</dependency>
    <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.20</version>
    <scope>provided</scope>
</dependency>
<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
<dependency>
     <groupId>com.alibaba</groupId>
     <artifactId>druid</artifactId>
     <version>1.2.6</version>
</dependency>
```

## 二、配置文件

```
spring:
  datasource:
    primary:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://127.0.0.1:3306/explorer-config?useUnicode=true&characterEncoding=UTF8&zeroDateTimeBehavior=CONVERT_TO_NULL&serverTimezone=GMT%2B8
      username: root
      password: root.123321
      type: com.alibaba.druid.pool.DruidDataSource
    secondary:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://127.0.0.1:3306/explorer-unspent?useUnicode=true&characterEncoding=UTF8&zeroDateTimeBehavior=CONVERT_TO_NULL&serverTimezone=GMT%2B8
      username: root
      password: root.123321
      type: com.alibaba.druid.pool.DruidDataSource
    druid:
      initial-size: 10  # 初始化连接池的连接数
      min-idle: 10      # 空闲链接保持数量
      max-active: 20   # 连接池最大活跃连接数
      max-wait: 100000  # 配置获取连接等待超时的时间
      time-between-eviction-runs-millis: 60000 # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
      min-evictable-idle-time-millis: 300000   # 配置一个连接在池中最小生存的时间，单位是毫秒
      validation-query: SELECT 1   # Druid用来测试连接是否可用的SQL语句,默认值每种数据库都不相同
      test-while-idle: true   #指明连接是否被空闲连接回收器(如果有)进行检验.如果检测失败,则连接将被从池中去除.注意: 设置为true后如果要生效,validationQuery参数必须设置为非空字符串
      test-on-borrow: false   # 指明是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个.注意: 设置为true后如果要生效,validationQuery参数必须设置为非空字符串
      test-on-return: false   # 指明是否在归还到池中前进行检验 注意: 设置为true后如果要生效,validationQuery参数必须设置为非空字符串


  jpa:
    hibernate:
      ddl-auto: none
```

## 三、多数据源配置

### 1、主数据源配置类JPAPrimaryConfig

```java
import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.boot.autoconfigure.orm.jpa.HibernateProperties;
import org.springframework.boot.autoconfigure.orm.jpa.HibernateSettings;
import org.springframework.boot.autoconfigure.orm.jpa.JpaProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.annotation.Resource;
import javax.persistence.EntityManager;
import javax.sql.DataSource;
import java.util.Map;

@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        entityManagerFactoryRef="entityManagerFactoryPrimary",
        transactionManagerRef="transactionManagerPrimary",
        basePackages= {"com.blockchain.primaryserver.dao"}) //Repository所在位置
public class JPAPrimaryConfig {

    @Resource
    private JpaProperties jpaProperties;
    @Resource
    private HibernateProperties hibernateProperties;


    @Primary
    @Bean(name = "primaryDataSource")
    @ConfigurationProperties(prefix="spring.datasource.primary")  //使用application.yml的primary数据源配置
    public DataSource primaryDataSource() {
        // 使用druid连接池
        return new DruidDataSource();
    }

    @Primary
    @Bean(name = "entityManagerPrimary")        //primary实体管理器
    public EntityManager entityManager(EntityManagerFactoryBuilder builder) {
        return entityManagerFactoryPrimary(builder).getObject().createEntityManager();
    }

    @Primary
    @Bean(name = "entityManagerFactoryPrimary")    //primary实体工厂
    public LocalContainerEntityManagerFactoryBean entityManagerFactoryPrimary (EntityManagerFactoryBuilder builder) {
        Map<String,Object> properties =
                hibernateProperties.determineHibernateProperties(
                        jpaProperties.getProperties(),
                        new HibernateSettings());
        return builder.dataSource(primaryDataSource())
                .properties(properties)
                .packages("com.blockchain.primaryserver.entity")     //换成你自己的实体类所在位置
                .persistenceUnit("primaryPersistenceUnit")
                .build();
    }

    @Primary
    @Bean(name = "transactionManagerPrimary")         //primary事务管理器
    public PlatformTransactionManager transactionManagerPrimary(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(entityManagerFactoryPrimary(builder).getObject());
    }
}
```

### 2、副数据源配置类JPASecondaryConfig

```java
import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.boot.autoconfigure.orm.jpa.HibernateProperties;
import org.springframework.boot.autoconfigure.orm.jpa.HibernateSettings;
import org.springframework.boot.autoconfigure.orm.jpa.JpaProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.annotation.Resource;
import javax.persistence.EntityManager;
import javax.sql.DataSource;
import java.util.Map;


@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        entityManagerFactoryRef="entityManagerFactorySecondary",
        transactionManagerRef="transactionManagerSecondary",
        basePackages= { "com.blockchain.secondaryserver.dao" }) //Repository所在位置
public class JPASecondaryConfig {

    @Resource
    private JpaProperties jpaProperties;
    @Resource
    private HibernateProperties hibernateProperties;


    @Bean(name = "secondaryDataSource")
    @ConfigurationProperties(prefix="spring.datasource.secondary")   //使用application.yml的secondary数据源配置
    public DataSource secondaryDataSource() {
        // 使用druid连接池
        return new DruidDataSource();
    }

    @Bean(name = "entityManagerSecondary")      //secondary实体管理器
    public EntityManager entityManager(EntityManagerFactoryBuilder builder) {
        return entityManagerFactorySecondary(builder).getObject().createEntityManager();

    }



    @Bean(name = "entityManagerFactorySecondary")
    public LocalContainerEntityManagerFactoryBean entityManagerFactorySecondary (EntityManagerFactoryBuilder builder) {

        Map<String,Object> properties =
                hibernateProperties.determineHibernateProperties(
                        jpaProperties.getProperties(),
                        new HibernateSettings());

        return builder
                .dataSource(secondaryDataSource())
                .properties(properties)
                .packages("com.blockchain.secondaryserver.entity") //换成你自己的实体类所在位置
                .persistenceUnit("secondaryPersistenceUnit")
                .build();
    }


    @Bean(name = "transactionManagerSecondary")
    PlatformTransactionManager transactionManagerSecondary(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(entityManagerFactorySecondary(builder).getObject());
    }
}
```

## 四、启动类

```java
@SpringBootApplication
@ComponentScan(basePackages = "com.blockchain")
public class ExplorerParseUnspentApplication {

    public static void main(String[] args) {
        SpringApplication.run(ExplorerParseUnspentApplication.class, args);
    }
}
```

## 五、事务

```java
不同的数据源，事务需要指定事务
@Transactional(transactionManager = "transactionManagerSecondary")
注意：不同数据源需要指定不同事务，同一方法下出现多个数据源，需要拆分开。 
```


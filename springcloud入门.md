# 	**SpringCloud-入门**

## 第一章、微服务架构概要

### 1.1、什么是微服务

马丁·福勒 ，他于2014年发表了一篇关于微服务的博客https://martinfowler.com/microservices/

 英文：https://martinfowler.com/articles/microservices.html#MicroservicesAndSoa

 微服务是一种架构风格，是以开发一组小型服务的方式来作为一个**独立**的应用系统，每个服务都运行在
自已的进程中，服务之间采用轻量级的**HTTP**通信机制 ( 通常是采用HTTP的RESTful API )进行通信。这
些服务都是围绕具体业务进行构建的，并且可以独立部署到生产环境上。这些服务可以用**不同的编程语**
**言**编写，并且可以使用**不同的数据存储**技术。对这些微服务我们只需要使用一个非常轻量级的集中式管
理来进行协调。

### 1.2、单体应用架构

一个应用包含了应用程序的所有功能。

```java
优点： 
	易于开发、测试、部署 
	易于整体扩展，当负载压力大时，可以部署多个后台用nginx进行负载均衡 
缺点： 
	复杂度高，单个应用包含多个模块，依赖关系不清晰、代码的质量参差不齐 
	技术债务，随着时间推移，需求、人员的变更，会逐渐形成应用程序的技术债务 
	阻碍技术创新，需要使用相同的开发语音和持久化存储
```



### 1.3、微服务架构

微服务是一种架构风格，是以开发一组小型服务的方式来作为一个独立的应用系统，每个服务都运行在自已
的进程中，服务之间采用轻量级的HTTP通信机制 ( 通常是采用HTTP的RESTful API )进行通信

```java
优点:  
	易于开发和维护,一个微服务只会关注一个特定的业务功能，所以业务清晰、代码量较少  
    单个微服务启动较快  
    局部修改容易部署  
    技术栈不受限制  
    按需伸缩 
缺点: 
    运维要求高  
    分布式固有的复杂性  
    接口调整成本高
```

### 1.4、微服务架构技术栈

| 技术实现                               | 微服务技术维度                                          |
| -------------------------------------- | ------------------------------------------------------- |
| 服务开发                               | Spring Boot、Spring、Spring MVC等                       |
| 服务注册与发现                         | Eureka、Zookeeper等                                     |
| 服务调用                               | Rest、RPC等                                             |
| 服务熔断器                             | Hystrix、Envoy等                                        |
| 负载均衡                               | Ribbon、Nginx等                                         |
| 服务接口调用(客户端调用服务的简化工具) | Feign等                                                 |
| 消息队列                               | Kafka、ActiveMQ等                                       |
| 服务配置中心管理                       | Spring Cloud Config等                                   |
| 服务路由（API网关）                    | Zuul等                                                  |
| 服务监控                               | Zabbix、Nagios等                                        |
| 全链路追踪                             | Zipkin，Brave等                                         |
| 服务部署                               | Docker、OpenStack等                                     |
| 数据流处理                             | Spring Cloud Stream（Redis,Rabbit,Kafka等发送接收消息） |
| 事件消息总线                           | Spring Cloud Bus                                        |

## 第二章、Spring Cloud概要

- 官网: http://spring.io/projects/spring-cloud
- 各组件说明（中文版）：https://springcloud.cc/spring-cloud-netflix.html
- 详细文档版：英文版：https://cloud.spring.io/spring-cloud-static/Finchley.SR2/single/spring-cloud.html
- 中文版：https://springcloud.cc/spring-cloud-dalston.htmlSpring
- Cloud 中国社区：http://springcloud.cn/Spring
- Cloud 中文网：[https://springcloud.c](https://springcloud.c/)

### 2.1、什么是SpringCloud

 SpringCloud是基于Springboot进行开发的(Springboot可以离开SpringCloud进行开发，但是SpringCloud离不开Springboot)，提供了一套微服务解决方案，包括服务注册与发现，配置中心，全链路监控，服务网关，负载均衡，熔断器等组件，除了基于NetFlix的开源组件做高度抽象封装之外，还有一些选型中立的开源组件。

 SpringCloud使用RESTful API实现服务之间通行

 Dubbo使用RPC(远程过程调用)实现服务之间通信

## 第三章、Rest构建分布式微服务架构

### 3.1、自定义Rest配置类

```java
@Configuration
public class ConfigBean {

    /**
     * 向容器中添加 RestTemplate 组件,直接通过此组件可调用 REST 接口
     */
    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

### 3.2、使用RestTemplate

RestTemplate 组件, 提供了多种简单便捷的访问 Restful 服务的方法，是Spring提供的用于访
问Rest服务的客户端模板工具集。 (url, requestMap, ResponseBean.class)这三个参数分别代表：
REST请求地址、请求参数、HTTP响应转换被转换成的对象类型。

```java
@RestController
public class ProductController_Consumer {

    private static final String REST_URL_PREFIX = "http://localhost:8001";

    @Autowired
    private RestTemplate restTemplate;

    @RequestMapping(value = "/consumer/product/add")
    public boolean add(Product product) {
        return restTemplate.postForObject(REST_URL_PREFIX + "/product/add", product, Boolean.class);
    }

    @RequestMapping(value = "/consumer/product/get/{id}")
    public Product get(@PathVariable("id") Long id) {
        return restTemplate.getForObject(REST_URL_PREFIX + "/product/get/" + id, Product.class);
    }

    @RequestMapping(value = "/consumer/product/list")
    public List<Product> list() {
        return restTemplate.getForObject(REST_URL_PREFIX + "/product/list", List.class);
    }
}
```

## 第四章、Eureka 服务的注册与发现

为什么要用注册中心？

1. 微服务数量众多，要进行远程调用就需要知道服务端的ip地址和端口，注册中心帮助我们管理这些服务
   的ip和端口。
2. 微服务会实时上报自己的状态，注册中心统一管理这些微服务的状态，将存在问题的服务踢出服务列
   表，客户端获取到可用的服务进行调用

### 4.1、Eureka介绍

- Spring Cloud Eureka 是对Netflix公司的Eureka的二次封装，它实现了服务治理的功能，Spring Cloud
  Eureka 提供 Eureka Server 服务端与 Eureka Client 客户端 ，服务端即是Eureka服务注册中心，客户端完成
  微服务向Eureka服务的注册与发现
- 客户端同时也具备一个内置的使用轮询(round-robin)负载算法的负载均衡器。在微服务启动后，将会向
  Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心
  跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）

```java
1. Eureka Server 是服务端，负责管理各个微服务注册和发现。
2. 在微服务上添加 Eureka Client 代码，就会访问到 Eureka Server 将此微服务注册在Eureka Server中，从而
使服务消费方能够找到。
3. 微服务（服务消费者）需要调用另一个微服务（服务提供者）时，从 Eureka Server 中获取服务调用地址，进行远程调用。
```

### 4.2、搭建单机版 Eureka Server 服务注册中心

```java
1、引入pom坐标
			<!--Eureka-server 服务端依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
  
2、配置文件
  server:
  port: 6001 # 服务端口
eureka:
  instance:
    hostname: localhost # eureka服务端的实例名称
  client:
    registerWithEureka: false # 服务注册，false表示不将自已注册到Eureka服务中
    fetchRegistry: false # 服务发现，false表示自己不从Eureka服务中获取注册信息
    serviceUrl: # Eureka客户端与Eureka服务端的交互地址，集群版配置对方的地址，单机版配置自己（如果不配置则默认本机8761端口）
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

3、启动类
   添加注解 @EnableEurekaServer //标识一个Eureka Server服务注册中心
```

### 4.3、服务注册到 Eureka Server 服务注册中心

```java
1、引入pom坐标
	      <!-- 导入Eureka客户端的依赖，将 微服务提供者 注册进 Eureka -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
  
2、配置文件
# 将服务提供者 注册到 Eureka服务注册中心
eureka:
  client:
    registerWithEureka: true # 服务注册开关
    fetchRegistry: true # 服务发现开关
    serviceUrl: # 客户端(服务提供者)注册到哪一个Eureka Server服务注册中心，多个用逗号分隔
      defaultZone: http://localhost:6001/eureka
  instance:
    instanceId: ${spring.application.name}:${server.port} # 指定实例ID,就不会显示主机名了
    prefer-ip-address: true # 鼠标放到实例ID上后显示 ip地址

3、启动类
  添加注解 @EnableEurekaClient //本服务启动后会自动注册进Eureka中心
```

### 4.4、Eureka Server自我保护机制

```java
【自我保护现象】
   如长时间没有访问、检测不到心跳，或者修改实例名称，eureka启动自我保护机制下图红色提示信息：表示已启动了自我保护机制在某时刻某一个微服务不可用了，Eureka不会立刻删除，依旧会对该微服务的信息进行保存
  
【自我保护机制】
   当Eureka Server 在一定时间内（默认90秒）没有接收到某个微服务的心跳，Eureka Server会从服务列表将此服务实例注销。但是如果出现网络异常情况（微服务本身是正常的），微服务与Eureka Server之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是正常的，此时本不应该注销这个微服务。
   Eureka Server有一种 “自我保护模式” 来解决这个问题——当Eureka Server在短时间内丢失过多客户端时（可能发生了网络故障），此时Eureka Server会进入自保护模式，一旦进入该模式，Eureka Server就会保护服务注册表中的信息，不再删除服务注册表中的数据（也就是不会注销任何微服务）。当网络故障恢复后，该Eureka Server会自动退出自我保护模式。
   所以，自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务（健康的微服
务和不健康的微服务都会保留），也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的
健壮、稳定。
```

```java
// 关闭自我保护配置
eureka:
	server:
		enable-self-preservation: false # 禁用自我保护模式
```

![](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/spring-cloud-eureka-%25E8%2587%25AA%25E6%2588%2591%25E4%25BF%259D%25E6%258A%25A41.png)

### 4.5、搭建集群版 Eureka Server 服务注册

**为了避免 Eureka Server的失效，Eureka Server 高可用环境需要部署两个及以上Eureka Server**

```
1、在实际使用时Eureka Server至少部署两台服务器，实现高可用。
2、两台Eureka Server互相注册,注册时需要通过域名访问。
3、微服务需要连接两台Eureka Server注册，当其中一台Eureka死掉也不会影响服务的注册与发现
4、微服务会定时向Eureka Server发送心跳，报告自己的状态。
5、微服务从注册中心获取服务地址以RESTful方式发起远程调用。
```

- 创建新的Eureka Server服务端
- eureka.client.serviceUrl.defaultZone 配置对方服务端的域名地址
- 微服务端(消费提供者) eureka.client.serviceUrl.defaultZone: 服务端1地址，服务端1地址。配置两个服务端地址,逗号隔开
- 如果配置三台以上服务端，服务端配置 另外两台服务端域名地址，微服务端(消费提供者)配置三个服务端地址即可！
- 消费者的集群配置见下章

## 第五章、 Ribbon 客户端负载均衡

### 5.1、什么是负载

```java
1、LB，即负载均衡(Load Balance)，负载均衡是微服务架构中经常使用的一种技术。 负载均衡是我们处理高并
发、缓解网络压力和进行服务端扩容的重要手段之一，简单的说就是将用户的请求平摊的分配到多个服务
上，从而实现系统的高可用性集群。
2、负载均衡可通过 硬件设备 及 软件 进行实现，软件比如：Nginx等，硬件比如：F5等
3、负载均衡相应的在中间件，例如：Dubbo 和 SpringCloud 中均给我们提供了负载均衡组件
```

## 5.2、什么是客户端负载均衡（Ribbon）

 客户端负载均衡 与 服务端负载均衡的区别在于：客户端负载均衡要维护一份服务列表。客户端负载均衡和服务端负载均衡最大的区别在于**服务清单所存储的位置**。在客户端负载均衡中，每个客户端服务都有一份自己要访问的服务端清单，这些清单统统都是从Eureka服务注册中心获取的。而在服务端负载均衡中，只要负载均衡器维护一份服务端列表 。

 Ribbon 从 Eureka Server 获取服务列表，Ribbon根据负载均衡算法直接请求到具体的微服务，中间省去了负载均衡服务。

 **简单来说**，nginx负载均衡属于服务端负载均衡，所有请求必须先通过nginx，然后nginx中配置了所有服务端，根据负载均衡算法进行分配。而Ribbon属于客户端负载均衡，所有请求直接访问，没有中间的nginx(也可以理解为每个客户端有配置了nginx)，每个客户端都知道所有的消费提供者，然后根据负载均衡算法自己进行选择。

### 5.3、 Ribbon 服务调用配置实战

```java
1、服务消费者端(80)引入pom坐标
	<!-- Ribbon 相关依赖,eureka会自动引入Ribbon -->
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
	</dependency>

2、修改application.yml配置文件
eureka:
  client:
    registerWithEureka: false # 服务注册，false表示不将自已注册到Eureka服务中
    fetchRegistry: true # 服务发现，false表示自己不从Eureka服务中获取注册信息
    serviceUrl: # Eureka客户端与Eureka服务端的交互地址，集群版配置对方的地址，单机版配置自己（如果不配置则默认本机8761端口）
      defaultZone: http://eureka6001.com:6001/eureka/,http://eureka6002.com:6002/eureka/

3、修改自定义配置类ConfigBean
  添加@LoadBalanced注解，@LoadBalanced表示这个RestTemplate开启负载均衡，在调用服务提供者的接口时，可使用服务名称替代真实IP地址。服务名称 就是服务提供者在application.yml中配置的spring.application.name属性的值 。
4、 修改消费者控制层(根据第三步)
  private static final String REST_URL_PREFIX = "http://spring-cloud-product";
```

### 5.4、 Ribbon 负载均衡架构

Ribbon 在工作时分成两步

- 先选择 Eureka Server ,它优先选择在同一个区域内负载较少的server
- 再根据用户指定的策略，在从 Eureka Server 获取的服务注册列表中选择一个地址。 其中Ribbon提供了多种策略：比如轮询、随机和根据响应时间加权等。

![](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/SpringCloud-Ribbon.png)

## 第六章、Feign 客户端接口调用

### 6.1、什么是Feign

 Feign是Netflix公司开源的轻量级Rest客户端( https://github.com/OpenFeign/feign )，使用 Feign 可以非常方便、简单的实现 Http 客户端，使用 Feign 只需要定义一个接口，然后在接口上添加注解即可。Spring Cloud 对 Feign 进行了封装，Feign **默认集成了 Ribbon** 实现了客户端负载均衡调用。

```java
1、引入pom坐标
	<!--feign 依赖-->
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-openfeign</artifactId>
	</dependency>
  
2、创建接口
/**
 * 指定调用的服务 spring-cloud-product
 */
@FeignClient(value = "spring-cloud-product")
public interface ProductClientService {

    @RequestMapping(value = "/product/get/{id}",method = RequestMethod.GET)
    Product get(Long id);

    @RequestMapping(value = "/product/list",method = RequestMethod.GET)
    List<Product> list();

    @RequestMapping(value = "/product/add",method = RequestMethod.POST)
    boolean add(Product product);
}

3、controller
@RestController
public class ProductController_Consumer__Feign {

    @Autowired
    private ProductClientService productClientService;

    @RequestMapping(value = "/consumer/product/add")
    public boolean add(Product product) {
        return productClientService.add(product);
    }
    
    @RequestMapping(value = "/consumer/product/get/{id}")
    public Product get(@PathVariable("id") Long id) {
        return productClientService.get(id);
    }
    
    @RequestMapping(value = "/consumer/product/list")
    public List<Product> list() {
        return productClientService.list();
    }
}

4、启动类
  //会扫描标记了指定包下@FeignClient注解的接口，并生成此接口的代理对象
  添加注解 @EnableFeignClients(basePackages = {"com.demo.service"})
```

### 6.2、 Feign 注意事项

SpringCloud对Feign进行了增强兼容了SpringMVC的注解 ，我们在使用SpringMVC的注解时需要注意：

- @FeignClient接口方法有基本类型参数在参数必须加@PathVariable(“XXX”) 或 @RequestParam(“XXX”)
- @FeignClient接口方法返回值为复杂对象时，此类型必须有无参构造方法。

## 第七章、Hystrix 熔断器(断路器)

### 7.1、什么是Hystrix

在服务之间调用的链路上由于网络原因、资源繁忙或者自身的原因，服务并不能保证100%可用，如果单个服务出现问题，调用这个服务就会出现线程阻塞，导致响应时间过长或不可用，此时若有大量的请求涌入，容器的线程资源会被消耗完毕，导致服务瘫痪。服务与服务之间的依赖性，故障会传播，会对整个微服务系统造成灾难性的严重后果，这就是服务故障的“雪崩”效应。
​ 为了解决这个问题，业界提出了熔断器模型。

 Hystrix 是Netflix公司开源项目( https://github.com/Netflix/Hystrix)，实现了熔断器模型，Spring Cloud 对这一组件进行了整合。具有**服务熔断**、**服务监控**的作用。

### 7.2、服务熔断

熔断机制是应对雪崩效应的一种微服务链路保护机制。在微服务架构中，一个请求需要调用多个服务是非常常见的当服务之间调用的链路上某个微服务不可用或者响应时间太长时，会导致连锁故障。当失败的调用到一定阈值（缺省是5秒内20次调用失败) 就会启动熔断机制。在 SpringCloud 框架里熔断机制通过Hystrix实现，Hystrix会监控微服务间调用的状况。熔断机制的注解是**@HystrixCommand**，熔断器打开后，可用避免连锁故障**fallback**方法可以直接返回一个固定值。

 服务熔断分为 **服务端熔断** 和 **客户端熔断**

#### 7.2.1、 服务端熔断

```java
1、导入pom坐标
<!-- 导入hystrix依赖 -->
	<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>

2、修改Controller，如果出现异常，使用@HystrixCommand注解，如果出现异常，会自动调用 @HystrixCommand 注解中 fallbackMethod属性指定的当前类中的方法
  
//当get方法出现异常时，则会调用hystrixGet方法处理
	@HystrixCommand(fallbackMethod = "getFallback")
	@RequestMapping(value = "/product/get/{id}", method = RequestMethod.GET)
	public Product get(@PathVariable("id") Long id) {
	Product product = productService.get(id);
	//模拟异常
		if(product == null) {
			throw new RuntimeException("ID=" + id + "无效");
		}
		return product;
	}

3、启动类添加 注解 @EnableHystrix 开启 Hystrix 熔断机制的支持
```

#### 7.2.2、Feign 客户端服务熔断

Feign 是自带断路器的，也就是针对 消费者（客户端）进行服务熔断，需要在配置文件中开启它

```java
1、配置文件开启 hystrix
# 需要开启 hystrix
feign:
	hystrix:
		enabled: true
      
2、在已存在的接口上的@FeignClient 注解中，加上 fallback 指定熔断处理类即可
@FeignClient(value = "MICROSERVICE-PRODUCT", fallback =ProductClientServiceFallBack.class)
public interface ProductClientService {
	@RequestMapping(value = "/product/get/{id}",method = RequestMethod.GET)
	Product get(@PathVariable("id") Long id);
 }

3、创建熔断处理类,并实现接口,在将熔断处理类注入bean中
@Component 
public class ProductClientServiceFallBack implements ProductClientService {
	@Override
	public Product get(Long id) {
		return new Product(id, "id="+ id + "无数据--feign&hystrix", "无有效数据库");
	}
```

**注意：如果服务端和客户端都设置了熔断处理，则返回值已客户端为主！**

### 7.3、 Hystrix Dashboard监控平台搭建

- 除了隔离依赖服务的调用以外，Hystrix还提供了准实时的调用监控（Hystrix Dashboard），Hystrix会持续地记录所有通过Hystrix发起的请求的执行信息，并以统计报表和图形的形式展示给用户，包括每秒执行多少请求多少成功，多少失败等。
- Netflix 通过 hystrix-metrics-event-stream 项目实现了对以上指标的监控。Spring Cloud也提供了Hystrix Dashboard 的整合，对监控内容转化成可视化界

#### 7.3.1、什么是服务监控

- 除了隔离依赖服务的调用以外，Hystrix还提供了准实时的调用监控（Hystrix Dashboard），Hystrix会持续
  地记录所有通过Hystrix发起的请求的执行信息，并以统计报表和图形的形式展示给用户，包括每秒执行多少
  请求多少成功，多少失败等。
- Netflix 通过 hystrix-metrics-event-stream 项目实现了对以上指标的监控。Spring Cloud也提供了
  Hystrix Dashboard 的整合，对监控内容转化成可视化界面

#### 7.3.2、搭建服务监控

```java
1、添加pom坐标
<!--导入 hystrix 与 hystrix-dashboard 依赖-->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>

2、启动类 添加 @EnableHystrixDashboard 注解开启服务监控
  
3、配置application.yml 
server:
	port: 9001

4、启动微服务监控，访问 http://localhost:9001/hystrix    
```

#### 7.3.3、为要被监控的服务添加配置

```java
1、 在需要监控的服务 pom.xml 中的 dependencies 节点中新增 spring-boot-starter-actuator 监控依赖，
以开启监控相关的端点，并确保已经引入断路器的依赖 spring-cloud-starter-netflix-hystrix

<!--监控依赖-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<!--已经存在 ，则不重复添加-->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>

2、在需要监控的服务 application.yml 配制中添加暴露端点
# 在被监控服务上添加暴露端点
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream

3、启动项目后，访问 http://localhost:8001/actuator/hystrix.stream 就可以看到json格式数据

4、访问服务监控 http://localhost:9001/hystrix 
   然后输入http://localhost:8001/actuator/hystrix.stream
   Delay：2000 Title：product-8001
   点击 Monitor Stream就可以看到图形化界面
```

#### 7.3.4、监控图解

```java
实心圆：颜色的变化代表了实例的健康程度，绿色<黄色<橙色<红色递减 流量越大则实心圆就越大
曲线：用来记录2分钟内流量的相对变化
```

![](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/hysyrix.png)

## 第八章、Zuul路由网关

### 8.1、什么是Zuul

Spring Cloud Zuul 是整合Netflix公司的 Zuul开源项目（官方：https://github.com/Netflix/zuul）；

Zuul 包含了对**请求路由和校验过滤**两个最主要的功能：

 其中路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础，

 客户端请求网关/api/product，通过路由转发到 product

 服务客户端请求网关/api/order，通过路由转发到 order 服务

 而过滤功能则负责对请求的处理过程进行干预，是实现请求校验等功能的基础.

Zuul 和 Eureka 进行整合，将 Zuul 自身注册为 Eureka 服务治理中的服务，同时从 Eureka 中获得其他微服务的消息，也即以后的访问微服务都是通过Zuul跳转后获得。

 注意：Zuul服务最终还是会注册进Eure

## 8.2、搭建路由网关zuul

```java
1、引入pom坐标
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
  <!--zuul路由网关依赖-->
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-netflix-zuul</artifactId>
	</dependency>
</dependencies> 

2、application.yml配置文件

server:
  port: 7001
spring:
  application:
    name: spring-cloud-zuul

# 将服务提供者 注册到 Eureka服务注册中心
eureka:
  client:
    registerWithEureka: true # 服务注册开关
    fetchRegistry: true # 服务发现开关
    serviceUrl: # 客户端(服务提供者)注册到哪一个Eureka Server服务注册中心，多个用逗号分隔
      defaultZone: http://eureka6001.com:6001/eureka,http://eureka6002.com:6002/eureka
  instance:
    instanceId: ${spring.application.name}:${server.port} # 指定实例ID,就不会显示主机名了
    prefer-ip-address: true # 鼠标放到实例ID上后显示 ip地址

3、启动类新增 @EnableZuulProxy 注解 开启zuul功能
      
4、启动eureka注册中心、启动服务提供者、启动路由网关zuul，检查zuul是否注册到eureka

5、不用路由直接访问  http://localhost:8001/product/get

6、使用路由访问 注意端口为zuul端口 http://localhost:7001/spring-cloud-product/product/get/1
```

### 8.3、路由转发映射配置

```java
1、zuul服务端的application.yml配置文件追加

zuul:
  routes:
    povider-product:           # 路由名称，名称任意，保持所有路由名称唯一
      path: /product/**        # 访问路径
      serviceId: spring-cloud-product # 指定服务ID，会自动从Eureka中找到此服务的ip和端口
  stripPrefix: false 
  # 代理转发时去掉前缀，false:代理转发时不去掉前缀 例如:为true时请求 /product/get/1，代理转发到/get/1
  
  #如果多个服务需要经过路由，则同povider-product方式一样继续添加即可！

2、重启路由zuul服务

3、测试zuul路由功能
未配置路由转发映射，通过路由请求方式：http://localhost:7001/spring-cloud-product/product/get/1
已配置路由转发映射，通过路由请求方式：http://localhost:7001/product/product/get/1

product开头的路径，就会转发到spring-cloud-product实例的服务。
```

### 8.4、Zuul 过滤器实战

```java
Zuul核心就是过虑器，通过过虑器实现请求过虑，身份校验等。
实现步骤：自定义一个过滤器，并注入bean即可。

自定义过虑器需要继承 ZuulFilter，ZuulFilter是一个抽象类，需要覆盖它的4个方法，如下：
	filterType：返回字符串代表过滤器的类型，返回值有：
		pre：在请求路由之前执行
		route：在请求路由时调用
		post：请求路由之后调用， 也就是在route和errror过滤器之后调用
		error：处理请求发生错误时调用
	filterOrder：此方法返回整型数值，通过此数值来定义过滤器的执行顺序，数字越小优先级越高。
	shouldFilter：返回Boolean值，判断该过滤器是否执行。返回true表示要执行此过虑器，false不执行。
	run：过滤器的业务逻辑。
```

```java
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

/**
 * 自定义zuul过滤器
 */
@Component
@Slf4j
public class LoginFilter extends ZuulFilter {
    @Override
    public String filterType() {
        //请求路由前调用
        return "pre";
    }

    @Override
    public int filterOrder() {
        //int值来定义过滤器的执行顺序，数值越小优先级越高
        return 1;
    }

    @Override
    public boolean shouldFilter() {
        //该过滤器是否执行，true执行，false不执行
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext context = RequestContext.getCurrentContext();
        HttpServletRequest request = context.getRequest();
        //获取请求参数token的值
        String token = request.getParameter("token");
        if (token == null) {
            log.warn("此操作需要先登录系统...");
            context.setSendZuulResponse(false);// 拒绝访问
            context.setResponseStatusCode(200);// 设置响应状态码
            try {
                //响应结果
                context.getResponse().getWriter().write("token is empty");
            } catch (IOException e) {
                e.printStackTrace();
            }
            return null;
        }
        log.info("ok");
        return null;
    }
}
```

```java
测试：
	不带token访问：http://localhost:7001/product/product/get/1
	带token访问：http://localhost:7001/product/product/get/1?token=123

设置多个过滤器时，多个都会执行！
```

## 第九章、Spring Cloud Config 分布式配置中心
# **一、简介**

 Prometheus可以监控服务器情况，也可以监控应用服务内部情况。主要配合Grafana使用，Prometheus负责采集数据推送到Grafana,Grafana负载可视化展示。

官网：https://prometheus.io/

下载地址：https://prometheus.io/download/

# **二、Prometheus的特点**

Prometheus 是一个开源的完整监控解决方案，其对传统监控系统的测试和告警模型进行了彻底的颠覆，形成了基于中央化的规则计算、统一分析和告警的新模型。 相比于传统监控系统，Prometheus 具有以下优点：

## **1、易于管理**

Prometheus 核心部分只有一个单独的二进制文件，不存在任何的第三方依赖(数据库，缓存等等)。唯一需要的就是本地磁盘，因此不会有潜在级联故障的风险。

Prometheus 基于 Pull 模型的架构方式，可以在任何地方（本地电脑，开发环境，测试环境）搭建我们的监控系统。

对于一些复杂的情况，还可以使用 Prometheus 服务发现(Service Discovery)的能力动态管理监控目标。

## **2、监控服务的内部运行状态**

Pometheus 鼓励用户监控服务的内部状态，基于Prometheus 丰富的 Client 库，用户可以轻松的在应用程序中添加对Prometheus 的支持，从而让用户可以获取服务和应用内部真正的运行状态。

## **3、强大的数据模型**

所有采集的监控数据均以指标(metric)的形式保存在**内置的时间序列数据库**当中（TSDB，Time Series DB）。所有的样本除了基本的指标名称以外，还包含一组用于描述该样本特征的标签。

```java
http_request_status{
    code='200',
    content_path='/api/path',
    environment='produment'
} =>
[value1@timestamp1,value2@timestamp2...]

http_request_status{ # 指标名称
    code='200', # 维度的标签
    content_path='/api/path2',
    environment='produment'
} =>
[value1@timestamp1,value2@timestamp2...] # 存储的样本值
```

每一条时间序列由**指标名称(Metrics Name)**以及一组**标签(Labels)**唯一标识。每条时间序列按照时间的先后顺序存储一系列的样本值。

```java
http_request_status：指标名称(Metrics Name)

{code=‘200’,content_path=’/api/path’,environment=‘produment’}：表示维度的标签，基于这些 Labels 我们可以方便地对监控数据进行聚合，过滤，裁剪。

[value1@timestamp1,value2@timestamp2…]：按照时间的先后顺序存储的样本值。
```

## **4、强大的查询语言 PromQL**

Prometheus 内置了一个强大的数据查询语言 PromQL。 通过 PromQL 可以实现对监控数据的查询、聚合。同时 PromQL 也被应用于数据可视化(如 Grafana)以及告警当中。

通过PromQL 可以轻松回答类似于以下问题：

```
在过去一段时间中 95%应用延迟时间的分布范围？

预测在 4 小时后，磁盘空间占用大致会是什么情况？

CPU 占用率前 5 位的服务有哪些？(过滤)
```

## **5、高效**

对于监控系统而言，大量的监控任务必然导致有大量的数据产生。而Prometheus 可以高效地处理这些数据，对于单一Prometheus Server 实例而言它可以处理：

```
数以百万的监控指标
每秒处理数十万的数据点
```

## **6、可扩展**

可以在每个数据中心、每个团队运行独立的Prometheus Sevrer。Prometheus 对于联邦集群的支持，可以让多个 Prometheus 实例产生一个逻辑集群，当单实例 Prometheus Server 处理的任务量过大时，通过使用功能分区(sharding)+联邦集群(federation)可以对其进行扩展。

## **7、易于集成**

使用Prometheus 可以快速搭建监控服务，并且可以非常方便地在应用程序中进行集成。目前支持：Java，JMX，Python，Go，Ruby，.Net，Node.js 等等语言的客户端 SDK，基于这些 SDK 可以快速让应用程序纳入到 Prometheus 的监控当中，或者开发自己的监控数据收集程序。

同时这些客户端收集的监控数据，不仅仅支持 Prometheus，还能支持 Graphite 这些其他的监控工具。

同时Prometheus 还支持与其他的监控系统进行集成：Graphite，Statsd，Collected， Scollector， muini， Nagios 等。 Prometheus 社区还提供了大量第三方实现的监控数据采集支持：JMX，CloudWatch，EC2，MySQL，PostgresSQL，Haskell，Bash，SNMP， Consul，Haproxy，Mesos，Bind，CouchDB，Django，Memcached，RabbitMQ，Redis，RethinkDB，Rsyslog 等等。

## **8、可视化**

Prometheus Server 中自带的 `Prometheus UI`，可以方便地直接对数据进行查询，并且支持直接以图形化的形式展示数据。同时 Prometheus 还提供了一个独立的基于Ruby On Rails 的 Dashboard 解决方案 Promdash。

最新的 `Grafana` 可视化工具也已经提供了完整的Prometheus 支持，基于 Grafana 可以创建更加精美的监控图标。

基于Prometheus 提供的API 还可以实现`自己的监控可视化UI`。

## **9、开放性**

通常来说当我们需要监控一个应用程序时，一般需要该应用程序提供对相应监控系统协议的支持，因此应用程序会与所选择的监控系统进行绑定。为了减少这种绑定所带来的限制，对于决策者而言要么你就直接在应用中集成该监控系统的支持，要么就在外部创建单独的服务来适配不同的监控系统。

而对于 Prometheus 来说，使用 Prometheus 的 client library 的输出格式不止支持Prometheus 的格式化数据，也可以输出支持其它监控系统的格式化数据，比如 Graphite。因此你甚至可以在不使用Prometheus 的情况下，采用 Prometheus 的 client library 来让你的应用程序支持监控数据采集。


# **三、Prometheus 的架构**

![img](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/d6476d493d8232e51931cdae138f05f4.png)

prometheus 负责从 `pushgateway` 和 `job` 中采集数据， 存储到后端 `Storatge` 中，可以通过`PromQL` 进行查询， 推送 alerts 信息到 `AlertManager`。 AlertManager 根据不同的路由规则进行报警通知

## **1、存储计算层**

```
Prometheus Server，里面包含了存储引擎和计算引擎。
Retrieval 组件为取数组件，它会主动从 PushGateway 或者 Exporter 拉取指标数据。
Service discovery，可以动态发现要监控的目标。
TSDB，数据核心存储与查询。
HTTP server，对外提供 HTTP 服务。
```

## **2、采集层**

```
采集层分为两类，一类是生命周期较短的作业，还有一类是生命周期较长的作业。
短作业：直接通过 API，在退出时间指标推送给PushGateway 。(一般检查不固定的服务)
长作业：Retrieval 组件直接从 Job 或者 Exporter 拉取数据。(一般检查固定服务器的服务)
```

## **3、应用层**

应用层主要分为两种，一种是AlertManager（报警），另一种是数据可视化。

```
AlertManager
	对接Pagerduty，是一套付费的监控报警系统。可实现短信报警、5 分钟无人 ack 打电话通知、仍然无人 ack，通知值班人员 Manager…Emial，发送邮件… 
数据可视化
	Prometheus build-in WebUI、Grafana、其他基于API 开发的客户端
```

# 四、Prometheus 的安装

官网：https://prometheus.io/

下载地址：https://prometheus.io/download/

## 1、prometheus安装

```
1、上传 prometheus-2.29.1.linux-amd64.tar.gz
2、tar  -zxvf  prometheus-2.29.1.linux-amd64.tar.gz -C /opt/module
3、修改配置文件prometheus.yml
```

```yml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["127.0.0.1:9090"]
    
# 添加 PushGateway 监控配置
  - job_name: 'pushgateway'
    static_configs:
    - targets: ['127.0.0.1:9091']
      labels:
        instance: pushgateway
        
# 添加 Node Exporter 监控配置
  - job_name: 'node exporter'
    static_configs:
    - targets: ['127.0.0.1:9100']

```

## 2、node_exporter安装

```java
1、上传 node_exporter-1.2.2.linux-amd64.tar.gz 
2、解压文件 tar  -zxvf  node_exporter-1.2.2.linux-amd64.tar.gz  -C  /opt/module
3、启动服务 nohup ./node_exporter > node_exporter.log 2>&1 &
4、访问 IP:9100/metrics 即可获取服务器参数
```

![image-20220615220203001](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220615220203001.png)

## 3、prometheus启动

```java
1、进入prometheus目录
2、启动服务 nohup ./prometheus --config.file=prometheus.yml >./prometheus.log 2>&1 & 后台启动
3、访问 IP:9090
```

![image-20220615215434281](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220615215434281.png)

## 4、pushgateway 安装与启动

```java
1、上传 pushgateway-1.4.3.linux-amd64.tar.gz 
2、解压文件 tar  -zxvf  pushgateway-1.4.3.linux-amd64.tar.gz   -C  /opt/module
3、启动服务 nohup ./pushgateway --web.listen-address:9091 >./pushgateway.log 2>&1 &
4、访问 IP:9091/metrics
```

至此，所有服务均已启动

![image-20220615221701595](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220615221701595.png)

# 五、PromQL介绍

Prometheus 通过指标名称（metrics name）以及对应的一组标签（labelset）唯一定义一条时间序列。指标名称反映了监控样本的基本标识，而 label 则在这个基本特征上为采集到的数据提供了多种特征维度。用户可以基于这些特征维度过滤，聚合，统计从而产生新的计算后的一条时间序列。

PromQL 是 Prometheus 内置的数据查询语言，其提供对时间序列数据丰富的查询，聚合以及逻辑运算能力的支持。并且被广泛应用在Prometheus的日常应用当中，包括对数据查询、可视化、告警处理当中。

PromQL 是Prometheus 所有应用场景的基础


![image-20220615224314639](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220615224314639.png)

在首页输入框内，输入指标名称，即可查询。也可以编写类似sql语言格式，查询指标数据。

所有指标 IP:9090/metrics 即可查询。或者点击输入框右侧按钮

![image-20220615224540078](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220615224540078.png)

## 1、基本用法

### **查询时间序列**

当 Prometheus 通过 Exporter 采集到相应的监控指标样本数据后，我们就可以通过PromQL 对监控样本数据进行查询。

当我们直接使用监控指标名称查询时，可以查询该指标下的所有时间序列。如：

```
prometheus_http_requests_total                  
等同于： 
prometheus_http_requests_total{}     
```

该表达式会返回指标名称为 prometheus_http_requests_total 的所有时间序列：

```
prometheus_http_requests_total{
    code="200",
    handler="alerts",
    instance="localhost:9090",
    job="prometheus",
    method="get"
}= (20889@1518096812.326)

prometheus_http_requests_total{
    code="200",
    handler="graph",
    instance="localhost:9090",
    job="prometheus",
    method="get"
}= (21287@1518096812.326)
```

PromQL 还支持用户根据时间序列的标签匹配模式来对时间序列进行过滤，目前主要支持两种匹配模式：完全匹配和正则匹配。

PromQL 支持使用 = 和 != 两种完全匹配模式：

通过使用 label=value 可以选择那些标签满足表达式定义的时间序列；
反之使用 label!=value 则可以根据标签匹配排除时间序列；

例如，如果我们只需要查询所有 prometheus_http_requests_total 时间序列中满足标签 instance 为 localhost:9090 的时间 序列，则可以使用如下表达式：

```
prometheus_http_requests_total{instance="localhost:9090"}       
```

反之使用 instance!=“localhost:9090” 则可以排除这些时间序列：

```
prometheus_http_requests_total{instance!="localhost:9090"}
```

PromQL 还可以支持使用正则表达式作为匹配条件，多个表达式之间使用 | 进行分离：

使用 label=~regx 表示选择那些标签符合正则表达式定义的时间序列；
反之使用 label!~regx 进行排除；

例如，如果想查询多个环节下的时间序列序列可以使用如下表达式：

```
prometheus_http_requests_total{environment=~"staging|testing|development",method!="GET"}
排除用法
prometheus_http_requests_total{environment!~"staging|testing|development",method!="GET"}

```

### **范围查询**

直接通过类似于 PromQL 表达式 httprequeststotal 查询时间序列时，返回值中只会包含该时间序列中的最新的一个样本值，这样的返回结果我们称之为瞬时向量。而相应的这样的表达式称之为 瞬时向量表达式。

而如果我们想过去一段时间范围内的样本数据时，我们则需要使用区间向量表达式。区间向量表达式和瞬时向量表达式之间的差异在于在区间向量表达式中我们需要定义时间选择的范围，时间范围通过时间范围选择器 [] 进行定义。 例如，通过以下表达式可以选择

最近 5 分钟内的所有样本数据：

```
prometheus_http_requests_total{}[5m]
```

通过区间向量表达式查询到的结果我们称为区间向量。 除了使用 m 表示分钟以外，

PromQL 的时间范围选择器支持其它时间单位：

```
s - 秒
m - 分钟
h - 小时
d - 天
w - 周
y - 年
```

### 时间位移操作

在瞬时向量表达式或者区间向量表达式中，都是以当前时间为基准：

```
prometheus_http_requests_total{} # 瞬时向量表达式，选择当前最新的数据 
prometheus_http_requests_total{}[5m] # 区间向量表达式，选择以当前时间为基准，5 分钟内的数据
```

而如果我们想查询，5 分钟前的瞬时样本数据，或昨天一天的区间内的样本数据呢? 这个时候我们就可以使用位移操作，位移操作的关键字为 **offset**。 可以使用 offset 时间位移操作：

```
prometheus_http_requests_total{} offset 5m
prometheus_http_requests_total{}[1d] offset 1d
```

### 使用聚合操作

一般来说，如果描述样本特征的标签(label)在并非唯一的情况下，通过 PromQL 查询数据，会返回多条满足这些特征维度的时间序列。而 PromQL 提供的聚合操作可以用来对这些时间序列进行处理，形成一条新的时间序列：

```
# 查询系统所有 http 请求的总量
sum(prometheus_http_requests_total)
# 按照 mode 计算主机 CPU 的平均使用时间
avg(node_cpu_seconds_total) by (mode)
# 按照主机查询各个主机的 CPU 使用率
sum(sum(irate(node_cpu_seconds_total{mode!='idle'}[5m])) /  sum(irate(node_cpu_seconds_total [5m]))) by (instance)
```

###  标量和字符串

除了使用瞬时向量表达式和区间向量表达式以外，PromQL 还直接支持用户使用标量(Scalar)和字符串(String)。

➢ 标量（Scalar）：一个浮点型的数字值

标量只有一个数字，没有时序。 例如：10

需要注意的是，当使用表达式 count(prometheus_http_requests_total)，返回的数据类型，依然是瞬时向量。用户可以通过内置函数scalar()将单个瞬时向量转换为标量。

➢ 字符串（String）：一个简单的字符串值

直接使用字符串，作为PromQL 表达式，则会直接返回字符串。

```
"this is a string"
'these are unescaped: \n \\ \t'
`these are not unescaped: \n ' " \t
```

### 合法的 PromQL 表达式

所有的 PromQL 表达式都**必须至少包含一个指标名称**(例如 http_request_total)，或者一个**不会匹配到空字符串**的标签过滤器(例如{code=”200”})。

```
因此以下两种方式，均为合法的表达式：
prometheus_http_requests_total # 合法
prometheus_http_requests_total{} # 合法
{method="get"} # 合法

而如下表达式，则不合法：
{job=~".*"} # 不合法

同时，除了使用  {label=value}  的形式以外，我们还可以使用内置的 name 标签来指定监控指标名称：
{  name  =~"prometheus_http_requests_total"} # 合法
{  name  =~"node_disk_bytes_read|node_disk_bytes_written"} # 合法
```

### PromQL 操作符

使用PromQL 除了能够方便的按照查询和过滤时间序列以外，PromQL 还支持丰富的操作符，用户可以使用这些操作符对进一步的对事件序列进行二次加工。这些操作符包括：数学运算符，逻辑运算符，布尔运算符等等。

```
数学运算
+ (加法)
- (减法)
* (乘法)
/ (除法)
% (求余)
^ (幂运算)

布尔运算
== (相等)
!= (不相等)
>(大于)
< (小于)
>= (大于等于)
<= (小于等于)

```

➢ 使用bool 修饰符改变布尔运算符的行为

布尔运算符的默认行为是对时序数据进行过滤。而在其它的情况下我们可能需要的是真正的布尔结果。例如，只需要 知道当前模块的 HTTP 请求量是否>=1000，如果大于等于1000 则返回 1（true）否则返回 0（false）。这时可以使 用 bool 修饰符改变布尔运算的默认行为。 例如：

prometheus_http_requests_total > bool 1000
使用 bool 修改符后，布尔运算不会对时间序列进行过滤，而是直接依次瞬时向量中的各个样本数据与标量的比较结果 0 或者 1。从而形成一条新的时间序列。

同时需要注意的是，如果是在两个标量之间使用布尔运算，则必须使用 bool 修饰符：2 == bool 2 # 结果为 1

### 使用集合运算符

使用瞬时向量表达式能够获取到一个包含多个时间序列的集合，我们称为瞬时向量。通过集合运算，可以在两个瞬时向量与瞬时向量之间进行相应的集合操作。

目前，Prometheus 支持以下集合运算符：

```
and (并且)
or (或者)
unless (排除)
```

vector1 and vector2 会产生一个由 vector1 的元素组成的新的向量。该向量包含vector1 中完全匹配 vector2 中的元素组成。

vector1 or vector2 会产生一个新的向量，该向量包含 vector1 中所有的样本数据，以及 vector2 中没有与 vector1 匹配到的样本数据。

vector1 unless vector2 会产生一个新的向量，新向量中的元素由 vector1 中没有与vector2 匹配的元素组成。

### 操作符优先级

对于复杂类型的表达式，需要了解运算操作的运行优先级。例如，查询主机的 CPU 使用率，可以使用表达式：

```
100 * (1 - avg (irate(node_cpu_seconds_total{mode='idle'}[5m])) by(job) ) 
```

其中irate 是PromQL中的内置函数，用于计算区间向量中时间序列每秒的即时增长率。在 PromQL 操作符中优先级由高到低依次为：

```
^
*, /, %
+, -
==, !=, <=, =, >
and, unless
or
```

### PromQL 聚合操作

Prometheus 还提供了下列内置的聚合操作符，这些操作符作用域瞬时向量。可以将瞬时表达式返回的样本数据进行 聚合，形成一个新的时间序列。

```
sum (求和)
min (最小值)
max (最大值)
avg (平均值)

stddev (标准差)
stdvar (标准差异)
count (计数)
count_values (对 value 进行计数)
bottomk (后 n 条时序)
topk (前n 条时序)
quantile (分布统计)
```

使用聚合操作的语法如下：

```
<aggr-op>([parameter,] <vector expression>) [without|by (<label list>)] 
```


其中只有 count_values , quantile , topk , bottomk 支持参数(parameter)。

without 用于从计算结果中移除列举的标签，而保留其它标签。by 则正好相反，结果向量中只保留列出的标签，其余标签则移除。通过 without 和 by 可以按照样本的问题对数据进行聚合。

例如：

```
sum(prometheus_http_requests_total) without (instance)                  等价于
sum(prometheus_http_requests_total) by (code,handler,job,method)       
```

如果只需要计算整个应用的 HTTP 请求总量，可以直接使用表达式：sum(prometheus_http_requests_total)

count_values 用于时间序列中每一个样本值出现的次数。count_values 会为每一个唯一的样本值输出一个时间序列，并且每一个时间序列包含一个额外的标签。 例如：count_values(“count”, prometheus_http_requests_total)

topk 和 bottomk 则用于对样本值进行排序，返回当前样本值前 n 位，或者后 n 位的时间序列。

获取 HTTP 请求数前 5 位的时序样本数据，可以使用表达式：topk(5, prometheus_http_requests_total)

quantile 用于计算当前样本数据值的分布情况 quantile(φ, express)其中 0 ≤ φ ≤ 1。

例如，当φ 为 0.5 时，即表示找到当前样本数据中的中位数：quantile(0.5, prometheus_http_requests_total)

# 六、Prometheus 和 Grafana 集成

官网下载安装：https://grafana.com/grafana/download

![image-20220615225957649](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220615225957649.png)

### 1、安装

```
wget https://dl.grafana.com/enterprise/release/grafana-enterprise-8.5.5-1.x86_64.rpmsudo 
yum install grafana-enterprise-8.5.5-1.x86_64.rpm
```

### 2、启动服务使用 

```
systemd 启动服务器  
sudo systemctl daemon-reload   
sudo systemctl start grafana-server   
sudo systemctl status grafana-server

后台启动
nohup ./bin/grafana-server web > ./grafana.log 2>&1 &
# 打开web：http://hadoop202:3000,默认用户名和密码：admin
```

### 3、配置数据源

![image-20220615230303285](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220615230303285.png)

![image-20220615230459521](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220615230459521.png)

### 4、手动创建仪表盘 Dashboard

![image-20220616214322877](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220616214322877.png)

![image-20220616214536938](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220616214536938.png)

![image-20220616214800778](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220616214800778.png)

![image-20220616214913538](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220616214913538.png)

一个仪表板可以配置多个监控项，添加其他监控项：

![image-20220616215134283](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220616215134283.png)

创建多个panel后，可拖动图表，放到其他row中

![image-20220616215907029](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220616215907029.png)

### 5、 Dashboard 模板

手动一个个添加 Dashboard 比较繁琐，Grafana 社区鼓励用户分享 Dashboard，通过https://grafana.com/dashboards 网站，可以找到大量可直接使用的 Dashboard 模板。

Grafana 中所有的Dashboard 通过 JSON 进行共享，下载并且导入这些 JSON 文件，就可以直接使用这些已经定义好的 Dashboard


![image-20220616222321846](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220616222321846.png)

下载JSON文件

![image-20220616222344832](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220616222344832.png)

导入模板

![image-20220616222414302](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220616222414302.png)

![image-20220616222619951](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220616222619951.png)

![image-20220616222803123](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220616222803123.png)

# 七、实战

## 1、监控RabbitMq

第一种：RabbitMQ内部集成Prometheus来获取指标

- 3.8.0之前版本，RabbitMQ可以使用单独的插件prometheus_rabbitmq_exporter来向Prometheus公开指标，要单独下载到RabbitMQ安装目录中进行安装；

　　　prometheus_rabbitmq_exporter：https://github.com/deadtrickster/prometheus_rabbitmq_exporter

- 3.8.0版开始，RabbitMQ附带了内置的Prometheus＆Grafana支持。虽然内置了该插件，但也要进行安装

　　  rabbitmq-prometheus：https://github.com/rabbitmq/rabbitmq-prometheus

第二种：使用独立程序来获取指标（RabbitMQ_exporter）

　　不管什么版本都能使用，要单独启动exporter进程

　　rabbitmq_exporter：https://github.com/kbudde/rabbitmq_exporter

RabbitMQ 官方监控介绍： 

- https://www.rabbitmq.com/monitoring.html
- https://www.rabbitmq.com/prometheus.html#overview-prometheus

**接下来采用第二种方式**

安装rabbitmq_exporter

注：在RabbitMQ集群下的任意一个节点部署它。

- **上传解压**

从 https://github.com/kbudde/rabbitmq_exporter/releases 下载并上传rabbitmq_exporter_1.0.0-RC16_linux_amd64.tar.gz安装包并解压到/usr/local目录

```
tar -xvf rabbitmq_exporter_1.0.0-RC16_linux_amd64.tar.gz
cd rabbitmq_exporter_1.0.0-RC16_linux_amd64/
```

- **配置**

使用默认配置

- **启动**

进入根目录下，输入以下命令：

```
cd /usr/local/rabbitmq_exporter_1.0.0-RC16_linux_amd64
RABBIT_USER=zat RABBIT_PASSWORD=zat123 OUTPUT_FORMAT=json PUBLISH_PORT=9090 RABBIT_URL=http://localhost:15672 nohup ./rabbitmq_exporter &
tail -1000f nohup.out
```

参数说明：

RABBIT_USER：rabbit用户名

RABBIT_PASSWORD：rabbit密码

RABBIT_URL：rabbit服务地址和端口

OUTPUT_FORMAT：输出格式

PUBLIC_PORT：暴露端口

启动成功后，可以访问 http://IP:9090/metrics/ ，（IP和端口要改成相应环境的）

看抓取的信息如下：

![image-20220617095924813](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220617095924813.png)

**Prometheus配置**

修改prometheus组件的prometheus.yml加入rabbitMQ节点：

```yml
    
# 添加 rabbitmq_export 监控配置
  - job_name: 'rabbitmq_export'
    scrape_interval: 60s
    scrape_timeout: 60s
    static_configs:
    - targets: ['127.0.0.1:19090']
```

先kill掉Prometheus进程，用以下命令重启它，然后查看targets：

**Grafana配置**

下载对应的模板，不需要修改即可使用

![image-20220617104125774](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220617104125774.png)

![image-20220617104148302](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220617104148302.png)
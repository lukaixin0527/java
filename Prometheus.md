# **一、简介**

 Prometheus可以监控服务器情况，也可以监控应用服务内部情况。主要配合Grafana使用，Prometheus负责采集数据推送到Grafana,Grafana负载可视化展示。

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

```
1、上传 node_exporter-1.2.2.linux-amd64.tar.gz 
2、tar  -zxvf  node_exporter-1.2.2.linux-amd64.tar.gz  -C  /opt/module
3、./node_exporter
4、访问 IP:9090即可获取服务器参数
```


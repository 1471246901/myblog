# consul简介与安装

[consul文档](https://www.consul.io/intro/index.html)

## 简介

### 什么是consul

Consul is a service mesh solution providing a full featured control plane with service discovery, configuration, and segmentation functionality. Each of these features can be used individually as needed, or they can be used together to build a full service mesh. Consul requires a data plane and supports both a proxy and native integration model. Consul ships with a simple built-in proxy so that everything works out of the box, but also supports 3rd party proxy integrations such as Envoy.

>   Consul是一个服务网格解决方案，它提供了一个功能齐全的控制平面，具有服务发现、配置和分段功能。这些特性中的每一个都可以根据需要单独使用，也可以一起用于构建全服务网格。Consul需要一个数据平面，并支持代理和本机集成模型。consul与一个简单的内置代理，使一切工作的开箱即用，但也支持第三方代理集成，如Envoy。

### consul的主要特点是

-   **Service Discovery**: Clients of Consul can register a service, such as or , and other clients can use Consul to discover providers of a given service. Using either DNS or HTTP, applications can easily find the services they depend upon.`api mysql`

    提供HTTP/DNS两种服务发现方式

-   **Health Checking**: Consul clients can provide any number of health checks, either associated with a given service ("is the webserver returning 200 OK"), or with the local node ("is memory utilization below 90%"). This information can be used by an operator to monitor cluster health, and it is used by the service discovery components to route traffic away from unhealthy hosts.

    支持多种方式,HTTP、TCP、Docker、shell脚本定制化

-   **KV Store**: Applications can make use of Consul's hierarchical key/value store for any number of purposes, including dynamic configuration, feature flagging, coordination, leader election, and more. The simple HTTP API makes it easy to use.

    支持Key Value 存储

-   **Secure Service Communication**: Consul can generate and distribute TLS certificates for services to establish mutual TLS connections. [Intentions](https://www.consul.io/docs/connect/intentions) can be used to define which services are allowed to communicate. Service segmentation can be easily managed with intentions that can be changed in real time instead of using complex network topologies and static firewall rules.

-   **Multi Datacenter**: Consul supports multiple datacenters out of the box. This means users of Consul do not have to worry about building additional layers of abstraction to grow to multiple regions.

    多数据中心与可视化界面

>   -   **服务发现**：consul的客户可以注册服务，如 或 ，其他客户端可以使用consul来发现给定服务的提供者。使用 DNS 或 HTTP，应用程序可以轻松找到它们所依赖的服务。`api``mysql`
>   -   **运行状况检查**：Consul 客户端可以提供与给定服务关联的任数量运行状况检查（"返回 200 OK 的 Web 服务器"），或与本地节点（"内存利用率低于 90%"） 关联的运行状况检查。操作员可以使用此信息来监视群集运行状况，服务发现组件也使用此信息将流量路由到不正常的主机。
>   -   **KV 存储**：应用程序可以使用 Consul 的分层密钥/值存储用于任何多个目的，包括动态配置、功能标记、协调、领导者选择等。简单的 HTTP API 使其易于使用。
>   -   **安全服务通信**：consul可以生成和分发 TLS 证书的服务，以建立相互的 TLS 连接。[意图](https://www.consul.io/docs/connect/intentions)可用于定义允许哪些服务进行通信。服务分段可以方便地使用可以实时更改的意图进行管理，而不是使用复杂的网络拓扑和静态防火墙规则。
>   -   **多数据中心**：consul支持开箱即用的多个数据中心。这意味着 Consul 的用户不必担心构建额外的抽象层来增长到多个区域。

## 安装

### 下载链接

[https://www.consul.io/downloads.html](https://www.consul.io/downloads.html)

本人下载的windows版本 

[安装和学习](https://www.springcloud.cc/spring-cloud-consul.html)

下载解压完成之后只有consul.exe 一个文件,可以直接在命令行下使用`consul --version` 来查看版本,使用 `consul agent -dev` 来以开发者模式启动,启动之后可以在本地8500端口访问到consul的首页

![image-20210131155329140](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210131155329140.png)

![image-20210131155359738](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210131155359738.png)

consul集群搭建 见[consul分布式集群搭建](https://www.cnblogs.com/duanxz/p/10564502.html)
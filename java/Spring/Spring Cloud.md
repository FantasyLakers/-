## 1、Spring Cloud主要组件

### Spring Cloud Netflix旗下产品

- Eureka：Eureka服务注册和发现组件，分成两类，一是注册中心及EurekaServer，用于提供服务注册、服务申请等功能。另一个是被注册者及服务提供者EurekaClient，用于向EurekaServer注册服务并可从EurekaServer获取需要调用的服务地址信息。需要向外提供服务的应用，需要使用EurekaClient来向Server注册服务。
- Ribbon：负责进行客户端负载均衡的组件，一般与RestTemplate结合，在访问EurekaClient提供的服务时进行负载均衡处理。也就是说，Ribbon用于服务调用者向被调用者进行服务调用，并且如果服务者有多个节点时，会进行客户端的负载均衡处理
- Hystrix：Hystrix是一个熔断组件，它除了一些基本的熔断器功能外，还能够实现服务降级、服务限流的功能。另外Hystrix提供了熔断器的健康监测，以及熔断器健康数据的API接口。Hystrix Dashboard组件提供了单个服务熔断器的健康状态数据的界面展示功能，Hystrix Turbine组件提供了多个服务的熔断器的健康状态数据的界面展示功能
- Zuul：路由网关Zuul有智能路由和过滤的功能。内部服务的API接口通过Zuul网关统一对外暴露，内部服务的API接口不直接暴露，防止了内部服务敏感信息对外暴露。Zuul的过滤功能是通过拦截请求来实现的，可以对一些用户的角色和权限进行判断，起到安全验证的作用，同时也可以用于输出实时的请求日志

### Spring Cloud OpenFeign 声明式远程调度

与Ribbon功能类似，用于调用方与被调用方的服务调用，同时进行负载均衡的处理。不过它能提供类似本地调用的方式调用远程的EurekaClient提供的服务。它实际上在Ribbon基础上进行了进一步的封装来提供调用服务的简便性。

### Spring Cloud Consul

Consul 是 HashiCorp 公司推出的开源产品，用于实现分布式系统的服务发现、服务隔离、服务配置，这些功能中的每一个都可以根据需要单独使用，也可以同时使用所有功能。Consul 官网目前主要推 Consul 在服务网格中的使用。与其它分布式服务注册与发现的方案相比，Consul 的方案更“一站式”——内置了服务注册与发现框架、分布一致性协议实现、健康检查、Key/Value 存储、多数据中心方案，不再需要依赖其它工具。Consul 本身使用 go 语言开发，具有跨平台、运行高效等特点，也非常方便和 Docker 配合使用。

### Spring Cloud Gateway

Spring Cloud Gateway 是 Spring 官方基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关，Spring Cloud Gateway 旨在为微服务架构提供一种简单而有效的统一的 API 路由管理方式，统一访问接口。Spring Cloud Gateway 作为 Spring Cloud 生态系中的网关，目标是替代 Netflix ZUUL，其不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/埋点，和限流等。它是基于Nttey的响应式开发模式。

### Spring Cloud Config 配置中心

Spring Cloud Config组件提供了配置文件统一管理的功能。Spring Cloud Config包括Server端和Client端，Server端读取本地仓库或者远程仓库的配置文件，所有的Client从Server读取配置信息，从而达到配置文件统一管理的目的。

### Spring Cloud Bus 消息总线

Spring Cloud总线使用轻量级消息代理链接分布式系统的节点。这可用于广播状态更改（例如配置更改）或其他管理指令。该项目为AMQP代理或Kafka提供了入门服务。



以下Spring Cloud Netflix模块和相应的启动器将进入维护模式（将模块置于维护模式意味着Spring Cloud团队将不再向模块添加新功能）：

- spring-cloud-netflix-archaius
- spring-cloud-netflix-hystrix-contract
- spring-cloud-netflix-hystrix-dashboard
- spring-cloud-netflix-hystrix-stream
- spring-cloud-netflix-hystrix
- spring-cloud-netflix-ribbon
- spring-cloud-netflix-turbine-stream
- spring-cloud-netflix-turbine
- spring-cloud-netflix-zuul



## 2、其它Spring Cloud 产品

### Spring Cloud Alibaba

Spring Cloud Alibaba provides a one-stop solution for distributed application development. It contains all the components required to develop distributed applications, making it easy for you to develop your applications using Spring Cloud.

With Spring Cloud Alibaba, you only need to add some annotations and a small amount of configurations to connect Spring Cloud applications to the distributed solutions of Alibaba, and build a distributed application system with Alibaba middleware.

### Spring Cloud Zookeeper

Service discovery and configuration management with Apache Zookeeper.

### Spring Cloud Kubernetes

Spring Cloud Kubernetes provide Spring Cloud common interface implementations that consume Kubernetes native services. The main objective of the projects provided in this repository is to facilitate the integration of Spring Cloud and Spring Boot applications running inside Kubernetes.

### Spring Cloud Cluster

Leadership election and common stateful patterns with an abstraction and implementation for Zookeeper, Redis, Hazelcast, Consul.


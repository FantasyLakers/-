# Spring Cloud

## 1、Spring Cloud主要组件

### Eureka 服务注册和发现组件

分成两类，一是注册中心及EurekaServer，用于提供服务注册、服务申请等功能。另一个是被注册者及服务提供者EurekaClient，用于向EurekaServer注册服务并可从EurekaServer获取需要调用的服务地址信息。需要向外提供服务的应用，需要使用EurekaClient来想Server注册服务。

### Ribbon 负载均衡组件

负责进行客户端负载均衡的组件，一般与RestTemplate结合，在访问EurekaClient提供的服务时进行负载均衡处理。也就是说，Ribbon用于服务调用者向被调用者进行服务调用，并且如果服务者有多个节点时，会进行客户端的负载均衡处理。

### Feign 声明式远程调度

与Ribbon功能类似，用于调用方与被调用方的服务调用，同时进行负载均衡的处理；不过它能提供类似本地调用的方式调用远程的EurekaClient提供的服务；它实际上在Ribbon基础上进行了进一步的封装来提供调用服务的简便性。

### Hystrix 熔断组件

Hystrix是一个熔断组件，它除了一些基本的熔断器功能外，还能够实现服务降级、服务限流的功能。另外Hystrix提供了熔断器的健康监测，以及熔断器健康数据的API接口。Hystrix Dashboard组件提供了单个服务熔断器的健康状态数据的界面展示功能，Hystrix Turbine组件提供了多个服务的熔断器的健康状态数据的界面展示功能。

### Zuul 路由网关

路由网关Zuul有智能路由和过滤的功能。内部服务的API接口通过Zuul网关统一对外暴露，内部服务的API接口不直接暴露，防止了内部服务敏感信息对外暴露。Zuul的过滤功能是通过拦截请求来实现的，可以对一些用户的角色和权限进行判断，起到安全验证的作用，同时也可以用于输出实时的请求日志。

### Spring Cloud Config 配置中心

Spring Cloud Config组件提供了配置文件统一管理的功能。Spring Cloud Config包括Server端和Client端，Server端读取本地仓库或者远程仓库的配置文件，所有的Client从Server读取配置信息，从而达到配置文件统一管理的目的。

### Spring Cloud Bus 消息总线

消息总线，配置Spring Cloud Config用于动态刷新服务的配置

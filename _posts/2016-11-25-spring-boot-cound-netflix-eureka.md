---
layout: post
title:  "spring-cloud-netflix之------Eureka（服务注册与发现）"
date:   2016-11-25
desc: "spring-cloud-netflix之------Eureka（服务注册与发现）"
keywords: "spring-boot spring-cloud spring-cloud-eureka"
categories: [microservice]
tags: [spring-cloud,spring-cloud-eureka]
icon: fa-bookmark-o
---
spring-cloud-netflix之------Eureka（服务注册与发现）
========

> 简述
spring-cloud-netflix-eureka是spring cloud 集成netflix oss套件eureka这个轻量级的服务注册发现技术，通过集成，简化了eureka的使用。同时默认spring-cloud-netflix-eureka包含了Ribbon的依赖，可以很容易得配置eureka做负载均衡实现。

# Eureka

## Eureka概述

Eureka是netflix oss提供的rest形式的服务，主要用于服务发现以及负载均衡，Eureka包含两部分Eureka server服务端，服务端是一个简单的web应用，可以部署到tomcat运行，同时暴露服务地址提供给其他服务注册。Eureka Client客户端，客户端将自己注册到Eureka服务供其他客户端服务发现，同时自己也可以发现注册到Euraka的服务。

## Eureka原理

- 微服务启动后， 会周期性地向Eureka Server发送心跳（ 默认周期为30秒） 以续约自己的信息。 如果 Eureka Server在一定时间内没有接收到某个微服务节点的心跳， Eureka Server将会注销该微服务节点（ 默认90秒） ；

- 每个Eureka Server同时也是Eureka Client， 多个Eureka Server之间通过复制的方式完成服务注册表的
同步；

- Eureka Client会缓存Eureka Server中的信息。 即使所有的Eureka Server节点都宕掉， 服务消费者依然
可以使用缓存中的信息找到服务提供者。

> 综上， Eureka通过心跳检测、 健康检查和客户端缓存等机制， 提高了系统的灵活性、 可伸缩性和可用性。

## Eureka架构图

Eureka架构图：

![Eureka架构图](https://github.com/Netflix/eureka/raw/master/images/eureka_architecture.png)

通过上图可以看到

- Eureka Client通过Register注册到EurekaServer中，通过renew更新服务端注册信息。
- Eureka 服务可以做集群，多个Eureka服务之间可以互相同步（replicate）注册信息。
- Eureka Client互相访问的时候，先通过Get Registry获取到应用服务地址然后再通过直接远程调用（make remote call）。
- Eureka client访问服务时get registry过程，可以从Eureka Server中获取相同服务名的多个实例，这时候可以通过集成Ribbon做负载均衡，找状态最好的服务来处理请求。

## 为什么使用它

引用官网的话**Eureka is a REST (Representational State Transfer) based service that is primarily used in the AWS cloud for locating services for the purpose of load balancing and failover of middle-tier servers**，实现了服务之间的发现注册，使服务之间的调用变简单，同时提供了负载均衡功能，使服务之间的调用效率更高。

# spring-cloud Eureka

## 概述

spring-cloud对Eureka进行了集成，包括Eureka客户端与服务端，同时客户端的包里面默认包含了Ribbon的依赖，使得服务间调用负载均衡变得非常简单。


## 实现

- server实现：

1. 创建spring-boot的应用并添加对spring-cloud-starter-eureka-server的引用
		```
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-starter-eureka-server</artifactId>
			</dependency>
		```

2. 程序中加入注释@EnableEurekaServer
		```
			@SpringBootApplication
			@EnableEurekaServer
			public class EurekaserverApplication
			{
				public static void main(String[] args)
				{
					SpringApplication.run(EurekaserverApplication.class, args);
				}
			}
		```
3. 添加属性配置文件application.yml
		```
		server:
	  port: 8761
	spring:
	  application:
	    name: springcloudereurekaserver
	eureka:
	  instance:
	    hostname: localhost

	  client:
	    registerWithEureka: false
	    fetchRegistry: false
	    serviceUrl:
	      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

		```

4.	启动程序，并打开浏览器输入地址[http://localhost:8761](http://localhost:8761),服务状态页面，可以查看已经注册到服务的服务信息。



- client端实现：
客户端由两种service，userservice与movieservice，movieservice调用userservice。


1.	创建maven项目，添加pom引用。
		```
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		```
2.	userservice，通过`@EnableDiscoveryClient`声明为Eureka并注册到Eureka服务端。
	```
@Configuration
@ComponentScan
@EnableAutoConfiguration
@RestController
@EnableDiscoveryClient
public class UserService {

    //@Autowired
    //private EurekaClient discoveryClient;
    @Value("${fuck}")
    String fuck;


    @Autowired
    private Environment env;
    @RequestMapping("/message")
    public String hello()
    {
        try {
            sleep(1000);
            System.out.print("progress 1s");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //InstanceInfo instance=discoveryClient.getNextServerFromEureka("eurekaserver",true);
        return "hellofff"+fuck+"by userservice:"+env.getProperty("server.port");

    }
	public static void main(String[] args) {
        new SpringApplicationBuilder(UserService.class).web(true).run(args);
	}
}
	```
3.	添加application.yml，配置Eureka客户端配置
	```
	eureka:
	  instance:

		  preferIpAddress: true
	  client:
	      healthcheck:
	            enabled: true
	      serviceUrl:
	            defaultZone: http://localhost:8761/eureka/
	server:
	  port: 8882

	spring:
	  application:
	    name: userservice
	```
4.	启动客户端，可以改变端口，启动多个Client，然后浏览地址[http://localhost:8761](http://localhost:8761)查看已经注册的客户端。
5.	用相同的方式创建movieservice，Eureka客户端，实现代码如下：
controller层代码：
	```
@RestController
public class MovieController {
@Autowired
MovieService movieService;
@RequestMapping("/message")
public String getMessage()
{
	 return movieService.getuserMessage();
}
}
```
6.	service层代码

		```
	@Service
	public class MovieService {
	    @Autowired
	    RestTemplate restTemplate;
	    public String getuserMessage()
	    {
	        return restTemplate.getForObject("http://userservice/message",String.class);
	    }
	}
	```
7.	启动app程序代码
		```
		@EnableEurekaClient
		@SpringBootApplication
		public class MovieServiceApplication {

		@Bean
		@LoadBalanced
		public RestTemplate restTemplate() {
			return new RestTemplate();
		}
		public static void main(String[] args) {

			SpringApplication.run(MovieServiceApplication.class, args);
		}
	}
		```
8.	**Ribbon**配置：spring-could默认使用了`org.springframework.cloud.netflix.ribbon.RibbonClientConfiguration`作为配置类。同时也允许通过自定义类的方式修改负载均衡策略，下图为自定义类的实现。其中name，代表调用哪个服务时使用此配置，重写`IRule` 构造返回 `BestAvailableRule`，最好可用策略，具体的实现可以查看`BestAvailableRule`的源码，查看请求时选择服务的算法（负载均衡算法）。

				```
					@Configuration
					@RibbonClient(name = "microservice-provider-user", configuration = RibbonConfiguration.class)
					public class RibbonConfiguration {
					@Bean
					public IRule ribbonRule() {
					return new BestAvailableRule();					}
					}
					```

## 总结
> 通过对spring-cloud Eureka的学习，发现，spring-could可以通过很简单的配置来完成服务注册、发现以及简单地重写IRule来应用spring-could自带的负载均衡算法，可以看到它实现的负载均衡算法还是很简单的。以后也可以根据自身环境的状况，结合api提供的一些参数，书写自己的负载均衡算法。
> 服务注册于调用是微服务重要的组成部分，如果需要更深入的使用，还需要更加深入的学习与实践。

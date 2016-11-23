---
layout: post
title:  "spring-cloud之------config"
date:   2016-11-23
desc: "spring-cloud之------config"
keywords: "spring-boot spring-cloud spring-cloud-config"
categories: [microservice]
tags: [spring-cloud,spring-cloud-config]
icon: fa-bookmark-o
---
spring-cloud之------config
========
>简述
spring-cloud是一套开发分布式应用的工具集，包含了分布式应用需要的**配置管理、服务发现、服务熔断、智能路由、服务代理、控制总线**等。

#spring-cloud-config

##概述

spring-cloud-config是为其他微服务提供配置参数的服务，实现了配置的集中管理。如下图所示为应用配置信息集中管理的配置服务器：
![图挂了](http://images2015.cnblogs.com/blog/4758/201601/4758-20160114111514319-352101707.png)
##为什么使用它
引用官网的话**With the Config Server you have a central place to manage external properties for applications across all environments**，实现了对所有环境应用程序外部属性的集中管理。
##demo
- server实现：

1. 创建spring-boot的应用并添加对spring-cloud-config-server的引用
		```
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-config-server</artifactId>
			</dependency>
		```

2. 程序中加入注释@EnableConfigServer
		```
			@SpringBootApplication
			@EnableConfigServer
			public class DemoApplication {

			public static void main(String[] args) {
			SpringApplication.run(DemoApplication.class, args);
			}
		}
		```
3. 修改程序属性文件
		```
		server.port: 8887										  spring.cloud.config.server.git.uri: file://D:/work/jspro/config-repo

		```
4. 	新建uri属性配置文件，文件名的格式为{applicationname}- {profile}.properties，由程序名-配置名.properties组成，这些在后面客户端会用。
	```
	$ cd $HOME
	$ mkdir config-repo
	$ cd config-repo
	$ git init .
	$ echo fuck: true > config-repo-dev.properties
	$ git add -A .
	$ git commit -m "Add config-repo-dev.properties"
	```
5.	启动程序，并打开浏览器输入地址[http://localhost:8887/config-repo/dev](http://localhost:8887/config-repo/dev),如下面会显示我们设置的程序配置的信息，name是config-repo，prifiles是dev，里面包含`{"name":"file://D:/work/jspro/config-repo/config-repo-dev.properties","source":{"fuck":"true"}}`
	```
	{"name":"config-repo","profiles":["dev"],"label":"master","version":"3b75a227aac6661eaea024d0a512d6d548a9339b","state":null,"propertySources":[{"name":"file://D:/work/jspro/config-repo/config-repo-dev.properties","source":{"fuck":"true"}},{"name":"file://D:/work/jspro/config-repo/application.properties","source":{"info.foo":"bar"}}]}
	```

- client端实现：

1.	创建maven项目，添加pom引用。
		```
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		```
2.	设置controler，并通过`@Value("${fuck}")`获取属性。
	```
@SpringBootApplication
@RestController
public class SpringcloudclientApplication {
	@Value("${fuck}")
	String fuck;
	@RequestMapping("/")
	public String hello()
	{
		return "hello"+fuck;
	}
	public static void main(String[] args) {
		SpringApplication.run(SpringcloudclientApplication.class, args);
	}
}
	```
3.	在src/main/resource目录新建bootstrap.yml，添加configserver配置，**此处必须是bootstrap.yml不能是application.yml，因为加载配置服务实在启动阶段加载的，如果放在application会影响到配置加载。**
	```
	spring:
		cloud:
			config:
				uri: http://localhost:8887
			    name: config-repo
			    profile: dev

	---
    server:
	   port: 8886
	```
4.	启动客户端web应用，浏览地址[http://localhost:8886/](http://localhost:8886/)浏览器显示结果`hellotrue`

##总结
>通过spring-cloud中基础的spring-cloud-config的配置，感受到spring-boot集成开发的强大，对于开发者关系的分布式、微服务的一些模块，spring-boot都提供了这种开箱即用`out of the box`的实现方式。a good start


---
layout: post
title:  "spring-cloud֮------config"
date:   2016-11-23
desc: "spring-cloud֮------config"
keywords: "spring-boot spring-cloud spring-cloud-config"
categories: [microservice]
tags: [spring-cloud,spring-cloud-config]
icon: fa-bookmark-o
---
spring-cloud֮------config
========
> ����
spring-cloud��һ�׿����ֲ�ʽӦ�õĹ��߼��������˷ֲ�ʽӦ����Ҫ��**���ù��������֡������۶ϡ�����·�ɡ����������������**�ȡ�

# spring-cloud-config

## ����

spring-cloud-config��Ϊ����΢�����ṩ���ò����ķ���ʵ�������õļ��й�������ͼ��ʾΪӦ��������Ϣ���й�������÷�������

![ͼ����](http://images2015.cnblogs.com/blog/4758/201601/4758-20160114111514319-352101707.png)

## Ϊʲôʹ����

���ù����Ļ�**With the Config Server you have a central place to manage external properties for applications across all environments**��
ʵ���˶����л���Ӧ�ó����ⲿ���Եļ��й���

## demo

- serverʵ�֣�

1. ����spring-boot��Ӧ�ò���Ӷ�spring-cloud-config-server������

		```
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-config-server</artifactId>
			</dependency>
		```

2. �����м���ע��@EnableConfigServer

		```
			@SpringBootApplication
			@EnableConfigServer
			public class DemoApplication {

			public static void main(String[] args) {
			SpringApplication.run(DemoApplication.class, args);
			}
		}
		```

3. �޸ĳ��������ļ�

		```
		server.port: 8887
		spring.cloud.config.server.git.uri: file://D:/work/jspro/config-repo

		```

4. 	�½�uri���������ļ����ļ����ĸ�ʽΪ{applicationname}- {profile}.properties���ɳ�����-������.properties��ɣ���Щ�ں���ͻ��˻��á�

	```
	$ cd $HOME
	$ mkdir config-repo
	$ cd config-repo
	$ git init .
	$ echo fuck: true > config-repo-dev.properties
	$ git add -A .
	$ git commit -m "Add config-repo-dev.properties"
	```

5.	�������򣬲�������������ַ[http://localhost:8887/config-repo/dev](http://localhost:8887/config-repo/dev),
���������ʾ�������õĳ������õ���Ϣ��name��config-repo��prifiles��dev���������
`{"name":"file://D:/work/jspro/config-repo/config-repo-dev.properties","source":{"fuck":"true"}}`

	```
	{"name":"config-repo","profiles":["dev"],"label":"master","version":"3b75a227aac6661eaea024d0a512d6d548a9339b","state":null,"propertySources":[{"name":"file://D:/work/jspro/config-repo/config-repo-dev.properties","source":{"fuck":"true"}},{"name":"file://D:/work/jspro/config-repo/application.properties","source":{"info.foo":"bar"}}]}
	```

- client��ʵ�֣�

1.	����maven��Ŀ�����pom���á�

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

2.	����controler����ͨ��`@Value("${fuck}")`��ȡ���ԡ�

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

3.	��src/main/resourceĿ¼�½�bootstrap.yml�����configserver���ã�**�˴�������bootstrap.yml������application.yml��
��Ϊ�������÷���ʵ�������׶μ��صģ��������application��Ӱ�쵽���ü��ء�**

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

4.	�����ͻ���webӦ�ã������ַ[http://localhost:8886/](http://localhost:8886/)�������ʾ���
`hellotrue`

## �ܽ�

> ͨ��spring-cloud�л�����spring-cloud-config�����ã����ܵ�spring-boot���ɿ�����ǿ��
���ڿ����߹�ϵ�ķֲ�ʽ��΢�����һЩģ�飬spring-boot���ṩ�����ֿ��伴��`out of the box`��ʵ�ַ�ʽ��a good start


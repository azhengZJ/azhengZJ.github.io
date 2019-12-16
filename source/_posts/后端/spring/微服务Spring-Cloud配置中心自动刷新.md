---
title: 微服务Spring-Cloud配置中心自动刷新
date: 2019-12-15 12:00:00
author: azheng
img: /medias/banner/8.jpg
cover: false
categories: 后端
tags:
  - Java
  - jvm
---

只需要添加 bus（rabbitmq ）依赖和连接地址 会自动开启自动刷新
访问 http://{ 配置中心地址 }/actuator/bus-refresh 触发自动刷新

重点： 获取属性的属性类上必须要增加@RefreshScope 注解。
```java
@RefreshScope
public class EnvController {
    @Value("${girl.age}")
    private String age;

    @GetMapping("/env")
    public String getGirlInfo(){
        return age;
    }
}
```


## config项目
--------------------------------------------------------------------------------
启动类增加注解
```java
@EnableEurekaClient
@EnableConfigServer
```

application.yml配置
```yml
server:
  port: 8888
spring:
  application:
    name: CONFIG
  rabbitmq:
    host: 192.168.213.128
    port: 5672
    username: guest
    password: guest
  cloud:
    config:
      server:
        git:
          uri: http://git.baozun.com/mis/mis-config-repo.git
          username: hsh10732
          password: azheng@2018
          basedir: D:\Users\hsh10732\MIS\spring-cloud\config-repo
management:
  endpoints:
      web:
        exposure:
          include: "*" #暴露所有url，用于触发自动刷新
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8762/eureka/
```

添加的依赖
```groovy
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-config-server'
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    implementation 'org.springframework.cloud:spring-cloud-starter-bus-amqp'
}
```


## register项目
--------------------------------------------------------------------------------
启动类增加注解
@EnableEurekaServer


bootstrap.yml配置
```yml
spring:
  application:
    name: Eureka #项目名称(必填：会作为文件夹名称前缀查找配置文件)
  cloud:
    config:
      profile: dev #指定环境(必填：会作为文件夹名称后缀查找配置文件)
```

添加的依赖
```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springframework.cloud:spring-cloud-starter-config'
    implementation 'org.springframework.cloud:spring-cloud-starter-bus-amqp'
    runtime 'org.springframework.boot:spring-boot-devtools'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    compileOnly 'org.projectlombok:lombok'
}
```

对应到git库中的配置文件
Eureka-dev.yml

## client项目
--------------------------------------------------------------------------------
启动类增加注解
@EnableEurekaClient


bootstrap.yml配置
```yml
spring:
  application:
    name: BiDataApi
  cloud:
    config:
      profile: dev
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8762/eureka/
```

添加的依赖
```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    implementation 'org.springframework.cloud:spring-cloud-starter-config'
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    implementation 'org.springframework.cloud:spring-cloud-starter-bus-amqp'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    runtimeOnly 'org.springframework.boot:spring-boot-devtools'
    runtimeOnly 'mysql:mysql-connector-java'
    compileOnly 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

对应到git库中的配置文件
BiDataApi-dev.yml



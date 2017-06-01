---
layout: post
title:  "springboot—配置不同环境"
date:   2017-06-02 11:03:01 +0800
categories: spring springboot
tag: spring springboot
sid: 1496337989
---


1. 新建一个maven项目,pom.xml文件如下
    ~~~xml
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>

        <groupId>com.vcg.community</groupId>
        <artifactId>spring-boot-sample</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <packaging>jar</packaging>

        <name>spring-boot-sample</name>
        <description>Demo project for Spring Boot</description>

        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>1.5.3.RELEASE</version>
            <relativePath/> <!-- lookup parent from repository -->
        </parent>

        <properties>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
            <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
            <java.version>1.8</java.version>
        </properties>

        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
                <version>1.5.3.RELEASE</version>
            </dependency>
        </dependencies>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </build>
    </project>
    ~~~

2. 在src/main/resources下新建四个.properties文件
    - application.properties
    - application-test.properties
    - application-dev.properties
    - application-prod.properties

**application.properties**
~~~java
spring.profiles.active=dev或test或prod
~~~

**application-test.properties**
~~~java
server.port=8091
~~~

**application-dev.properties**
~~~java
server.port=8092
~~~

**application-prod.properties**
~~~java
server.port=8093
~~~

3. 新建SpringBootSampleApplication.java
~~~java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableAutoConfiguration
public class SpringBootSampleApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringBootSampleApplication.class, args);
	}

}
~~~

4. 运行,分别在设置application.properties中的内容为spring.profiles.active=dev或test或prod,查看启动日志,可以顺利看到在不同的端口启动。
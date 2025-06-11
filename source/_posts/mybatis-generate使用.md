---
title: mybatis-generate使用
date: 2024-12-02 09:17:13
categories:
 - java
 - springboot
tags: 
 - springboot
---

#### mybatis generator
###### 每次配置mybatis都很痛苦，还容易出错，推荐使用mybatis generator
1. 在maven里面添加插件依赖
```
<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
          <plugin>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-maven-plugin</artifactId>
            <version>1.4.1</version>
            <configuration>
              <configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
              <verbose>true</verbose>
              <overwrite>true</overwrite>
            </configuration>
            <dependencies>
              <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>8.0.28</version>
              </dependency>
            </dependencies>
          </plugin>
        </plugins>
      <resources>
        <resource>
          <directory>src/main/resources</directory>
        </resource>
        <resource>
          <directory>src/main/java</directory>
          <includes>
            <include>**/*.xml</include>
          </includes>
        </resource>
      </resources>
```
2. 编写generatorConfig.xml文件，放在指定目录下src/main/resources/generatorConfig.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
    PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
    "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <context id="MySQLTables" targetRuntime="MyBatis3">
        <!-- 数据库连接配置 -->
        <jdbcConnection
            driverClass="com.mysql.cj.jdbc.Driver"
            connectionURL="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=Asia/Shanghai"
            userId="root"
            password="">
        </jdbcConnection>
     <!-- Java类型解析器 -->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>
        <!-- 实体类生成配置 -->
        <javaModelGenerator targetPackage="org.apache.dubbo.springboot.demo.provider.entity"
            targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>
        <!-- Mapper XML文件生成配置 -->
        <sqlMapGenerator targetPackage="mapper"
            targetProject="src/main/java/org/apache/dubbo/springboot/demo/provider">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>
        <!-- Mapper接口生成配置 -->
        <javaClientGenerator type="XMLMAPPER"
            targetPackage="org.apache.dubbo.springboot.demo.provider.mapper"
            targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>
        <!-- 要生成的表配置 -->
        <table tableName="Student" domainObjectName="Student"
            enableCountByExample="false" enableUpdateByExample="false"
            enableDeleteByExample="false" enableSelectByExample="false"
            selectByExampleQueryId="false">
            <property name="useActualColumnNames" value="false"/>
        </table>
    </context>
</generatorConfiguration>

3.  注意 ：我们生成的xml文件放在src/main/java/org/apache/dubbo/springboot/demo/provider下面，所以需要配置resources目录下的文件为资源目录
```
       <resources>
        <resource>
          <directory>src/main/resources</directory>
        </resource>
        <resource>
          <directory>src/main/java</directory>
          <includes>
            <include>**/*.xml</include>
          </includes>
        </resource>
      </resources>
```
4.  在启动类加上@MapperScan注解，指定src/main/java/org/apache/dubbo/springboot/demo/provider
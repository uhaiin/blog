---
layout: mypost
title: SpringBoot集成通用mapper和lombok
categories: [tk.mybatis,lombok,java]
---

**通用 Mapper**（即 `tk.mybatis`）是一个基于 MyBatis 的**通用 CRUD 框架**，它通过泛型和注解的方式，为实体类自动生成常用的增删改查（CRUD）SQL 操作，**无需编写 XML 或 SQL**，大幅提升开发效率。

它不是 MyBatis 的替代品，而是**对 MyBatis 的增强工具**，让开发者专注于复杂业务 SQL，而不用重复写基础操作。

### 核心优势

| 优点                 | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| ✅ **减少重复代码**   | 不用手写 `insert`、`update`、`delete`、`selectById` 等基础方法 |
| ✅ **使用简单**       | 只需继承通用接口，即可拥有通用方法                           |
| ✅ **支持注解配置**   | 实体类通过注解映射数据库字段，无需 XML                       |
| ✅ **兼容 MyBatis**   | 完全兼容原生 MyBatis，可与自定义 SQL 混用                    |
| ✅ **支持多种数据库** | MySQL、Oracle、SQL Server、PostgreSQL 等主流数据库           |
| ✅ **高度可扩展**     | 可自定义通用方法，支持插件机制                               |

### 修改 Mybatis Generator

tk.mybatis 默认的生成器不带 lombok，需要修改本地仓库的以下jar包

```xml
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.4.2</version>
</dependency>
```

编译以下代码为`LombokPlugin.class`文件，然后替换`mybatis-generator-core`相应版本下`org.mybatis.generator.plugins`包下的`LombokPlugin.class`文件。

> 存在则替换，不存在则新增

```java
package org.mybatis.generator.plugins;

import java.util.List;

import org.mybatis.generator.api.IntrospectedColumn;
import org.mybatis.generator.api.IntrospectedTable;
import org.mybatis.generator.api.Plugin;
import org.mybatis.generator.api.PluginAdapter;
import org.mybatis.generator.api.dom.java.Method;
import org.mybatis.generator.api.dom.java.TopLevelClass;

public class LombokPlugin extends PluginAdapter {

	@Override
	public boolean validate(List<String> warnings) {
		return true;
	}

	public boolean modelBaseRecordClassGenerated(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {
		topLevelClass.addImportedType("lombok.Data");
		topLevelClass.addImportedType("lombok.Builder");
		topLevelClass.addAnnotation("@Data");
		topLevelClass.addAnnotation("@Builder");
		return true;
	}

	public boolean modelSetterMethodGenerated(Method method, TopLevelClass topLevelClass,
			IntrospectedColumn introspectedColumn, IntrospectedTable introspectedTable,
			Plugin.ModelClassType modelClassType) {
		return false;
	}

	public boolean modelGetterMethodGenerated(Method method, TopLevelClass topLevelClass,
			IntrospectedColumn introspectedColumn, IntrospectedTable introspectedTable,
			Plugin.ModelClassType modelClassType) {
		return false;
	}

}
```

这样魔改后就支持生成的代码使用lombok插件

### 完整POM文件

> postgresql 版本，需要其他的数据库版本请自行替换依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.uhaiin</groupId>
	<artifactId>mybatis-generator</artifactId>
	<version>1.0.0</version>

	<properties>
		<maven.compiler.source>21</maven.compiler.source>
		<maven.compiler.target>21</maven.compiler.target>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>

	<dependencies>
		<!--Mybatis 通用mapper tk单独使用，自己独有+自带版本号-->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>3.5.13</version>
		</dependency>
		<!-- Mybatis Generator 自己独有+自带版本号-->
		<dependency>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-core</artifactId>
			<version>1.4.2</version>
		</dependency>
		<!--通用Mapper-->
		<dependency>
			<groupId>tk.mybatis</groupId>
			<artifactId>mapper</artifactId>
			<version>4.2.3</version>
		</dependency>
		<!--postgresql -->
		<dependency>
			<groupId>org.postgresql</groupId>
			<artifactId>postgresql</artifactId>
			<version>42.7.5</version>
		</dependency>
		<!--persistence-->
		<dependency>
			<groupId>javax.persistence</groupId>
			<artifactId>persistence-api</artifactId>
			<version>1.0.2</version>
		</dependency>
		<!--lombok-->
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<version>1.18.36</version>
			<optional>true</optional>
		</dependency>
	</dependencies>

	<build>
		<resources>
			<resource>
				<directory>${basedir}/src/main/java</directory>
				<includes>
					<include>**/*.xml</include>
				</includes>
			</resource>
			<resource>
				<directory>${basedir}/src/main/resources</directory>
			</resource>
		</resources>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<version>3.4.1</version>
				<configuration>
					<excludes>
						<exclude>
							<groupId>org.projectlombok</groupId>
							<artifactId>lombok</artifactId>
						</exclude>
					</excludes>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.mybatis.generator</groupId>
				<artifactId>mybatis-generator-maven-plugin</artifactId>
				<version>1.4.2</version>
				<configuration>
					<configurationFile>
						${basedir}/src/main/resources/generatorConfig.xml</configurationFile>
					<overwrite>true</overwrite>
					<verbose>true</verbose>
				</configuration>
				<dependencies>
					<!--postgresql -->
					<dependency>
						<groupId>org.postgresql</groupId>
						<artifactId>postgresql</artifactId>
						<version>42.2.28</version>
					</dependency>
					<dependency>
						<groupId>tk.mybatis</groupId>
						<artifactId>mapper</artifactId>
						<version>4.2.3</version>
					</dependency>
				</dependencies>
			</plugin>
		</plugins>
	</build>

</project>
```

### 配置文件

#### 数据库配置

在 resources目录下新增：db.properties

```properties
package.name=com.uhaiin
jdbc.driverClass=org.postgresql.Driver
jdbc.url=jdbc:postgresql://ip:port/db?currentSchema=public&encoding=UTF-8
jdbc.username=username
jdbc.password=password
```

#### 生成配置

在 resources目录下新增：generatorConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
	<properties resource="db.properties" />

	<context id="postgresql" targetRuntime="MyBatis3Simple"
		defaultModelType="flat">
		<property name="beginningDelimiter" value="`" />
		<property name="endingDelimiter" value="`" />

		<plugin type="tk.mybatis.mapper.generator.MapperPlugin">
			<property name="mappers"
				value="tk.mybatis.mapper.common.Mapper" />
			<property name="caseSensitive" value="true" />
		</plugin>

		<plugin type="org.mybatis.generator.plugins.LombokPlugin">
			<property name="hasLombok" value="true" />
		</plugin>

		<commentGenerator>
			<property name="suppressDate" value="true" />
			<!-- 是否去除自动生成的注释 true：是 ： false:否 -->
			<property name="suppressAllComments" value="true" />
		</commentGenerator>

		<jdbcConnection driverClass="${jdbc.driverClass}"
			connectionURL="${jdbc.url}" userId="${jdbc.username}"
			password="${jdbc.password}">
		</jdbcConnection>

		<javaModelGenerator
			targetPackage="${package.name}.entity" targetProject="src/main/java" />

		<sqlMapGenerator
			targetPackage="${package.name}.mapper" targetProject="src/main/java" />

		<javaClientGenerator
			targetPackage="${package.name}.mapper" targetProject="src/main/java"
			type="XMLMAPPER" />

		<table tableName="sys_user" domainObjectName="User">
            <!-- 非主键自增的可以不写 -->
			<generatedKey column="user_id" sqlStatement="JDBC" />
		</table>
	</context>
</generatorConfiguration>
```

### 生成代码

#### eclipse

配置eclipse运行插件配置

![idea.png](/assets/img/blog/20260203/eclipse-1.png)

新增以下配置：mybatis-generator:generate

![idea.png](/assets/img/blog/20260203/eclipse-2.png)

以后需要运行时候只需要修改完`generatorConfig.xml`中的

```xml
<table tableName="sys_user" domainObjectName="User">
    <!-- 非主键自增的可以不写 -->
    <generatedKey column="user_id" sqlStatement="JDBC" />
</table>
```

即可运行eclipse插件配置

#### idea

双击执行插件

![idea.png](/assets/img/blog/20260203/idea.png)

### 使用方法

把生成的**实体类**、**Mapper.java**、**Mapper.xml**拷贝到项目中即可使用，注意项目中的pom文件需要引入以下依赖：

```xml
<!--预编译工具-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<!-- PostgreSQL 驱动 -->
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
<!--通用Mapper4之tk.mybatis-->
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper</artifactId>
    <version>4.3.0</version>
</dependency>
<!-- MyBatis Spring Boot Starter -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>4.0.1</version>
</dependency>
```

在springboot项目的启动类上添加扫描注解，注意使用tk.mybatis

```java
@MapperScan(basePackages = "com.example.mapper")
@SpringBootApplication
public class Application { }
```

### 与 MyBatis-Plus 对比

| 特性      | tk.mybatis        | MyBatis-Plus             |
| --------- | ----------------- | ------------------------ |
| 通用 CRUD | ✅ 支持            | ✅ 支持，更强大           |
| 主键策略  | 需手动配置        | 内置多种策略（@TableId） |
| 分页支持  | 需配合 PageHelper | 内置分页插件             |
| 代码生成  | 需额外工具        | 提供代码生成器           |
| 活跃度    | 已基本停止更新    | 活跃维护，生态丰富       |
| 学习成本  | 低，简单直接      | 稍高，功能多             |

> 💡 建议：新项目可优先考虑 **MyBatis-Plus**，但老项目或轻量级场景仍可使用

### 总结

**tk.mybatis 通用 Mapper 适合：**

- 单表操作多的项目
- 想快速上手、减少模板代码
- 不想引入复杂框架的轻量级项目

**一句话总结：**

> 用一个接口继承，换 80% 的 CRUD 不用手写。


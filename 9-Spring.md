# Spring

[TOC]

## 1 Spring概述

### 1.1 什么是Spring？

Spring 是一个轻量级 Java 开发框架，目的是为了解决企业级应用开发的业务逻辑层和其他各层的耦合问题。它是以恶搞分层的 JavaSE/JavaEE full-stack（一站式）轻量级开源框架，为开发 Java 应用程序提供全面的基础架构支持。Spring 复杂基础架构，因此 Java 开发者可以专注于应用程序的开发。

Spring 的根本作用是解决企业级应用开发的复杂性，即简化 Java 开发。

Spring 的功能底层都依赖于它的两个重要特性：**依赖注入（dependency injection， DI）**和**面向切面编程（aspect-oriented-programming，AOP）**。

#### Spring 降低 Java 开发复杂性采取的四种关键策略：

- 基于 POJO 的轻量级和最小侵入性编程；
- 通过依赖注入和面向接口实现松耦合；
- 基于切面和管理进行声明式编程；
- 通过切面和模板减少样板式代码。

### 1.2 Spring 框架的设计目标、设计理念及核心是什么？

#### Spring设计目标

Spring 为开发者提供一个一站式轻量级应用开发平台。

#### Spring设计理念

在 JavaEE 开发中，支持 POJO 和 JavaBean 开发方式，使应用面向接口编程，充分支持面向对象设计方法；Spring 通过IOC容器实现对象耦合关系的管理，并实现依赖反转，将对象之间的依赖关系交给 IOC 容器，实现解耦。

#### Spring框架的核心

**IOC容器**和**AOP模块**。

通过IOC容器管理POJO对象以及他们之间的耦合关系；通过AOP动态非侵入的方式增强服务。

IOC让相互写作的组件保持松散的耦合，而AOP编程允许把遍布于应用各层的功能分离出来形成可重用的功能组件。

### 1.3 使用Spring的好处是什么？

- **轻量**：Spring 是轻量的，最基本的版本大约 2MB；
- **控制反转**：Spring 通过控制反转实现了松散耦合，对象给出它们的依赖，而不是创建或查找依赖的对象们；
- **面向切面的编程（AOP）**：Spring支持面向切面的编程，并把应用业务逻辑和系统服务分开；
- **容器**：Spring包含管理应用中对象的生命周期和配置；
- **MVC框架**：Spring的web框架是个精心设计的框架，是web框架的一个很好的替代品；
- **事务管理**：Spring提供的一个持续的事务管理接口，可以扩展上至本地事务，下至全局事务（JTA）；
- **异常处理**：Spring提供方便的API把具体技术相关的异常转换为一致的 unchecked 异常。

### 1.4 Spring由哪些模块组成

![在这里插入图片描述](https://img-blog.csdnimg.cn/c4f39275dba3460ca77cee25ff6ce486.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBASmF2Yeeoi-W6j-mxvA==,size_13,color_FFFFFF,t_70,g_se,x_16)

- 核心容器（Core Container）
- 
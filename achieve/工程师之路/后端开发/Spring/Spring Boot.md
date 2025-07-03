---
tags:
  - spring
---
# Spring MVC 处理流程

# 控制反转 IoC (Inversion of Control)

- 依赖注入（Dependency Injection，DI）
- 依赖查询

## Singleton Bean 循环依赖

通过三级缓存来解决：
- **一级缓存（singletonObjects）**，存放最终形态的 Bean（已经实例化、属性填充、初始化）
- **二级缓存（earlySingletonObjects）**，存放过渡 Bean（半成品，尚未属性填充以及初始化）
- **三级缓存（singletonFactories）**，存放`ObjectFactory`

# 面向切面编程 AOP (Aspect-Oriented Programming)


# Spring 事务

- **编程式事务**，在代码中硬编码(在分布式系统中推荐使用)，通过 `TransactionTemplate`或者 `TransactionManager` 手动管理事务。
- **声明式事务**，直接基于注解 `@Transactional`，通过 AOP 实现

主要使用的事务传播行为：
- **`TransactionDefinition.PROPAGATION_REQUIRED`**， 默认值
- **`TransactionDefinition.PROPAGATION_REQUIRES_NEW`**
- **`TransactionDefinition.PROPAGATION_NESTED`**
- **`TransactionDefinition.PROPAGATION_MANDATORY`**

核心接口：
- **`PlatformTransactionManager`**，事务管理器，Spring 事务策略的核心
- **`TransactionDefinition`**，事务定义信息(事务隔离级别、传播行为、超时、只读、回滚规则)
- TransactionStatus

# 自动装配

- AutoConfigurationImportSelector

# 参考

- https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html
- 
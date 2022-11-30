# Spring源码学习

环境配置：

- JDK1.8
- Spring-Framework5.2.12.RELEASE

> [源码地址](https://gitcode.net/mirrors/spring-projects/spring-framework/-/tree/v5.2.12.RELEASE)

配置镜像：

```shell
# 在build.gradle中加入阿里云镜像
repositories {
			maven { url 'https://maven.aliyun.com/nexus/content/groups/public/' }
			maven { url 'https://maven.aliyun.com/nexus/content/repositories/jcenter'}
			mavenCentral()
			maven { url "https://repo.spring.io/libs-spring-framework-build" }
}

# 删除plugin
// id 'io.spring.gradle-enterprise-conventions' version '0.0.2'
```

预编译：

```shell
# 在源码根目录下执行命令
./gradlew :spring-oxm:compileTestJava
```

导入IntelliJ：

File -> New -> Project from Existing Sources -> Navigate to directory -> Select build.gradle

## Spring工作原理

## 1. Bean的创建流程(不考虑循环依赖)

```java
class UserService {
  
}
```

UserService类 --> 推断构造方法-->普通对象-->依赖注入-->初始化前(@PostConstruct)-->初始化(InitializingBean)-->初始化后(AOP)-->代理对象-->放入singletonFactories-->放入singletonObjects单例池

- 构造推断

    如果没有声明构造函数，Spring使用默认的无参构造。

    如果同时声明了无参构造和有参构造，Spring优先使用无参构造。

    如果想要指定某个构造函数，在构造方法上加上@Autowired。

    ```java
    class UserService {
      // 1.使用默认无参构造
    }
    
    class UserService {
      // 2.使用无参构造
      UserService() {
        ...
      }
      UserService(OrderService orderService) {
        ...
      }
    }
    
    class UserService {
      // 3.NoSuchMethodException: UserService.<init>()
      UserService(OrderService orderService) {
        ...
      }
      UserService(OrderService orderService1, OrderService orderService2) {
        ...
      }
    }
    ```

- 依赖注入(先byType再byName)

    当使用下面构造器进行依赖注入时，spring先去单例池（单例池是一个Map<beanName, Bean对象>）中找OrderService类，如果没有，则spring会去创建orderService的bean。如果找到多个OrderService类，再根据beanName去找。

    ```java
    class UserService {
    	private OrderService orderService;
      
      @Autowired
      public UserService (OrderService orderService) {
        this.orderService = orderService;
        ...
      }
    }
    ```

- AOP

    spring的aop使用cglib，cglib是基于父子类的动态代理：

    ```java
    class UserServiceProxy extends UserService {
      // target是普通对象，spring会在依赖注入之后把普通对象赋值给target
    	UserService target;
      
      public void test () {
        // 执行切面逻辑
        ...
        // 执行普通对象的test方法
        target.test();
        // super.test(); // 使用代理对象执行test方法
      }
    }
    
    class UserService {
      ...
      @Transactional
      public void test () {
        ...
      }
    }
    ```

    > Target.test()是通过普通对象去调用方法，而不是代理对象。

- 事务

    ```java
    class UserServiceProxy extends UserService {
      UserService target;
      
      public void test () {
        // 1. 判断是否有@Transactional注解
        // 2. 事务管理器新建一个数据库连接conn
        // 3. conn.autocommit = false
        // 4. target.test
        // 5. conn.commit() / conn.rollback()
      }
    }
    
    class UserService {
      ...
      @Transactional
      public void test () {
        insert();
        // 此时test2()的@Transactional注解失效，因为在代理类中是普通对象调用test2()，并没有经过aop
        test2();
      }
      
      @Transactional
      public void test2() {
        insert();
      }
    }
    ```
    
    

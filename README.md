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


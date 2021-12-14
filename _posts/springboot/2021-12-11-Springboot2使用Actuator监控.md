---

layout: post
title:  "Springboot2使用Actuator监控"
date:   2021-12-11 20:14:54
categories: SpringBoot
tags: SpringBoot
excerpt: Springboot2使用Actuator监控
mathjax: true

---

* content
{:toc}


SpringBoot自带监控功能Actuator，可以帮助实现对程序内部运行情况监控，比如监控状况、Bean加载情况、环境变量、日志信息、线程信息等

## 配置Actuator

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

启动应用程序后访问 http://localhost:8080/actuator/ 可以看到所有的访问链接。

以下是所有的访问链接

如果要设置启用所有的，或者可以启用部分["health","info"]

```
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

| HTTP方法 | 路径                     | 描述                                                         | 是否敏感信息 |
| -------- | ------------------------ | ------------------------------------------------------------ | ------------ |
| GET      | /actuator/auditevents    | 显示当前审计信息                                             | true         |
| GET      | /actuator/configprops    | 查看配置属性，包括默认配置, 显示一个所有@ConfigurationProperties的整理列表 | true         |
| GET      | /actuator/beans          | bean及其关系列表, 显示一个应用中所有Spring Beans的完整列表   | true         |
| GET      | /actuator/heapdump       | 堆信息                                                       | true         |
| GET      | /actuator/env            | 查看所有环境变量                                             | true         |
| GET      | /actuator/env/{name}     | 查看具体变量值                                               | true         |
| GET      | /actuator/health         | 查看应用健康指标, 当使用一个未认证连接访问时显示一个简单的’status’，使用认证连接访问则显示全部信息详情 | false        |
| GET      | /actuator/info           | 查看应用信息                                                 | false        |
| GET      | /actuator/mappings       | 查看所有url映射, 即所有@RequestMapping路径的整理列表         | true         |
| GET      | /actuator/metrics        | 查看应用基本指标                                             | true         |
| GET      | actuator/metrics/{name}  | 查看具体指标                                                 | true         |
| POST     | /actuator/shutdown       | 关闭应用，允许应用以优雅的方式关闭（默认情况下不启用）       | true         |
| GET      | /actuator/httptrace      | 查看基本追踪信息，默认为最新的一些HTTP请求                   | true         |
| GET      | /actuator/scheduledtasks | 定时任务信息                                                 | true         |
| GET      | /actuator/threaddump     | 执行一个线程dump                                             |              |

默认启用的连接，Spring Boot 2.X 中，Actuator 默认只开放 health 和 info 两个端点。

```
/actuator
/actuator/health
/actuator/health/{component}
/actuator/health/{component}/{instance}
/actuator/info
```



## health健康监控

/actuator/health

```
{"status":"UP"}
```

只返回了status，是因为未启用其他数据支持。

### 1.通过配置management.endpoint.health.show-details属性

| **名称**        | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| never           | 详细信息从不显示。                                           |
| when-authorized | 详细信息仅显示给授权用户，可以使用management.endpoint.health.roles配置授权角色。 |
| always          | 详细信息显示给所有用户。                                     |

默认为never，因此可以配置为always

```
management:
  endpoint:
    health:
      show-details: always
```

然后再访问，就发现包含很多健康监测的信息了，需要注意的是，其中某个监测失败，就会导致整个状态失败

有以下监测指标

| **名称**                     | **描述**                             |
| ---------------------------- | ------------------------------------ |
| CassandraHealthIndicator     | 检查Cassandra数据库是否已启动。      |
| CouchbaseHealthIndicator     | 检查Couchbase群集是否启动。          |
| DiskSpaceHealthIndicator     | 检查磁盘空间是否不足。               |
| DataSourceHealthIndicator    | 检查是否可以获取到DataSource的连接。 |
| ElasticsearchHealthIndicator | 检查Elasticsearch群集是否启动。      |
| InfluxDbHealthIndicator      | 检查InfluxDB服务器是否启动。         |
| JmsHealthIndicator           | 检查JMS代理是否启动。                |
| MailHealthIndicator          | 检查邮件服务器是否启动。             |
| MongoHealthIndicator         | 检查Mongo数据库是否已启动。          |
| Neo4jHealthIndicator         | 检查Neo4j服务器是否启动。            |
| RabbitHealthIndicator        | 检查Rabbit服务器是否启动。           |
| RedisHealthIndicator         | 检查Redis服务器是否启动。            |
| SolrHealthIndicator          | 检查Solr服务器是否启动。             |

### 2.启用禁用指标

```
management:
  endpoint:
    health:
      show-details: always
  health:
    defaults:
      enabled: false
    db:
      enabled: true
```

### 3.自定义healrh指标

自定义是通过实现HealthIndicator接口实现health方法，返回响应。

或者继承AbstractHealthIndicator的，实现doHealthCheck方法

实现类为XXXHealthIndicator，这样指标就是XXX

## Info信息

/actuator/info

测试比如

```
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;
 
@Component
public class MyHealthIndicator implements HealthIndicator {
 
    private static int num = 0;
 
    @Override
    public Health health() {
        // 进行一些特定的健康检查
        int errorCode = check();
        if (errorCode != 0) {
            return Health.down().withDetail("Error Code", errorCode).build();
        }
        return Health.up().build();
    }
 
    // 这里模拟检查，设置为一次正常一次异常
    private int check() {
        num++;
        return num % 2;
    }
}
```

返回

```
{
    "status": "DOWN",
    "components": {
        "my": {
            "status": "DOWN",
            "details": {
                "Error Code": 1
            }
        }
    }
}


{
    "status": "UP",
    "components": {
        "my": {
            "status": "UP"
        }
    }
}
```

关于启用禁用参考上面。

### 1.配置文件方式

```
info:
  appName: boot-admin
  version: 2.0.1
  mavenProjectName: @project.artifactId@  #使用@@可以获取maven的pom文件值
  mavenProjectVersion: @project.version@
```

### 2.编写InfoContributor

```
import java.util.Collections;

import org.springframework.boot.actuate.info.Info;
import org.springframework.boot.actuate.info.InfoContributor;
import org.springframework.stereotype.Component;

@Component
public class ExampleInfoContributor implements InfoContributor {

    @Override
    public void contribute(Info.Builder builder) {
        builder.withDetail("example",
                Collections.singletonMap("key", "value"));
    }
}
```

以下俩种都可以输出info信息

## 启用 shutdown

 shutdown 默认关闭 ,添加一下配置 开启shutdown,且只支持post请求

```
management:
  endpoint:
    shutdown:       
      enabled: true
```

## ENV环境变量

/actuator/env

关注

## metrics指标监控

/actuator/metrics

```
{
	"names": ["jvm.memory.max", "jvm.threads.states", "jvm.gc.memory.promoted", "jvm.memory.used", "jvm.gc.max.data.size", "jvm.gc.pause", "jvm.memory.committed", "system.cpu.count", "logback.events", "http.server.requests", "jvm.buffer.memory.used", "tomcat.sessions.created", "jvm.threads.daemon", "system.cpu.usage", "jvm.gc.memory.allocated", "tomcat.sessions.expired", "jvm.threads.live", "jvm.threads.peak", "process.uptime", "tomcat.sessions.rejected", "process.cpu.usage", "jvm.classes.loaded", "jvm.classes.unloaded", "tomcat.sessions.active.current", "tomcat.sessions.alive.max", "jvm.gc.live.data.size", "jvm.buffer.count", "jvm.buffer.total.capacity", "tomcat.sessions.active.max", "process.start.time"]
}
```

以下指标

- JVM最大内存（jvm.memory.max），单位:KB
- JVM已使用内存（jvm.memory.used），单位:KB
- CPU使用率（system.cpu.usage）
- 系统正常运行时间（process.uptime），单位:秒
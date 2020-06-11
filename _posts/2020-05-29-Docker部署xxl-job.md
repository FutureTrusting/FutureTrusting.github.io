---
layout:     post
title:      Docker部署xxl-job
subtitle:   Docker部署xxl-job
date:       2020-05-29
author:     Square
header-img: img/wallhaven-zm3mxw.jpg
catalog: true
tags:
    - Linux
---


##### 拉取xxl-job 镜像
docker pull xuxueli/xxl-job-admin:2.0.2
##### 下载application.properties
修改数据库配置，以及端口号等配置
```
wget https://raw.githubusercontent.com/xuxueli/xxl-job/2.0.2/xxl-job-admin/src/main/resources/application.properties
```
##### 启动镜像容器
```
docker run -d --name xxl-job-admin -v 修改后的配置文件位置/application.properties  --net host -e PARAMS='--spring.config.location=/application.properties' xuxueli/xxl-job-admin:2.0.2
```
##### 也可以直接添加参数启动镜像容器
注意一些参数如邮箱可以省略
```
docker run -e PARAMS="--spring.datasource.url=jdbc:mysql://数据库地址:3306/xxl-job?Unicode=true&characterEncoding=UTF-8 --spring.datasource.password=数据库密码 --spring.mail.host=smtp.163.com --spring.mail.username=邮箱名 --spring.mail.password=邮箱密码 --xxl.job.login.password=登录密码" -p 18092:8080 -v /tmp:/data/applogs --name xxl-job-admin --privileged=true -d xuxueli/xxl-job-admin:2.0.2
```

##### 1.添加依赖
文档介绍：http://www.xuxueli.com/xxl-job/#/
```
<dependency>
 <groupId>com.xuxueli</groupId>
 <artifactId>xxl-job-core</artifactId>
 <version>2.0.2</version>
</dependency>
```
##### 2.增加配置
```
xxl:
  job:
    admin:
      addresses: http://10.158.1.158:19031/xxl-job-admin
    executor:
      appname: square-executor
      ip:
      port: ${job-port:31400}
      logpath: /app/data/logs/
      logretentiondays: -1
    accessToken:
```
##### 3.java配置类
XxlJobConfig.java
```java
@Data
@Component
@ConfigurationProperties(prefix = "xxl.job")
public class XxlJobProperties {
    private JobAdmin admin;
    private String accessToken;
    private JobExecutor executor;

    @Data
    public static class JobAdmin {
        private String addresses;
    }

    @Data
    public static class JobExecutor {
        private String appName;
        private String ip;
        private int port;
        private String logPath;
        private int logRetentionDays;
    }
}
```
```java
/**
 * xxl-job config
 *
 * @author xuxueli 2017-04-28
 */
@Configuration
@Slf4j
@ComponentScan(basePackages = "com.square.cape.jobhandler")
public class XxlJobConfig {

    @Autowired
    XxlJobProperties xxlJobProperties;

    @Bean(initMethod = "start", destroyMethod = "destroy")
    public XxlJobSpringExecutor xxlJobExecutor() {
        log.info(">>>>>>>>>>> xxl-job config init.");
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(xxlJobProperties.getAdmin().getAddresses());
        xxlJobSpringExecutor.setAppName(xxlJobProperties.getExecutor().getAppName());
        xxlJobSpringExecutor.setIp(xxlJobProperties.getExecutor().getIp());
        xxlJobSpringExecutor.setPort(xxlJobProperties.getExecutor().getPort());
        xxlJobSpringExecutor.setAccessToken(xxlJobProperties.getAccessToken());
        xxlJobSpringExecutor.setLogPath(xxlJobProperties.getExecutor().getLogPath());
        xxlJobSpringExecutor.setLogRetentionDays(xxlJobProperties.getExecutor().getLogRetentionDays());

        return xxlJobSpringExecutor;
    }

}
```

#### 4.task示例
```java
@Slf4j
@JobHandler(value = "xxlQueryJobHandler")
@Service
public class XxlQueryJobHandler extends IJobHandler {

    @Override
    public ReturnT<String> execute(String param) throws Exception {
        log.info("xxlQueryJobHandler start.");
        System.err.println("xxlQueryJobHandler" + param);
        log.info("xxlQueryJobHandler end.");
        return SUCCESS;
    }
}
```
##### 5.启动项目
项目启动参数添加job-port：-Djob-port=30000


---
layout:     post
title:      springboot分布式应用session共享
# subtitle:   java web 请求拦截的方法
date:       2020-06-20
author:     BY wyl53
# header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - springboot
    - session
    - redis
---
# 前言
现在越来越多的应用采用分布式的架构，相同的服务可能会部署多个，这样session共享就十分有必要了。大多数共享session是使用redis存储的，本文提供一种实现方法。

# springboot应用
添加maven依赖，注意spring-session-data-redis依赖。
```
 <parent>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-parent</artifactId>
     <version>2.2.5.RELEASE</version>
 </parent>

 <dependencies>
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-web</artifactId>
     </dependency>

     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-data-redis</artifactId>
     </dependency>

     <dependency>
         <groupId>org.springframework.session</groupId>
         <artifactId>spring-session-data-redis</artifactId>
     </dependency>
 </dependencies>
 <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

编写controller，实现两个端点，一个设置session的key/value，一个读取session的key/value。读取session的key/value时，响应内容带上应用的监听端口，就知道那个进程提供的返回内容。
```
@RestController
@RequestMapping("/")
public class HomeController implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext _applicationContext) throws BeansException {
        this.applicationContext = _applicationContext;
    }

    @GetMapping("getSession")
    public String getSession(HttpSession session){
        String value = (String) session.getAttribute("param");
        return applicationContext.getEnvironment().getProperty("spring.port") + " get session value = " + value;
    }

    @GetMapping("setSession")
    public String setSession(HttpSession session, @RequestParam("value") String value){
        session.setAttribute("param",value);
        return applicationContext.getEnvironment().getProperty("spring.port");
    }
}
```

application.yml 配置
```
server:
  port: 8080
spring:
  redis:
    host: 192.168.0.125
    database: 1
```

# Nginx集群配置
添加 upstream 配置，填写集群的地址。
```
    upstream backend {
		server localhost:8080   weight=1;
		server localhost:8081   weight=1;
	}
```

配置代理转发proxy_pass
```
    location / {
	    proxy_pass http://backend;
    }
```

打包项目，然后启动两个进程，分布监听8080和8081端口。
```
    java -Dserver.port=8080 -jar a.jar
    java -Dserver.port=8081 -jar a.jar
```

打开浏览器输入： ```http://localhost/setSession?value=testValue```

然后再访问： ```http://localhost/getSession```

多访问几次就会看到，监听8080端口和8081端口的进程轮流提供返回内容，获取到session的值是一样的，证明共享session成功。

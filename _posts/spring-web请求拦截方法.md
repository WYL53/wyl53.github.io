---
layout:     post
title:      java web开发请求拦截的方法
# subtitle:   java web 请求拦截的方法
date:       2018-07-06
author:     BY wyl53
# header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - java
    - spring
    - 过滤器（filter）
    - 拦截器（Interceptor）
    - 切面（aspect）
---
# 前言

在平时开发中，经常回需要对请求进行拦截以达到某些目的。本文介绍3种方式来实现对请求拦截，分别是：过滤器（`filter`）,拦截器（`Interceptor`），切面 （`aspect`）。下面分别用这3种技术实现计算处理请求消耗时间的功能。

# 过滤器（filter）

`Filter`是`Servlet`规范中定义的,可以拦截请求和响应，以变换或使用包含在请求或响应中的信息。实现`javax.servlet.Filter`接口中声明了三个方法：
```
//Servlet容器启动时会调用该方法
void init(FilterConfig) 

//请求与过滤器设置匹配的URL时，会先经过该方法。
void doFilter(ServletRequest, ServletResponse, FilterChain) 

 //Servlet容器销毁时调用改方法
void void destroy()
```

`Filter`实现请求耗时计算功能：
```
package com.example.learn.restApi.config;

import org.springframework.stereotype.Component;

import javax.servlet.*;
import java.io.IOException;

@Component
public class TimeFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("timeFiler init");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws IOException, ServletException {
        long start = System.currentTimeMillis();
        filterChain.doFilter(request,response);
        System.out.println("timeFilter :"+(System.currentTimeMillis()-start));
    }

    @Override
    public void destroy() {
        System.out.println("timeFilter destroy");
    }
}
```

# 拦截器（Interceptor）
`Filter`属于`Servlet`范畴的`API`，与`Spring`没什么关系。除了使用`Filter`来过滤请web请求外，还可以使用`Spring`提供的`HandlerInterceptor`（拦截器）。

实现自己的拦截器需要：
1. 自定义一个类并实现`HandlerInterceptor`接口
2. 自定义一个类继承`WebMvcConfigurerAdapter`类，重写`addInterceptors`方法。
3. 在`addInterceptors`把第一步中自定义类的实例添加到拦截器链。

`HandlerInterceptor`接口中定义了三个方法：

```
//在HandlerMapping 决定那个Controller处理请求，且在进入Contrller方法之前进行调用,返回值决定请求是否继续处理，如果返回false，请求会被终止。
boolean preHandle (HttpServletRequest, HttpServletResponse, Object)

//请求进行处理之后，也就是Controller 方法调用之后执行。但是它会在DispatcherServlet 进行视图返回渲染之前被调用，所以我们可以在这个方法中对Controller 处理之后的ModelAndView 对象进行操作。
public void postHandle(HttpServletRequest, HttpServletResponse, Object, ModelAndView)

//该方法将在整个请求结束之后，也就是在DispatcherServlet 渲染了对应的视图之后执行。
void afterCompletion(HttpServletRequest, HttpServletResponse, Object, Exception)
```

`HandlerInterceptor`实现请求耗时计算功能：
```
package com.example.learn.restApi.config;

import org.springframework.stereotype.Component;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component
public class TimeIntorceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //打印类名
        System.out.println("class name:" + ((HandlerMethod) handler).getBean().getClass().getName());
        //打印方法名
        System.out.println("method name:"+ ((HandlerMethod)handler).getMethod().getName());
        request.setAttribute("startTime",System.currentTimeMillis());
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("TimeIntorceptor :"+(System.currentTimeMillis() - (long) request.getAttribute("startTime")));
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("TimeIntorceptor afterCompletion");
    }
}
```
在`spring`中注册`HandlerInterceptor`：
```
package com.example.learn.restApi.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {
    @Autowired
    private TimeIntorceptor timeIntorceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        super.addInterceptors(registry);
        //registry.addInterceptor(timeIntorceptor);
        //addPathPatterns 可以设置匹配url规则
        registry.addInterceptor(timeIntorceptor).addPathPatterns("/**");
    }
}
```

# 切面（aspect）

```
package com.example.learn.restApi.config;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class TimeAspect {

    @Around("execution(public * com.example.learn.restApi.*.controller.*(..))")
    public void excuTime(ProceedingJoinPoint joinPoint) throws Throwable {
        Object[] objs = joinPoint.getArgs();
        for (Object o : objs) {
            System.out.println(o);
        }
        long start = System.currentTimeMillis();
        joinPoint.proceed();
        System.out.println("timeAspect time:"+(System.currentTimeMillis()-start));
    }
}
```
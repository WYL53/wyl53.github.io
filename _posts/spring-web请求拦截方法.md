---
layout:     post
title:      java web 请求拦截的方法
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

在平时开发中，经常回需要对请求进行拦截以达到某些目的。本文介绍3种方式来实现对请求拦截，分别是：过滤器（filter）,拦截器（Interceptor），切面 aspect。下面分别用这3种技术实现计算处理请求的消耗时间。

# 过滤器（filter）

实现 Filter接口
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

# 切面（aspect）
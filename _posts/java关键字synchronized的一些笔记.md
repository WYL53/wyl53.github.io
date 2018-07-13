---
layout:     post
title:      java关键字synchronized的一些笔记
# subtitle:   java web 请求拦截的方法
date:       2018-07-06
author:     BY wyl53
# header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - java
		- synchronized
		- 多线程
		- 线程安全
---
# 前言
  曾经听别人说过用`synchroinized`关键字来修饰一个方法或者一个代码块的时候，能够保证在同一时刻最多只有一个线程执行该段代码，这样就线程安全了。那是不是所有方法都加上`synchroinized`关键字就能写出线程安全的代码呢？今天，笔者就谈谈个人对`synchroinized`关键字的理解。`synchroinized`的用法有以下两种：

```
  synchronized void method(){
    ...
  }
```
和
```
  synchronized(Object){
    ...
  }
```

1. 当`synchronized`修饰非静态方法时，需要获取到`this`当前对象的锁，才能执行方法体。所以当多个线程同时访问同一个对象的`synchronized`方法时候，只有一个线程能获得执行方法的权力，其他线程会被阻塞住。持有锁的线程执行完方法后会自动释放锁，然后被。
2. 当synchronized修饰静态方法时，需要获取到`this`当前对象的锁，才能执行方法体。
```
class SynchronizedTest{

  public void method1(){
    System.out.println(System.currentTimeMillis()+" "+Thread.currentThread().getName() +" in method1");
  }

  public synchronized void method2(){
    try{
      System.out.println(System.currentTimeMillis()+" "+Thread.currentThread().getName() +" in method2");
      Thread.sleep(1000);
    }catch(InterruptedException ex){
      ex.printStackTrace();
    }
  }

  public synchronized void method3(){
    try{
      System.out.println(System.currentTimeMillis()+" "+Thread.currentThread().getName() +" in method3");
      Thread.sleep(1000);
    }catch(InterruptedException ex){
      ex.printStackTrace();
    }
  }

  public static synchronized void method4(){
    try{
      System.out.println(System.currentTimeMillis()+" "+Thread.currentThread().getName() +" in method4");
      Thread.sleep(2000);
    }catch(InterruptedException ex){
      ex.printStackTrace();
    }
  }

  public static void main(String[] args){
    SynchronizedTest st = new SynchronizedTest();
    SynchronizedTest st2 = new SynchronizedTest();

      new Thread(()->{ st.method1();}).start();
      new Thread(()->{ st.method1();}).start();
      new Thread(()->{ st2.method1();}).start();

      // new Thread(()->{ st.method2();}).start();
      // new Thread(()->{ st.method3();}).start();
      // new Thread(()->{ st2.method2();}).start();

      // new Thread(()->{ SynchronizedTest.method4();}).start();
      // new Thread(()->{ st.method2();}).start();
      // new Thread(()->{ st2.method2();}).start();
  }
}
```

输出：
```
1531475927399 Thread-0 in method1
1531475927400 Thread-1 in method1
1531475927400 Thread-2 in method1
```

输出：
```
1531476030123 Thread-0 in method2
1531476030124 Thread-2 in method2
1531476031124 Thread-1 in method3
```

输出：
```
1531476101834 Thread-0 in method4
1531476101835 Thread-1 in method2
1531476101836 Thread-2 in method2
```
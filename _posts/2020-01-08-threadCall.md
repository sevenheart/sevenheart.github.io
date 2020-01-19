---
title: 有关线程通信的Demo
date: 2020-01-08 
categories:
- JAVA
tags:
- 并发
- thread
---

一些个人测试的线程间通信的编程题目：

#### 三个线程轮流打印1——100间的数

##### park及unpark的实现

```java
public class test {
    static Thread t1,t2,t3;
    static int i = 1;
    public static void main(String[] args) {
        t1 = new Thread(()->{
            while (i <= 100) {
                System.out.println("t1:" + i++);
                LockSupport.unpark(t2);
                LockSupport.park();
            }
        });
        t2 = new Thread(()->{

            while (i<=100) {
                LockSupport.park();
                if(i <= 100)
                    System.out.println("t2:" + i++);
                LockSupport.unpark(t3);
            }
        });
        t3 = new Thread(()->{

            while (i <= 100) {
                LockSupport.park();
                if (i <= 100)
                    System.out.println("t3:" + i++);
                LockSupport.unpark(t1);
            }
        });
        t1.start();
        t2.start();
        t3.start();

    }
}

```

##### synchronized关键字搭配wait和notifyall去实现

```java
public class WaitAndNotify {
    static int type = 1;//123表示那个线程执行
    static int i = 1;//要打印的初始值，必须写成类方法，不然run方法中无法调用。
    static Object object = new Object();//锁对象
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            while (i <= 100) {
                synchronized (object) {
                    object.notifyAll();//激活所有线程，防止出现每个线程都按顺序进入后阻塞无人唤醒的情况
                    if (i <= 100 && type == 1) {
                        System.out.println("t1:" + i++);
                        type = 2;
                        try {
                            object.wait();//阻塞当前线程并释放锁
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                    object.notifyAll();//激活所有线程
                }
            }
        });
        Thread t2 = new Thread(() -> {

            while (i <= 100) {
                synchronized (object) {
                    object.notifyAll();//激活所有线程，防止出现每个线程都按顺序进入后阻塞无人唤醒的情况
                    if (i <= 100 && type == 2) {
                        System.out.println("t2:" + i++);
                        type = 3;
                        try {
                            object.wait();//阻塞当前线程并释放锁
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }


                }
            }
        });
        Thread t3 = new Thread(() -> {
            while (i <= 100) {
                synchronized (object) {
                    object.notifyAll();//激活所有线程，防止出现每个线程都按顺序进入后阻塞无人唤醒的情况
                    if (i <= 100 && type == 3) {
                        System.out.println("t3:" + i++);
                        type = 1;
                        try {
                            object.wait();//阻塞当前线程并释放锁
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    object.notifyAll();//激活所有线程
                }
            }

        });
        t1.start();
        t2.start();
        t3.start();

    }
}

```

##### 线程安全的原子类适合做计数器，想要线程按一定顺序进行工作就必须用到锁。

##### volitile关键字去做，突然发现是最简单的。

```java
public class WaitAndNotify {
    static  volatile int type = 1;//123去表示该谁执行
    static  int  i = 0;
    public static void main(String[] args) {

        Thread t1 = new Thread(()->{
            while (i <= 100) {
               if(type == 1&& i <=100) {
                   System.out.println(Thread.currentThread().getName() + i++);
                   type = 2;
               }
            }
        },"线程1");
        Thread t2 = new Thread(()->{
            while (i <= 100) {
                if(type == 2&& i <=100) {
                    System.out.println(Thread.currentThread().getName() + i++);
                    type = 3;
                }
            }
        },"线程2");
        Thread t3 = new Thread(()->{
            while (i <= 100) {
                if(type == 3 && i <=100) {
                    System.out.println(Thread.currentThread().getName() + i++);
                    type = 1;
                }
            }
        },"线程3");

        t1.start();
        t2.start();
        t3.start();
    }
}

```


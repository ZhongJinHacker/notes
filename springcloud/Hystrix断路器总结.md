```xml
Hystrix 是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的可能调用失败，比如超时、异常等；Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性

“断路器”本身是一个开关装置，当某个服务单元发生故障后，通过断路器的故障监控，向调用方返回一个服务预期的、可处理的备选响应（fallback），而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩
```



#### 服务降级 -> 解决单节点某个接口超时或报错问题问题

fallback

##### 服务器忙。请稍后再试，不让客户端等待并立刻返回一个友好提示，fallback

#### ==哪些情况会发出降级==

```xml
1. 程序运行异常
2. 超时
3. 服务熔断触发服务降级
4. 线程池或信号量打满也会导致服务降级
```

例子：

```java
    // 3000 表示 3秒以内是正常逻辑,c 超过 执行 paymentInfo_TimeoutHandler 方法
    @HystrixCommand(fallbackMethod = "paymentInfo_TimeoutHandler",
            commandProperties = { @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000") })
    public String paymentInfo_Timeout(Integer id) {
        // 暂停几秒钟的线程
        int timeout = 5;
        try {
            TimeUnit.SECONDS.sleep(timeout);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "线程池: " + Thread.currentThread().getName() + "PaymentInfo_Timeout,id: " + id + "\t" + "耗时" + timeout + "秒~~";
    }
```



#### Hystrix 既可以放在消费端，也可以放在服务端



#### 服务熔断

break

### 类比保险丝达到最大服务访问后，直接拒绝访问，然后调用==服务降级==的方法友好提示



#### 服务限流

flowlimit

##### 秒杀等高并发的操作，严禁一窝蜂地拥挤，大家排队，有序进行


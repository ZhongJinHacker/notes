### Springboot 拦截器总结

**拦截器大体分为两类 ： handlerInterceptor 和 methodInterceptor**

**而methodInterceptor 又有==XML 配置方法== 和==AspectJ配置方法==**



#### 1. handlerInterceptor用法

handlerInterceptor 主要用于拦截有url映射的方法（即controller中与url有关联的方法）

 **[1] 先创建一个拦截器**

```java
package com.grady.interceptordemo.interceptor;


import lombok.extern.slf4j.Slf4j;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 自定义拦截器 - 给予 springmvc
 * @ClassName: CustomInterceptor
 * @Description: springMVC项目中的拦截器，它拦截的目标是请求的地址，比MethodInterceptor先执行。
 *                  该拦截器只能过滤action请求（即有url映射的函数，其他的不行），
 *                  spring允许有多个拦截器存在，由拦截器链管理
 *                  当preHandle return true时，执行下一个拦截器，直到所有拦截器执行完，再运行 被拦截的请求
 *                  当preHandle return false时, 不再执行 后续的拦截器链 及 被拦截的请求。
 */


@Slf4j
public class CustomInterceptor implements HandlerInterceptor {
    /**
     * 我未实现一个方法却没有报错，因为使用了Java8的语法，默认函数
     */

    /**
     * 进入对应的controller前
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        log.info("CustomInterceptor : preHandle");
        // 这里如果return false，不会执行后面的拦截器，也不会执行请求的方法了
        //return false;

        return true;
    }

    /**
     * 执行完controller方法后，但在返回对应视图前（返回json就没有返回视图的过程了）
     * @param request
     * @param response
     * @param handler
     * @param modelAndView
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("CustomInterceptor : postHandle");
    }

    /**
     * 整个请求调用结束后，视图返回后，做资源清理工作
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        log.info("CustomInterceptor : afterCompletion");
    }
}

```

**[2] 将拦截器添加到InterceptorRegistry配置中**

```java
package com.grady.interceptordemo.config;

import com.grady.interceptordemo.interceptor.CustomInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * WebMvcConfigurerAdapter 实际被 Deprecated 所以就不用纠结去不去继承他了，直接实现WebMvcConfigurer
 */
@Configuration
public class WebAppConfigurer implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
      	//这里配置了customInterceptor这个拦截器拦截所有url
        registry.addInterceptor(customInterceptor()).addPathPatterns("/**");
    }

    @Bean
    public CustomInterceptor customInterceptor() {
        CustomInterceptor customInterceptor = new CustomInterceptor();
        return customInterceptor;
    }
}
```



之后访问url接口，可以看到以下日志

```properties
c.g.i.interceptor.CustomInterceptor      : CustomInterceptor : preHandle
c.g.i.interceptor.CustomInterceptor      : CustomInterceptor : postHandle
c.g.i.interceptor.CustomInterceptor      : CustomInterceptor : afterCompletion
```





#### 2. 使用 methodInterceptor

   **[1] 使用XML配置方法**

​	XML 配置 先要创建自己的xxMethodInterceptor （实现MethodInterceptor）

```java
@Slf4j
public class CustomMethodInterceptor implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
        log.info("CustomMethodInterceptor ==> invoke method: process method name is {}", methodInvocation.getMethod().getName());
        log.info("CustomMethodInterceptor ==> invoke method: process class name is {}", methodInvocation.getMethod().getDeclaringClass());
        // 这里做自定义操作

        return methodInvocation.proceed();
    }
}
```

**然后再xml中进行配置**

spring-aop.xml (注释部分是打开对注解的支持，暂时未开)

```xml
<?xml version = "1.0" encoding = "UTF-8"?>
<beans xmlns = "http://www.springframework.org/schema/beans"
       xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop = "http://www.springframework.org/schema/aop"
       xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
   http://www.springframework.org/schema/aop
   http://www.springframework.org/schema/aop/spring-aop-3.0.xsd ">

  <bean id="customMethodInterceptor" class="com.grady.interceptordemo.interceptor.CustomMethodInterceptor"/>

  <aop:config proxy-target-class="false">
  <!-- 方法拦截器，基于spring aop 实现配置 -->
  <!-- 扫描使用了注解的方法进行拦截 -->
<!--  <aop:advisor pointcut="@annotation(com.grady.interceptordemo.annotation.CustomAnnotation)" advice-ref="customMethodInterceptor" />-->
  <!-- 指定包路径下的方法 -->
  <aop:advisor pointcut="execution(* com.grady.interceptordemo.controller.*.*(..))" advice-ref="customMethodInterceptor" />
  </aop:config>
</beans>
```

在xxxApplication.java导入XML `@ImportResource("classpath:spring-aop.xml")`



日志

```properties
 c.g.i.interceptor.CustomInterceptor      : CustomInterceptor : preHandle
 c.g.i.i.CustomMethodInterceptor          : CustomMethodInterceptor ==> invoke method: process method name is hello
 c.g.i.i.CustomMethodInterceptor          : CustomMethodInterceptor ==> invoke method: process class name is class com.grady.interceptordemo.controller.HelloController
 c.g.i.interceptor.CustomInterceptor      : CustomInterceptor : postHandle
 c.g.i.interceptor.CustomInterceptor      : CustomInterceptor : afterCompletion
```

==从这里也可以看出，handlerInterceptor 的prehandle方法先于methodInterceptor 的invoke方法执行，其他的方法再invoke 方法之后执行==

**再在spring-aop.xml中打开注解，看是否支持注解版本(这里注释掉了包扫描的拦截方式)**

```xml
<?xml version = "1.0" encoding = "UTF-8"?>
<beans xmlns = "http://www.springframework.org/schema/beans"
       xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop = "http://www.springframework.org/schema/aop"
       xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
   http://www.springframework.org/schema/aop
   http://www.springframework.org/schema/aop/spring-aop-3.0.xsd ">

  <bean id="customMethodInterceptor" class="com.grady.interceptordemo.interceptor.CustomMethodInterceptor"/>

  <aop:config proxy-target-class="false">
  <!-- 方法拦截器，基于spring aop 实现配置 -->
  <!-- 扫描使用了注解的方法进行拦截 -->
  <aop:advisor pointcut="@annotation(com.grady.interceptordemo.annotation.CustomAnnotation)" advice-ref="customMethodInterceptor" />
  <!-- 指定包路径下的方法 -->
<!--  <aop:advisor pointcut="execution(* com.grady.interceptordemo.controller.*.*(..))" advice-ref="customMethodInterceptor" />-->
  </aop:config>
</beans>
```

日志一致，说明是**生效**的

```properties
c.g.i.interceptor.CustomInterceptor      : CustomInterceptor : preHandle
c.g.i.i.CustomMethodInterceptor          : CustomMethodInterceptor ==> invoke method: process method name is hello
c.g.i.i.CustomMethodInterceptor          : CustomMethodInterceptor ==> invoke method: process class name is class com.grady.interceptordemo.manager.impl.HelloManagerImpl
c.g.i.interceptor.CustomInterceptor      : CustomInterceptor : postHandle
c.g.i.interceptor.CustomInterceptor      : CustomInterceptor : afterCompletion
```



#### [2] 再看AspectJ版本（推荐方式）

  ==SpectJ 不需要XML 配置，比较方便==

​	==直接声明切面切点就搞定问题了==

```java
@Component
@Slf4j
public class AspectJInterceptor {

    // 声明一个切点，表示从哪里切入增强，这里是切controller目录下的所有public 方法
    @Pointcut("execution (* com.grady.interceptordemo.controller.*.*(..))")
    public void logPoint() {

    }

  	//声明切面的执行方式，这里是环绕切面，环绕切是有返回值的Object
    @Around("logPoint()")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        long startTime = System.currentTimeMillis();
        log.info("AspectJInterceptor around的函数为：" + point.getSignature());
        log.info("AspectJInterceptor around 方法开始时间 : " + startTime);
        Object obj = point.proceed();// ob 为方法的返回值
        log.info("AspectJInterceptor  around 耗时 : " + (System.currentTimeMillis() - startTime));
        return obj;
    }
}
```



这样就可以了

 查看日志

```properties
AspectJInterceptor around的函数为：String com.grady.interceptordemo.controller.HelloController.hello()
c.g.i.interceptor.AspectJInterceptor     : AspectJInterceptor around 方法开始时间 : 1567261903109
c.g.i.interceptor.AspectJInterceptor     : AspectJInterceptor  around 耗时 : 9
```

说明生效了

上面是根据表达式寻找函数切的；现在换基于注解切

修改代码

```java
@Aspect
@Component
@Slf4j
public class AspectJInterceptor {
    // 声明一个切点 基于注解
    @Pointcut("@annotation(com.grady.interceptordemo.annotation.CustomAnnotation)")
    public void logPoint2() {

    }

    @Around("logPoint2()")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        long startTime = System.currentTimeMillis();
        log.info("AspectJInterceptor around的函数为：" + point.getSignature());
        log.info("AspectJInterceptor around 方法开始时间 : " + startTime);
        Object obj = point.proceed();// ob 为方法的返回值
        log.info("AspectJInterceptor  around 耗时 : " + (System.currentTimeMillis() - startTime));
        return obj;
    }
}
```



日志：

```properties
2019-08-31 22:45:52.275  INFO 4516 --- [nio-8080-exec-1] c.g.i.interceptor.AspectJInterceptor     : AspectJInterceptor around的函数为：String com.grady.interceptordemo.manager.impl.HelloManagerImpl.hello()
2019-08-31 22:45:52.276  INFO 4516 --- [nio-8080-exec-1] c.g.i.interceptor.AspectJInterceptor     : AspectJInterceptor around 方法开始时间 : 1567262752274
```



可以看的是==HelloManagerImpl.hello== 方法上有注解，所以切面方法中打印的是这个方法；而之前的版本切的位置是controller包中的方法，所以打印的是==HelloController.hello==





#### PS：

**可以在application.properties 中打开这个开关，开启CGlib ，（这个实验中不开是可以的，原因是当spring aop 无效时（比如没有实现接口的类controller），会自动使用CGLib尝试）**

```properties
# 启用CGLib 来实现aop
# spring.aop.proxy-target-class=true
```


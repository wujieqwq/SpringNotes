# AOP

AOP:Aspect Oriented Programming	面向切面编程

在程序运行期间，不修改源码对已有方法进行增强

减少重复代码，提高开发效率，方便维护

应用场景：日志记录，事务管理，权限验证，性能监测

## 实现方式

动态代理

## AOP核心概念

* JoinPoint(连接点)

  能被拦截到的点，在spring中指方法
* PointCut(切入点)

  指要被拦截的JointPoint
* Advice(通知/增强)

  拦截到JoinPoint之后所要做的事情，对切入点增强的内容
* Target(目标)

  指代理的目标对象
* Weaving(织入)

  把增强代码应用到目标上。生成代理对象的过程
* Proxy(代理)

  生成的代理对象
* Aspect(切面)

  切入点和通知的结合

‍

### AOP配置

XML

1. 导入依赖spring-aspects
2. 配置文件导入约束
3. 抽取公共代码制作成通知
4. 通知类bean

    ```xml
    <!-- 配置通知 -->
    <bean id="txManager" class="com.zzxx.utils.TransactionManager"/>
    ```

5. aop:config

    ```xml
    <aop:config>
    		<aop:pointcut expression="execution(* com.zzxx.service.impl.*.*(..))" id="pt1"/>

    		<!-- 配置切面 -->
    		<aop:aspect id="logAdvice" ref="mylogger">
    			<!-- 1. 配置前置通知 -->
    			<!-- <aop:before method="beforePrintLog" pointcut-ref="pt1" /> -->
    			<!-- 2. 配置后置通知 -->
    			<!-- <aop:after-returning method="afterReturningPrintLog" pointcut-ref="pt1" 
    				/> -->
    			<!-- 3. 配置异常通知 -->
    			<!-- <aop:after-throwing method="afterThrowingPrintLog" pointcut-ref="pt1" 
    				/> -->
    			<!-- 4. 配置最终通知 -->
    			<!-- <aop:after method="afterPrintLog" pointcut-ref="pt1" /> -->
    			<!-- 5. 配置环绕通知 -->
    			<aop:around method="aroundPrintLog" pointcut-ref="pt1" />
    		</aop:aspect>
    	</aop:config>
    ```

	切入点表达式：execution([修饰符] 返回值类型 包名.类名。方法名(参数)

‍

注解配置

1. 扫描包
2. 注解通知类

    ```java
    /**
    * 事务控制类
    */
    @Component("txManager")
    @Aspect //声明当前类是一个切面类
    public class TransactionManager {
    }
    ```

3. 配置增强方法

    ```java
    //开启事务
    @Before("execution(* com.zzxx.service.impl.*.*(..))")
    //开启事务
    public static void beginTransaction() {
    	// 设置事务不⾃动提交
    	System.out.println("设置事务不⾃动提交");
    	// 开启事务
    	System.out.println("开启事务");
    }

    //提交事务
    @AfterReturning("execution(* com.zzxx.service.impl.*.*(..))")
    public static void commit() {
    	System.out.println("提交事务");
    }

    //回滚事务
    @AfterThrowing("execution(* com.zzxx.service.impl.*.*(..))")
    public static void rollback() {
    	System.out.println("回滚事务");
    }

    //释放资源
    @After("execution(* com.zzxx.service.impl.*.*(..))")
    public static void release() {
    	System.out.println("释放资源");
    }


    /**
    * 环绕通知
    * @param pjp
    * @return
    */
    @Around("execution(* com.zzxx.service.impl.*.*(..))")
    public Object transactionAround(ProceedingJoinPoint pjp) {
    	// 定义返回值
    	Object rtValue = null;
    	try {
    		// 获取⽅法执⾏所需的参数
    		Object[] args = pjp.getArgs();
    		// 前置通知:开启事务
    		beginTransaction();
    		// 执⾏⽅法
    		rtValue = pjp.proceed(args);
    		// 后置通知:提交事务
    		commit();
    	 } catch(Throwable e) {
    		// 异常通知:回滚事务
    		rollback();
    		e.printStackTrace();
    	 } finally {
    		//最终通知:释放资源
    		release();
    	 }
    	return rtValue;
    }
    ```

4. 开启注解支持

    ```xml
    <!-- 开启 spring 对注解 AOP 的⽀持 -->
    <aop:aspectj-autoproxy />
    ```

5. 切入点设置

    ```java
    @Pointcut("execution(* com.zzxx.service.impl.*.*(..))")
    private void pt1() {}
    ```

‍

	纯注解的spring配置

```java
@Configuration
@ComponentScan(basePackages="com.zzxx")
@EnableAspectJAutoProxy
public class SpringConfiguration {
}
```

‍

	```

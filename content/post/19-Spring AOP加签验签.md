---
title: 'Spring AOP加签验签'
tags: ["Spring","AOP"]
date: 2019-08-28
draft: false
---

# 1.场景

项目中需要对特定的接口进行验签和加签操作.对方法的请求参数验签,保证请求方的数据可信性.对方法的返回参数进行加签操作,确保发送方的数据可信性.验签和加签这两个方法已经封装到了一个`Service`中,只要在需要加签验签的接口调用此方法,就能满足需要.但是这样的方法会导致在接口中存在重复的验签,加签调用代码.

我们来大致看下项目此时的情况(伪代码,主要看代码逻辑)

**基本信息返回类`BaseResponse`**

```java
public class BaseResponse<T> {
    private String code;
    private String msg;
    private String sign;
    //getter setter方法
    public BaseResponse(String code,String msg){
        this.code = code;
        this.msg = msg;
    }
}
```

**加签验签`Service`**

```java
@Service
public class MD5Sign {
    /**
     * 返回加签后的sign
     * @param object
     * @return
     */
    public String getSign(Object object) {
        //省略方法实现细节
        return "sign";
    }

    /**
     * 检验签名是否可用
     * @return true->校验签名成功
     * false->校验签名失败
     */
    public boolean checkSign(Object object) {
        //省略方法实现细节
        return true;
    }

    /**
     * 对BaseResponse进行加签
     * @param baseResponse
     * @return
     */
    public BaseResponse setSignForRes(BaseResponse baseResponse) {
        String sign = this.getSign(baseResponse);
        baseResponse.setSign(sign);
        return baseResponse;
    }
}
```

**下单方法**

```java
@RestController
@RequestMapping("api")
public class PreOrderController {
    @PostMapping("order")
    public BaseResponse order(@RequestBody @Validated BasePayRequest request) {
        //验签失败,返回信息
        if (!md5Sign.checkSign(request)) {
            return md5Sign.setSignForRes(new BaseResponse("301","验签失败"));
        }

        //如果订单号重复,返回信息.
        if(mchrOrderNoisDuplicate==true){
            return md5Sign.setSignForRes(new BaseResponse("302","订单号重复"));
        }
        
        //初始化返回对象
        BaseResponse response = new BaseResponse();
        
        //省略下单代码中对response的处理
        return md5Sign.setSignForRes(response);
    }
}
```

从下单方法中就可以看出,加签方法被频繁的调用,虽然验签方法只被调用了一次,但是这仅是一个方法,后面还有需要加签的退款,订单查询,退款查询等接口.所以如果能够将这部分代码抽象出来,可以在一定程度上精简代码.

# 2.`AspectJ AOP`框架

针对以上的场景,需要做的改动就是对特定方法的请求参数进行校验,返回参数进行修改.这其实就是`Spring`的面向切面编程(`AOP`)思想.

如果有朋友对`AOP`的概念不是太了解,那咱们通过简单讲下`AspectJ`的用法来熟悉下.

首先确保你有`aopalliance.jar`、`aspectj.weaver.jar` 和 `spring-aspects.jar`这些`jar`包.

`xml`中添加如下配置代码:

```xml
	<!-- 配置自动扫描的包 -->
    <context:component-scan base-package="com.alex.aop"/>
    <!-- 使aspectj注解起作用 -->
    <aop:aspectj-autoproxy/>
```

然后创建我们需要拦截的方法,实现一个简易的计算器.

`ArithmeticCalculator.java`接口

```java
public interface ArithmeticCalculator {
    int add(int i, int j);
    int sub(int i, int j);
    int multiply(int i, int j);
    int div(int i, int j);
}
```

`ArithmeticCalculator.java`接口实现类

```java
@Component
public class ArithmeticCalculatorImpl implements ArithmeticCalculator {
    @Override
    public int add(int i, int j) {
        return i + j;
    }

    @Override
    public int sub(int i, int j) {
        return i - j;
    }

    @Override
    public int multiply(int i, int j) {
        return i * j;
    }

    @Override
    public int div(int i, int j) {
        return i / j;
    }
}
```

**`AspectJ`支持五种类型的通知注解**

* `@Before`: 前置通知, 在方法执行之前执行
* `@After`: 后置通知, 在方法执行之后执行
* `@AfterRunning`: 返回通知, 在方法返回结果之后执行
* `@AfterThrowing`: 异常通知, 在方法抛出异常之后
* `@Around`: 环绕通知, 围绕着方法执行

下面我们来配置一个切面类.

```java
//声明此类为切面类
@Aspect
@Component
public class LoggingAspect {
    
    //在方法执行前执行
    @Before("execution(public int com.alex.aop.ArithmeticCalculator.*(int,int ))")
    public void beforeMethod(JoinPoint joinPoint) {
        //获取方法名称
        String methodName = joinPoint.getSignature().getName();
        List<Object> list = Arrays.asList(joinPoint.getArgs());
        System.out.println("Before Method [" + methodName + "] Begins with " + list);
    }
    
    //在方法执行之后执行的代码. 无论该方法是否出现异常
    @After("execution(public int com.alex.aop.ArithmeticCalculator.*(int,int ))")
    public void afterMethod(JoinPoint joinPoint) {
        String name = joinPoint.getSignature().getName();
        System.out.println("After Method [" + name + "] Ends");
    }
    
    //在方法正常执行的情况下 AfterReturning可以拿到返回值
    @AfterReturning(value = "execution(public int com.alex.aop.ArithmeticCalculator.*(int,int ))", returning = "result")
    public void returnMethod(JoinPoint joinPoint, Object result) {
        String name = joinPoint.getSignature().getName();
        System.out.println("Method [" + name + "] Ends With [" + result + "] ");
    }
    
    //在方法发生异常的情况下执行 (可以通过Exception的类型,对所需捕获的异常进行限制)
    //捕捉所有异常 void afterThrowing(JoinPoint joinPoint, Exception e)
    //只捕捉空指针异常 void afterThrowing(JoinPoint joinPoint, NullPointerException e)
    @AfterThrowing(value = "execution(public int com.alex.aop.ArithmeticCalculator.*(int,int ))",throwing = "e")
    public void afterThrowing(JoinPoint joinPoint, Exception e) {
        String name = joinPoint.getSignature().getName();
        System.out.println("Method [" + name + "] Have Exception With [" + e + "] ");
    }
    
    //围绕方法执行 包含了以上四种注解功能
    @Around(value = "execution(public int com.alex.aop.ArithmeticCalculator.*(int,int ))")
    public Object around(ProceedingJoinPoint pjp) {
        Object result = null;
        String methodName = pjp.getSignature().getName();
        try {
            //前置通知
            System.out.println("Before Method [" + methodName + "] Begins with " + Arrays.asList(pjp.getArgs()));
            result = pjp.proceed();
            //返回通知
            System.out.println("Method [" + methodName + "] Ends With [" + result + "] ");
        } catch (Throwable throwable) {
            //异常通知
            System.out.println("Method [" + methodName + "] Have Exception With [" + throwable + "] ");
            throwable.printStackTrace();
        }
        //后置通知
        System.out.println("After Method [" + methodName + "] Ends");
        return result;
    }
}
```

# 3.解决方案

现在我们就将`AOP`思想与项目结合起来.

**`maven`依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

**创建一个`Sign`注解类,通过此注解来标识拦截的方法**

```java
//程序运行时,程序可获取该Annotation注解信息
@Retention(RetentionPolicy.RUNTIME)
//指定此注解修饰方法
@Target(ElementType.METHOD)
public @interface Sign {
}
```

**声明切面类**

```java
@Aspect
@Component
@Slf4j
public class SignAspect {

    @Autowired
    private MD5Sign md5Sign;

    //绑定Sign注解,这样Sign注解修饰的方法执行时将会进入使用signPointCut()的方法(如around(ProceedingJoinPoint joinPoint))
    @Pointcut("@annotation(com.kltong.oversea.payment.aspect.Sign)")
    public void signPointCut() {
    }

    //环绕方法
    @Around("signPointCut()")
    public Object around(ProceedingJoinPoint joinPoint) {
        //获取方法请求参数
        Object[] args = joinPoint.getArgs();
        //将需要校验的参数Class类型放入checkedClassList中(这里的请求类不再列举,各位根据实际情况修改).
        List<Class> checkedClassList = new ArrayList<>(Arrays.asList(BasePayRequest.class, BaseRefundRequest.class, BaseTradeQueryRequest.class));

        for (int i = 0; i < args.length; i++) {
            //如果args中的对象Class类型存在于checkedClassList中,对此对象验签.
            if (checkedClassList.contains(args[i].getClass())) {
                //验签失败,抛出异常,交由异常处理器处理.
                if (!md5Sign.checkSign(args[i])) {
                    log.error("checkSignFail#arg->{}", args[i]);
                    throw new PaymentException(ResponseConstant.CHECKSIGN_FAIL);
                }
            }
        }

        Object result = null;
        //获取方法名称
        String methodName = joinPoint.getSignature().getName();
        try {
            //执行方法之后取得执行结果
            result = joinPoint.proceed();
            //如果执行结果是BaseResponse类型,那么对结果加签并直接返回.
            if (result instanceof BaseResponse) {
                BaseResponse response = (BaseResponse) result;
                return md5Sign.setSignForRes(response);
            }
        } catch (Throwable throwable) {
            log.error(methodName + "#Exception", throwable);
        }
        return result;
    }
}
```

**在下单方法中的使用**

```java
@RestController
@RequestMapping("api")
public class PreOrderController {
    
    //添加Sign注解标识此方法
	@Sign
    @PostMapping("order")
    public BaseResponse order(@RequestBody @Validated BasePayRequest request) {
        
        //如果订单号重复,返回信息.
        if(mchrOrderNoisDuplicate==true){
            return new BaseResponse("302","订单号重复");
        }
        
        //初始化返回对象
        BaseResponse response = new BaseResponse();
        
        //省略下单代码中对response的处理
        return response;
    }
}
```

可以看出代码量减少了一些,在需要验签,加签的方法上只需要添加`@Sign`注解.

# 4.参考

[Springboot中Aspect实现切面（以记录日志为例）](https://blog.csdn.net/zhuzhezhuzhe1/article/details/80565067)

[使用Spring AOP修改请求、返回参数](<https://my.oschina.net/niithub/blog/1925045>)


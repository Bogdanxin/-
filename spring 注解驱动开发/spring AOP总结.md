# Spring AOP总结

## 动态代理  

动态代理是实现AOP的很好的方式，这里可以通过**JDK的动态代理**和**GGLib的动态代理**

### JDK动态代理

JDK动态代理主要涉及两个类：Proxy和InvocationHandler，InvocationHandler是一个接口，可以通过实现该接口定义横切逻辑，并通过反射机制调用目标类的代码，动态地将横切逻辑和业务逻辑编织在一起

Proxy 利用InvocationHandler动态**创建一个<font color=red>符合某一个接口的实例</font>，生成目标类的代理对象**

步骤：

1. 创建接口，并创建类实现接口。

2. 创建切面方法类，该类声明了要切入的方法，即，将横切逻辑和目标方法逻辑编织在一起。

3. 创建横切切面类，该类实现InvocationHanlder接口，重写`invoke(Object proxy, Method method, Object[] args)`方法。在这个方法中proxy代表原来目标类生成的代理对象，method指的是目标类中的方法，args代表方法的入参。在此方法中，描写切面方法，并将目标方法织入其中。

   调用目标类中的方法是使用的`Method#invoke(Object obj, Object... args)`方法，obj是目标类，args是目标类方法的参数

4. 声明目标类，将目标类传入横切类中，再通过Proxy类的静态方法`newProxyInstance()`方法，传入横切类，从而生成目标类的 代理类，该代理类是织入了横切方法的，相当于改动了目标类的原方法。

```java
// 定义接口
public interface ForumService {
    int remove(int id);

    void update(String name);
}

// 目标类实现接口
public class ForumServiceImpl implements ForumService {

    public int remove(int id) {
        System.out.println("ForumServiceImpl remove id" + id);
        return id;
    }

    public void update(String name) {
        System.out.println("ForumServiceImpl update");
    }
}

// 实现InvocationHandler接口，作为横切面
public class PerformanceHandler implements InvocationHandler {

    private Object target;

    public PerformanceHandler(Object target) {
        this.target = target;
    }

	// 通过这个方法将原来的目标方法和想要添加的方法进行织入
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        PerformanceMonitor.begin(target.getClass().getName() + "." + method.getName());// begin和end都是要织入的方法
        Object result = method.invoke(target, args);
        PerformanceMonitor.end();
        return result;
    }
}

public class JDKProxyTest {

    public static void main(String[] args) {
        ForumService target = new ForumServiceImpl();

        PerformanceHandler performanceHandler = new PerformanceHandler(target);

        // 通过Proxy生成代理类，代理类中的相关方法都是织入了想要的方法的
        ForumService forumService = (ForumService) Proxy.newProxyInstance(
            							target.getClass().getClassLoader(),
                						target.getClass().getInterfaces(),
                						performanceHandler);

        forumService.remove(123);
        forumService.update("fasdfasdf");


    }

}
```

利用JDK的动态代理方法，是需要接口实现的。如果没有接口，是没法通过JDK动态代理实现的

### CGLib动态代理

CGLib采用底层的字节码技术，**可以为一个类创建子类**，在子类中采用方法拦截的技术拦截所有父类的方法调用并顺势织入横切逻辑。

步骤：

1. 创建一个MethodInterceptor接口实现类，在这个类中传入需要代理的目标类，对目标类使用字节码技术动态创建子类实例。这一步需要Enhancer类的帮助
2. 实现类要实现接口的`MethodInterceptor#intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy)`方法，和JDK中的方法相似，也是要将目标类中的方法织入到其他方法中，通过methodProxy变量调用目标类（父类）中的方法

```java
public class CglibProxy implements MethodInterceptor {

    // 通过Enhancer类字节码技术动态创建子类实例
    private Enhancer enhancer = new Enhancer(); 
	// 设置需要创建的子类实例
    public Object getProxy(Class clazz) {
        enhancer.setSuperclass(clazz);
        enhancer.setCallback(this);
        return enhancer.create();
    }
	// 拦截父类的所有方法调用
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        // 切入方法
        PerformanceMonitor.begin(o.getClass().getName() + "." + method.getName());
        // 利用methodProxy的方法调用父类的方法
        Object result = methodProxy.invokeSuper(o, objects);
        PerformanceMonitor.end();
        return result;
    }
}

	@Test
	public void test3() {
        CglibProxy proxy = new CglibProxy();
        // 无需再让类实现接口，直接将类传入到横切类中，通过横切类的getProxy方法，
        // 通过字节码技术动态生成类的子类代理类，通过子类拦截父类的方法，从而织入新的逻辑
        ForumServiceImpl forumService = (ForumServiceImpl)proxy
            							.getProxy(ForumServiceImpl.class);
        
        forumService.removeForum(12332);
        forumService.removeTopic(12332);

    }


```



## 增强类

AOP使用增强类定义横切逻辑，定义的Advice接口用于增强

### 一般方式

步骤：

1. 声明一个目标接口，并创建实现类

2. 声明一个增强类，实现增强接口

   MethodBeforeAdvice（前置增强）、AfterReturningAdvice（后置增强）、MethodInterceptor（环绕增强）、ThrowAdvice（异常抛出增强）… …

3. 重写接口中的方法，在增强对应方法前置或者后置等等，描写对应的逻辑

4. 最后使用Spring提供的代理工厂类ProxyFactory，设置代理目标（设置目标类），为代理添加增强，通过工厂类生成代理类。通过代理类调用织入后的方法。

```java
// 声明目标类的接口
public interface Waiter {

    void greetTo(String name) ;

    void serveTo(String name);

}

// 声明目标类的接口实现类
public class NaiveWaiter implements Waiter {

    public void greetTo(String name) {
        System.out.println("NaiveWaiter:greet to " + name + "...");
    }

    public void serveTo(String name) {

        System.out.println("NaiveWaiter:serving to " + name + "...");
    }
}


public class AdviceTest {
	
    // 通过代理工厂类生成代理类
    @Test
    public void before() {
        Waiter waiter = new NaiveWaiter();
        BeforeAdvice advice = new GreetingBeforeAdvice();

        ProxyFactory proxyFactory = new ProxyFactory();
        // 设置代理目标
		proxyFactory.setTarget(waiter);
        // 添加增强
        proxyFactory.addAdvice(advice);
		// 通过工厂类生成代理类
        Waiter proxy = (Waiter) proxyFactory.getProxy();
        proxy.greetTo("zhangsan");
        proxy.serveTo("zhangsan");
    }
}
```

由于ProxyFactory可以通过JDK动态代理和CGLib动态代理两种方法创建代理类。

使用`ProxyFactory#setInterfaces()`方法可以设置为JDK动态代理，使用`ProxyFactory#setOptimize(true)`可以设置为CGLib动态代理

### 通过Spring配置

可以通过Spring配置将代理目标对象、增强类和ProxyFactoryBean注入到Spring容器中。这里的ProxyFactoryBean就是Spring提供的ProxyFactory在Spring中的实现。作为替代

并且对ProxyFactoryBean进行设置，指定其代理目标的接口，指定代理目标类、指定增强、指定是需要CGLib还是JDK动态

```java
@Configuration
public class AdviseConfig { 
	@Bean(name = "waiter")
    public ProxyFactoryBean proxyFactoryBean() throws ClassNotFoundException {
        ProxyFactoryBean proxyFactoryBean = new ProxyFactoryBean();
        proxyFactoryBean.setProxyInterfaces(new Class[] {Waiter.class});
        proxyFactoryBean.setInterceptorNames("greetingAdvice", "greetingAfterAdvise");
        proxyFactoryBean.setTargetName("target");
        return proxyFactoryBean;
    }
}
```



## 创建切面

通过声明切点、找到切面，可以通过切点对切面进行相关的增强。

### TODO：辨别切点和切面的区别

### 通过静态匹配切面

有两种方式，分别是通过普通的方法名匹配方法，另一种是通过正则表达式匹配，正则我不懂，其实差不多，就只看前面的方法了

步骤：

1. 创建若干个业务类
2. 创建切点类，声明匹配规则，用来匹配切点的方法。该类要继承StaticMethodMatcherPointcutAdvisor类，重写``matches(Method method, Class clazz)``方法切点的匹配规则，也可以重写``getClassFilter()``方法，用来匹配切点切入的类
3. 声明增强类，织入增强的方法逻辑
4. 配置切面将目标业务逻辑类、增强类、切点类、ProxyFactoryBean注入到Spring容器中。并且，在切点Bean中要注入想要添加的增强bean。将每个ProxyFactoryBean注入不同的业务逻辑类，切点Bean，最后生成不同业务逻辑类的对应的ProxyFactory
5. 从Spring容器中获取对应的代理类Bean，此时的代理类Bean通过切点已经织入了相关的逻辑对象。可以通过这些代理业务逻辑类调用方法了

```java
// 省略业务逻辑类的代码
// 声明切点类，继承StaticMethodMatcherPointcutAdvisor类
public class GreetingAdvisor extends StaticMethodMatcherPointcutAdvisor {

	/**
	 * 通过matches方法，书写匹配的规则，这样就相当于找到了对应的切点
	 */
    public boolean matches(Method method, Class<?> targetClass) {
        return "greetTo".equals(method.getName());
    }

    /**
     * 这个方法可以覆盖也可以不写，用来匹配类的，书写这个规则可以过滤相关的类
     */
    @Override
    public ClassFilter getClassFilter() {
        return new ClassFilter() {
            public boolean matches(Class<?> clazz) {
                return Waiter.class.isAssignableFrom(clazz);
            }
        };
    }
}

// 声明增强类
public class GreetingBeforeAdvice implements MethodBeforeAdvice {

    public void before(Method method, Object[] args, Object target) throws Throwable {
        System.out.println(target.getClass().getName() + "." + method.getName());
        String clientName = (String) args[0];
        System.out.println("How are you! Mr." + clientName + "!");
    }
}

// 声明配置类，将业务逻辑类、增强类、切点类、业务逻辑类对应的ProxyFactoryBean分别注入
// 对切点类注入相关的增强类，对ProxyFactoryBean注入相关配置
@Configuration
public class AdvisorConfig {
    @Bean("waiterTarget")
    public Waiter waiterTarget() {
        return new Waiter();
    }

    @Bean("sellerTarget")
    public Seller sellerTarget() {
        return new Seller();
    }

    @Bean(name = "greetingAdvice")
    public GreetingBeforeAdvice greetingBeforeAdvice() {
        return new GreetingBeforeAdvice();
    }

    @Bean("greetingAdvisor")
    public GreetingAdvisor greetingAdvisor() {
        GreetingAdvisor advisor = new GreetingAdvisor();
        advisor.setAdvice(greetingBeforeAdvice());
        return advisor;
    }

    @Bean("waiter")
    public ProxyFactoryBean proxyFactoryBean() {
        ProxyFactoryBean proxyFactoryBean = new ProxyFactoryBean();
        proxyFactoryBean.setInterceptorNames("greetingAdvisor");
        proxyFactoryBean.setProxyTargetClass(true);
        proxyFactoryBean.setTargetName("waiterTarget");
        return proxyFactoryBean;
    }



    @Bean("seller")
    public ProxyFactoryBean seller() {
        ProxyFactoryBean proxyFactoryBean  = new ProxyFactoryBean();
        proxyFactoryBean.setInterceptorNames("greetingAdvisor");
        proxyFactoryBean.setProxyTargetClass(true);
        proxyFactoryBean.setTargetName("sellerTarget");
        return proxyFactoryBean;
    }

}


public class AdvisorTest {

    @Test
    public void test1() {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AdvisorConfig.class);
        Waiter waiter = (Waiter) context.getBean("waiter");
        Seller sellefasdfr = (Seller) context.getBean("seller");

        waiter.greetTo("john");
        waiter.serveTo("john");
        seller.greetTo("john");

    }
}
```

### 动态切面、流程切面、复合切点切面…

## 自动创建代理

使用切点时候，都是要通过Spring的ProxyFactoryBean创建织入切面的代理，每个需要被代理的Bean都需要使用一个ProxyFactoryBean进行配置，比较麻烦。可以通过Spring的自动代理机制，让容器自动生成代理。内部使用的是Spring的BeanPostProcessor自动完成这个任务的。

基于BeanPostProcessor的自动代理创建实现类会根据一些规则自动在容器实例化Bean时候，为匹配的bean生成代理实例（这不仅在植入切面时候起作用，也在实例化、初始化bean等方面都有作用）。

这些代理创建器可以分为3类：

* **基于Bean配置名规则的自动代理创建器**：允许为一组特定配置名的Bean自动创建代理实例的代理创建器，实现类为BeanNameAutoWareProxyCreator
* **基于Advisor匹配机制的自动代理创建器**：他会对容器中的所有Advisor进行扫描，**自动将这些切面应用到匹配的Bean中（为目标Bean创建代理实例）**，实现类为DefaultAdvisorAutoProxyCreator
* **基于Bean中的AspectJ注解标签的自动代理创建器：**为包含AspectJ注解的bean自动创建代理实例，实现类为AnnotationAwareAspectJAutoProxyCreator

对于最后一种 AnnotationAwareAspectJAutoProxyCreator 需要后面再解析

### BeanNameAutoWareProxyCreator

使用步骤：

1. 前提是已有业务逻辑类，增强类（至于增强为前置还是后置返回等等不需要关心），并注入到Spring容器中
2. 注入BeanNameAutoWareProxyCreator中，并添加配置属性，可以通过``BeanNameAutoProxyCreator#setBeanNames()``方法设置根据BeanName筛选出目标类bean，并对筛选出的bean中的所有方法织入增强。

```java
@Configuration
public class AdviseConfig {
    
    @Bean
    public BeanNameAutoProxyCreator beanNameAutoProxyCreator() {
        BeanNameAutoProxyCreator beanNameAutoProxyCreator = new BeanNameAutoProxyCreator();
        beanNameAutoProxyCreator.setBeanNames("*er");
        beanNameAutoProxyCreator.setInterceptorNames("greetingAdvice", "greetingAfterAdvise");
        beanNameAutoProxyCreator.setOptimize(true);
        return beanNameAutoProxyCreator;
    }
}
```

这种方式只能根据BeanName筛选出所需要的bean并对其所有的方法织入切面。无法将筛选出的Bean某个一个方法织入切面

### DefaultAdvisorAutoProxyCreator

由于Advisor中已经包含了切点（织入哪里，这是matches方法要做的）、切入逻辑（要织入什么，这是增强方法要做的）。可以直接通过Advisor定义清楚了。所以我们可以通过扫描出所有的Advisor，并将其织入到匹配的bean中，即为匹配的目标Bean自动创建代理。这就是DefaultAdvisorAutoProxyCreator能够做到的。

步骤：

1. 业务Bean、增强Bean、Advisor切点bean、都要注入到Spring容器中。尤其是Advisor，在注入spring容器时候，一定要调用`setAdvice(Advice advice)`方法，设置增强方法。Advisor的定义，就多种多样了。
2. 在Spring容器中，注入DefaultAdvisorAutoProxyCreator组件，该组件会自动扫描所有的Advisor，并将Advisor相关逻辑织入到匹配的bean中。这样就能够对Bean进行增强

```java
// 省略了业务类的代码和实现增强接口的类，只在配置类注入Bean
@Configuration
public class Config {

    @Bean
    public Waiter waiter () {
        return new Waiter();
    }

    @Bean
    public Seller seller() {
        return new Seller();
    }

    // 对于切点类的定义，可以使用自定义的实现接口的增强类，
    // 也可以使用Spring创建好的切点类
    @Bean
    public GreetBeforeAdvisor greetBeforeAdvisor() {
        return new GreetBeforeAdvisor();
    }

//    @Bean
//    public GreetAdvisor greetAdvisor() {
//        GreetAdvisor greetAdvisor = new GreetAdvisor();
//        greetAdvisor.setAdvice(greetBeforeAdvisor());
//        return greetAdvisor;
//    }

    @Bean
    public RegexpMethodPointcutAdvisor regexpMethodPointcutAdvisor() {
        RegexpMethodPointcutAdvisor advisor = new RegexpMethodPointcutAdvisor();
        advisor.setAdvice(greetBeforeAdvisor());
        advisor.setPatterns(".*greet.*");
        return advisor;
    }

    @Bean
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
        return new DefaultAdvisorAutoProxyCreator();
    }
}
```



## 基于`@AspectJ`（主要）和Schema 的 AOP

一般步骤：

1. 创建业务类、切面类、配置类
2. 业务类专心于业务
3. 切面类添加`@AspectJ`注解，并创建方法，使用`@Before`、`@AfterReturning`等注解声明切点和增强类型
4. 配置类注入业务类、切面类Bean，并在配置类上添加`@EnableAspectJAutoProxy`注解。

```java
// 切面类
@Aspect
public class EnableSellerAspect {

    @DeclareParents(value = "springAOP.advice.NaiveWaiter", defaultImpl = SellerImpl.class)
    public Seller seller;
}

// 配置类
@Configuration
@EnableAspectJAutoProxy
public class AspectjConfig {

    @Bean
    public Waiter naiveWaiter() {
        return new NaiveWaiter();
    }

    @Bean
    public PreGreetingAspect greeting() {
        return new PreGreetingAspect();
    }

    @Bean
    public AnnotationAwareAspectJAutoProxyCreator annotationAwareAspectJAutoProxyCreator() {
        return new AnnotationAwareAspectJAutoProxyCreator();
    }
}

// 其他就不写了
```

后面的@AspectJ的一些详解和对切点声明的注解就不多说了，只需要记一记就可以了
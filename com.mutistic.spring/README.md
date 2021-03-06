# <a id="a_top">[Spring](https://docs.spring.io/spring/docs/current/javadoc-api)</a>
[Spring Boot](https://github.com/mutistic/mutistic.spring/blob/master/com.mutistic.boot/README.md)是伴随着Spring4.0诞生的<br/>
[spring Framework API](https://docs.spring.io/spring/docs/current/javadoc-api)<br/>
[spring Framework API-无框架](https://docs.spring.io/spring/docs/current/javadoc-api/overview-summary.html)<br/>
[Spring Framework Documentation](https://docs.spring.io/spring/docs/current/spring-framework-reference)<br/>

### <a id="a_catalogue">目录</a>：
1. <a href="#a_annotationConfigApplicationContext">AnnotationConfigApplicationContext 独立的应用程序上下文</a>
2. <a href="#a_configuration">@Configuration 配置注解</a>
3. <a href="#a_bean">@Bean bean注解</a>
4. <a href="#a_getBean">spring 获取bean的方式</a>
5. <a href="#a_initial">指定 bean的 initial（初始化） 和 destroy（销毁） 方法</a>
6. <a href="#a_component">使用@Component、@Repository、@Service、@Controller、@Aspect 等方式注册bean</a>
7. <a href="#a_componentScan">@ComponentScan 扫描注解<a/>
8. <a href="a_beanPostProcessor">BeanPostProcessor bean后置处理器</a>
9. <a href="#a_applicationContextAware">Aware 之 ApplicationContextAware</a>
10. <a href="#a_beanFactoryPostProcessor">BeanFactoryPostProcessor 上下文的基础工厂Bean后置处理器</a>
99. <a href="#a_down">down</a>

---
### <a id="a_annotationConfigApplicationContext">一、AnnotationConfigApplicationContext 独立的应用程序上下文：</a>
AnnotationConfigApplicationContext [org.springframework.context.annotation.AnnotationConfigApplicationContext](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/AnnotationConfigApplicationContext.html)

GenericApplicationContext [org.springframework.context.support.GenericApplicationContext](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/support/GenericApplicationContext.html)

AbstractApplicationContext [org.springframework.context.support.AbstractApplicationContext](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/support/AbstractApplicationContext.html)

```
1、加载机制：AnnotationConfigApplicationContext > GenericApplicationContext > AbstractApplicationContext.refresh()
2、独立的应用程序上下文，接受注释类作为输入 - 特别是注释类 @Configuration，但也包括普通 @Component类型和使用javax.inject注释的符合JSR-250和JSR-330的类。
3、允许使用逐个注册类register(Class...)以及使用类路径扫描 scan(String...)。
4、在多个@Configuration类的情况下，Bean后面的类中定义的@ 方法将覆盖在之前的类中定义的那些方法。这可以用来通过一个额外的@Configuration 类故意重写某些bean定义
```


1.1、通过加载 @Configuration 注解类（配置类）实现bean的注册</br>

```Java
package com.mutistic.annotation;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * 过加载 @Configuration 配置类实现bean的注册
 */
public class AnnotationMain {
	public static void main(String[] args) {
		AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AnnotationConfig.class);
		CommonConstant.printPref("@Configuration 配置类 注册的bean", ctx.getBean(Runnable.class));
		ctx.close();
	}
}
```

1.2、通过加载 Class 实现bean的注册</br>

```Java
package com.mutistic.annotation;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Controller;
import org.springframework.stereotype.Repository;
import org.springframework.stereotype.Service;
import com.mutistic.annotation.beans.TestAnnotationBean;
import com.mutistic.annotation.id.IDConfig;
import com.mutistic.annotation.register.TestAspect;
import com.mutistic.annotation.register.TestController;
import com.mutistic.annotation.register.TestRepositoryDao;
import com.mutistic.annotation.register.TestService;

/**
 * 通过加载 Class 实现bean的注册
 */
public class AnnotationMain {
	public static void main(String[] args) {
		AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AnnotationConfig.class,
				TestController.class, TestService.class, TestRepositoryDao.class, TestAspect.class, TestAnnotationBean.class);
		CommonConstant.printPref("@Configuration bean",ctx.getBean(AnnotationConfig.class));
		CommonConstant.printPref("@Controller bean", ctx.getBean(TestController.class));
		CommonConstant.printPref("@Service bean", ctx.getBean(TestService.class));
		CommonConstant.printPref("@Repository bean",ctx.getBean(TestRepositoryDao.class));
		CommonConstant.printPref("@TestAspect bean",ctx.getBean(TestAspect.class));
		CommonConstant.printPref("无任何注解 bean", ctx.getBean(TestAnnotationBean.class));
		ctx.close();
	}
}
```

1.3、通过加载 @ComponentScan 注解 Class（扫描类） 实现bean的注册 </br>
AnnotationMain.java

```Java
package com.mutistic.annotation;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import com.mutistic.annotation.beans.TestAnnotationBean;
import com.mutistic.annotation.id.IDConfig;
import com.mutistic.annotation.register.TestAspect;
import com.mutistic.annotation.register.TestRepositoryDao;

/**
 * 通过加载 @ComponentScan Class 实现bean的注册 
 */
public class AnnotationMain {
	public static void main(String[] args) {
		AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AnnotationScan.class);
		CommonConstant.printPref("无任何注解 bean", ctx.getBean(TestAnnotationBean.class));
		CommonConstant.printPref("@Configuration 注解 bean", ctx.getBean(IDConfig.class));
		CommonConstant.printPref("@Repository bean", ctx.getBean(TestRepositoryDao.class));
		CommonConstant.printPref("@Aspect 注解 bean", ctx.getBean(TestAspect.class));
	}
}
```

AnnotationScan.java

```Java
package com.mutistic.annotation;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;

import com.mutistic.annotation.beans.AnnotationBeansConfig;
import com.mutistic.annotation.id.IDConfig;
import com.mutistic.annotation.register.TestAspect;
import com.mutistic.annotation.register.TestController;
import com.mutistic.annotation.register.TestRepositoryDao;

/**
 * bean组件扫描 引导@Configuration类
 * 开启组件扫描
 */
@Configuration
// @ComponentScan("com.mutistic.annotation") // @ComponentScan 配置用于@Configuration类的组件扫描指令 可以指定用于定义要扫描的特定包
// @ComponentScan(value = {"com.mutistic.annotation.beans", "com.mutistic.annotation.id"}) // @ComponentScan 通过 value属性或者basePackages属性  可以指定多个需要扫描的包
// @ComponentScan(basePackages="om.mutistic.annotation", ) // 通过 value属性可以指定多个包
@ComponentScan(basePackages="com.mutistic.annotation"
	,includeFilters = @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {TestAspect.class}) // @ComponentScan 通过 includeFilters 属性 可以指定导入bean(类型具体参考 FilterType)
	,excludeFilters = @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {AnnotationBeansConfig.class, IDConfig.class}) // @ComponentScan 通过 excludeFilters 属性 可以指定忽略bean(类型具体参考 FilterType)
) 
public class AnnotationScan { }
```

---
### <a name="a_configuration">二、@Configuration 配置注解</a>
@Configuration [org.springframework.context.annotation.Configuration](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html)

AnnotationConfigWebApplicationContext [org.springframework.web.context.support.AnnotationConfigWebApplicationContext](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/context/support/AnnotationConfigWebApplicationContext.html)

ClassPathXmlApplicationContext [org.springframework.context.support.ClassPathXmlApplicationContext](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html)

2.1、引导@Configuration类：</br>

```
从Spring3.0，@Configuration用于定义配置类，可替换xml配置文件（等同于xml文件中的<beans>标签）。
被注解的类内部包含有一个或多个被@Bean注解的方法，这些方法将会被AnnotationConfigApplicationContext或AnnotationConfigWebApplicationContext类（替代了加载xml配置文件的ClassPathXmlApplicationContext类）进行扫描，并用于构建bean定义，初始化Spring容器。
```

2.2、通过 AnnotationConfigApplicationContext 指定加载</br>

```
1、@Configuration类通常使用任何一个AnnotationConfigApplicationContext或者具有web-capable的变体，来引导 AnnotationConfigWebApplicationContext。
2、@Configuration不可以是final类型：否则会报 org.springframework.beans.factory.parsing.BeanDefinitionParsingException 异常。
3、@Configuration不可以是匿名类（包含匿名内部类）。
4、@configuration不可以是非静态内部类：否则会报错 org.springframework.beans.factory.UnsatisfiedDependencyException 异常。
```
```Java
package com.mutistic.annotation;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class AnnotationMain {
	public static void main(String[] args) {
		// 通过无参构造器创建context
		AnnotationConfigApplicationContext context_mode1 = new AnnotationConfigApplicationContext();
		context_mode1.register(AnnotationConfig.class);
		context_mode1.refresh();
		context_mode1.close();
		// 通过有参构造器创建context
		AnnotationConfigApplicationContext context_mode2 = new AnnotationConfigApplicationContext(AnnotationConfig.class);
		context_mode2.close();
	}
}

```

2.3、通过 Spring <beans> XML 配置</br>
ConfigurationClassPostProcessor [org.springframework.context.annotation.ConfigurationClassPostProcessor](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/ConfigurationClassPostProcessor.html)
  
```
1、作为@Configuration直接针对bean 注册类 的替代方法AnnotationConfigApplicationContext，@Configuration类可以<bean>在Spring XML文件中声明为正常的定义。
2、<context:annotation-config/>：为了启用ConfigurationClassPostProcessor和其他注释相关的后置处理器，需要这些后置处理器便于处理@Configuration类
```
```xml
<beans>
  <context:annotation-config/>
  <bean class="com.mutisitic.MyConfig"/>
</beans>
```

---
### <a id="a_bean">三、@Bean bean注解</a>
@Bean [org.springframework.context.annotation.Bean](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html)

@Profile [org.springframework.context.annotation.Profile](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/Profile.html)

@Scope [org.springframework.context.annotation.Scope](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/Scope.html)

@Primary [org.springframework.context.annotation.Primary](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/Primary.html)

@Qualifier [org.springframework.beans.factory.annotation.Qualifier](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/annotation/Qualifier.html)

3.1、指示一个方法产生一个由Spring容器管理的bean：</br>

```
1、bean名称的默认策略是使用@Bean方法的名称。
2、使用该 name属性（或其别名value）。name接受一个字符串数组，允许为一个bean提供多个名称（即主bean名称加上一个或多个别名）。
3、@Bean注释不提供配置文件，范围，懒惰，依赖或主要的属性。相反，它应与结合使用 @Scope，@Lazy，@DependsOn， @Primary，@Qualifier注解来声明这些语义。
4、@Configuration在这种模式下，bean的类和他们的工厂方法不能被标记为final或private。
```

AnnotationConfig.java

```Java
package com.mutistic.annotation;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;

@Configuration
public class AnnotationConfig {
	/**
	 * 通过@Bean直接创建 bean
	 * @return RunnableFactory Bean
	 */
	@Bean(name = "annotationTestBean") // 指定 bean 的具体名称
  	@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE) // 指定bean 的作用域范围：单例，非单例，request，session
	public AnnotationTestBean createrAnnotationTestBean() {
		return new AnnotationTestBean();
	}
	
	/**
	 * 通过 FactoryBean<T>接口实现类 创建工厂 bean（name可以指定多个，默认为方法名）
	 * @return RunnableFactory Bean
	 */
	@Bean({"createrRunnableFactory", "runnableFactory"}) // 指定bean 多个名称
	public RunnableFactory createrRunnableFactory() {
		return new RunnableFactory();
	}
	
	/**
	 * 通过简单工厂类 创建工厂 bean 
	 * @return FocusBeanFactory Bean
	 */
	@Bean // 指定一个bean 其name默认为方法名
	@Primary  // 声明同类型bean为其主bean
	public FocusBeanFactory focusBeanFactory() {
		return new FocusBeanFactory();
	}
	
	/**
	 * 通过简单工厂类 创建工厂 bean 
	 * @return FocusBeanFactory Bean
	 */
	@Bean
	@Profile("dev") // 指定当一个或多个指定的配置文件处于活动状态时，组件可以注册
	public FocusBeanFactory craeterFocusBeanFactory() {
		return new FocusBeanFactory();
	}
	
	/**
	 * 通过简单工厂类 创建 实体bean 
	 * @param focusBeanFactory 检索后自动注入(spirng中无FocusBeanFactory：则无法自动注入。创建多个同类型bean可以使用 @Qualifier 指定具体一个bean)
	 * @return Focus Bean
	 */
	@Bean
	public Focus focus(/*@Qualifier("focusBeanFactory")*/ FocusBeanFactory focusBeanFactory) {
		return focusBeanFactory.createrFocus();
	}
}
```

RunnableFactory.java

```Java
package com.mutistic.annotation;
import org.springframework.beans.factory.FactoryBean;

/**
 * 通过 FactoryBean<T>接口实现类 创建工厂 RunnableFactory bean
 * 使用工厂类实现 org.springframework.beans.factory.FactoryBean<T> 创建bean实例，重写 getObject(); getObjectType(); isSingleton();
 */
public class RunnableFactory implements FactoryBean<Runnable> {

	/**
	 * 获取 FactoryBean 创建的bean实例
	 * @return Runnable 实例bean
	 * @see org.springframework.beans.factory.FactoryBean#getObject()
	 */
	@Override
	public Runnable getObject() throws Exception {
		return () -> {}; // 简单创建一个接口实例
	}

	/**
	 * 获取创建实例的类型
	 * @return Runnable.class
	 * @see org.springframework.beans.factory.FactoryBean#getObjectType()
	 */
	@Override
	public Class<?> getObjectType() {
		return Runnable.class;
	}

	/**
	 * 是否是单例模式(true：单例)(false：非单例)
	 * @return 是否是单例模式
	 * @see org.springframework.beans.factory.FactoryBean#isSingleton()
	 */
	@Override
	public boolean isSingleton() {
		return true;
	}

}

```

FocusBeanFactory.java

```Java
package com.mutistic.annotation;
/**
 * Focus 简单工厂类
 * 工厂类实现创建具体bean方法，config使用 @Bean 注解实现工厂类bean的创建方法，confgi使用 @Bean 注解实现具体bean的创建方法，入参为 工厂类（会自动搜索工厂类bean）。
 */
public class FocusBeanFactory {
	public Focus createrFocus() { return new Focus(); }
}
```

---
### <a id="a_getBean">四、spring 获取bean的方式</a>
4.1、通过@Bean注解的方法名获取bean

```Java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AnnotationConfig.class);
TestAnnotationBean testBean_1 = (TestAnnotationBean)context.getBean("createrTestAnnotationBean");
TestAnnotationBean testBean_2 = context.getBean("createrTestAnnotationBean", TestAnnotationBean.class);
```

4.2、通过name属性名称获取bean（支持class）

```Java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AnnotationConfig.class);
TestAnnotationBean testBean_1 = (TestAnnotationBean)context.getBean("testAnnotationBean");
TestAnnotationBean testBean_2 = context.getBean("testAnnotationBean", TestAnnotationBean.class);
```

4.4、通过 &+方法名获取bean工厂

```Java
TestAnnotationBean testBean = new AnnotationConfigApplicationContext(AnnotationConfig.class).getBean(TestAnnotationBean.class);
```

4.3、通过 &+方法名获取bean工厂

```Java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AnnotationConfig.class);
RunnableFactory factory_1 = context.getBean(RunnableFactory.class);
RunnableFactory factory_2 = context.getBean("&createrRunnableFactory"); // &：org.springframework.beans.factory.BeanFactory.FACTORY_BEAN_PREFIX
```

4.5、通过方法名获取工厂模式创建的具体bean

```Java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AnnotationConfig.class);
Runnable runnable = (Runnable)context.getBean("createrRunnableFactory");
```

4.6、获取同class所有bean

```Java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AnnotationConfig.class);
Map<String, TestAnnotationBean> testBeanMap = context.getBeansOfType(TestAnnotationBean.class);
```

---
### <a id="a_initial">五、指定 bean的 initial（初始化） 和 destroy（销毁） 方法</a>
InitializingBean [org.springframework.beans.factory.InitializingBean](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/InitializingBean.html)

DisposableBean [org.springframework.beans.factory.DisposableBean](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/DisposableBean.html)

@PostConstruct [javax.annotation.PostConstruct](https://docs.oracle.com/javase/10/docs/api/javax/annotation/PostConstruct.html)

@PreDestroy [javax.annotation.PreDestroy](https://docs.oracle.com/javase/10/docs/api/javax/annotation/PreDestroy.html)

5.1、通过InitializingBean和DisposableBean接口方式实现：</br>

```Java
package com.mutistic.annotation.id;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;

/**
 * 使用实现接口方式实现 BeanFactory 的初始化 和销毁 动作
 * InitializingBean.afterPropertiesSet 指定初始化方法。DisposableBean.destroy 指定销毁方法
 */
public class IDByInterface implements InitializingBean, DisposableBean {
	/**
	 * 在它设置了所有bean属性之后，由BeanFactory调用（并满足了BeanFactoryAware和applicationcontext）。这个方法允许bean实例只执行初始化
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet()
	 */
	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("IDByImplements implements InitializingBean.afterPropertiesSet");
	}

	/**
	 * 由一个BeanFactory调用的销毁单例对象（context.close();资源释放后调用bean消费方法）
	 * @see org.springframework.beans.factory.DisposableBean#destroy()
	 */
	@Override
	public void destroy() throws Exception {
		System.out.println("IDByImplements implements DisposableBean.destroy");		
	}
}
```

5.2、使用@Bean initMethod和destroyMethod 指定具体的方法：</br>

IDConfig.java

```Java
package com.mutistic.annotation.id;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * initial AND destroy 配置类
 */
@Configuration
public class IDConfig {
	@Bean(initMethod = "initial", destroyMethod = "destroy") // 声明bean 指定其 init 和 destroy方法
	public IDByBean idByBean() {
		return new IDByBean();
	}
}
```

IDByBean.java

```Java
package com.mutistic.annotation.id;
import org.springframework.context.annotation.Bean;
import com.mutistic.utils.CommonConstant;

/**
 * 使用@Bean initMethod和destroyMethod 指定具体的方法
 * @Bean#initMethod 指定初始化方法。 @Bean#destroyMethod 指定销毁方法
 */
public class IDByBean {
	/**
	 * 声明initial方法-对指定bean生效
	 */
	public void initial() {
		System.out.println("IDByBean： @Bean initMethod 指定initial");
	}

	/**
	 * 声明destroy方法-对指定bean生效
	 */
	public void destroy() {
		System.out.println("IDByBean： @Bean destroyMethod 指定destroy");		
	}
}
```

5.3、使用JSR-250 @PostConstruct和 @PreDestroy方式指定

```Java
package com.mutistic.annotation.id;
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

/**
 * 使用JSR-250@PostConstruct和@PreDestroy方式指定
 * @PostConstruct 指定 初始化方法。@PreDestroy 指定 销毁方法
 */
public class IDByJSR250 {
	/**
	 * 声明initial方法-对指定bean生效
	 */
	@PostConstruct // 指定 初始化方法
	public void initial() {
		System.out.println("IDByJSR25 @PostConstruct 指定initial");
	}

	/**
	 * 声明destroy方法-对指定bean生效
	 */
	@PreDestroy // 指定 销毁方法
	public void destroy() {
		System.out.println("IDByJSR250 @PreDestroy 指定 destroy");		
	}
}
```

---
### <a id="a_component">六、使用@Component、@Repository、@Service、@Controller、@Aspect 等方式注册bean</a>
6.1、@Component [org.springframework.stereotype.Component](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/Component.html)<br/>
一般用在没有明确的角色的bean可以用。@Component注解上不支持指定initial和destroy方法<br/>

```Java
package com.mutistic.annotation.register;
import javax.annotation.Resource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Component;
import com.mutistic.utils.CommonConstant;

/**
 * 使用  @Component 注解声明一个bean
 * 一般用在没有明确的角色的bean可以用。@Component注解上不支持指定initial和destroy方法
 */
@Component // 声明一个bean。bean名称默认为类名（首字母小写），value属性值指定其bean名称（不支持多个），其中value可以省略。
//@Component("testComponentBean")
//@Component(value = "myTestComponentBean") 
public class TestComponentBean {
	@Autowired // 使用 spring  @Autowired 自动注入bean
//	@Resource  // 使用 JSR-250 @Resource:javax.annotation.Resource 自动注入bean
	private ApplicationContext applicationContext;
	public void show() {
		System.out.println("TestComponentBean 使用 @Autowried自动注入的ApplicationConext："+ applicationContext.getClass());
	}
}
```

6.2、@Repository [org.springframework.stereotype.Repository](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/Repository.html)<br/>
一般用在数据访问层，也可以将这个注解应用到DAO类中。@Repository注解上不支持指定initial和destroy方法<br/>

```Java
package com.mutistic.annotation.register;
import org.springframework.stereotype.Repository;

/**
 * 使用  @Repository 注解声明一个(dao)bean
 * 一般用在数据访问层，也可以将这个注解应用到DAO类中。@Repository注解上不支持指定initial和destroy方法
 */
@Repository
//@Repository("testRepositoryDao")
//@Repository(value = "myTestRepositoryDao") 
public class TestRepositoryDao { }
```

6.3、@Service [org.springframework.stereotype.Service](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/Service.html)<br/>
一般用在业务逻辑层。@Service注解上不支持指定initial和destroy方法<br/>

```Java
package com.mutistic.annotation.register;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Service;

/**
 * 使用  @Service 注解声明一个(service)bean
 * 一般用在业务逻辑层。@Service注解上不支持指定initial和destroy方法
 */
@Service // 声明一个bean。bean名称默认为类名（首字母小写），value属性值指定其bean名称（不支持多个），其中value可以省略。
//@Service("testService")
//@Service(value = "myTestService") 
public class TestService {
	// 使用 @Autowired 自动注入 bean。	使用 @Qualifier 指定具体一个bean
	@Autowired
//	@Qualifier("testRepositoryDao") // 存在多个同类的bean，可以用使用@Qualifier指定具体一个bean。或已有@Primary声明的主bean
	private TestRepositoryDao testRepositoryDao;

	@Override
	public String toString() {
		return "TestService [testRepositoryDao=" + testRepositoryDao + "]";
	}
}
```

6.4、@Controller [org.springframework.stereotype.Controller](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/Controller.html)<br/>
一般用在数据展示层。@Controller注解上不支持指定initial和destroy方法<br/>

```Java
package com.mutistic.annotation.register;
import javax.annotation.Resource;
import javax.inject.Inject;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Controller;

/**
 * 使用  @Controller 注解声明一个(controller)bean
 * 一般用在数据展示层。@Controller注解上不支持指定initial和destroy方法
 */
@Controller // 声明一个bean。bean名称默认为类名（首字母小写），value属性值指定其bean名称（不支持多个），其中value可以省略。
//@Controller("testController")
//@Controller(value = "myTestController") 
public class TestController {

	// 使用 JSR-250 @Resource 自动注入 bean
	@Resource
	private TestService TestService;

	// 使用 JSR-330 @Inject 自动注入 bean。使用 @Qualifier 指定具体一个bean
	@Inject
	@Qualifier("testRepositoryDao")
	private TestRepositoryDao testRepositoryDao;

	@Override
	public String toString() {
		return "TestController [TestService=" + TestService + ", testRepositoryDao=" + testRepositoryDao + "]";
	}
}
```

JSR-330（javax.inject） pom依赖：

```xml
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

6.5、@Aspect [org.aspectj.lang.annotation.Aspect](https://www.eclipse.org/aspectj/doc/released/aspectj5rt-api/org/aspectj/lang/annotation/Aspect.html)<br/>
把当前类标识为一个切面供容器读取。@Aspect注解上不支持指定initial和destroy方法<br/>

```Java
package com.mutistic.annotation.register;
import org.aspectj.lang.annotation.Aspect;

/**
 * 使用  @Aspect 注解声明一个bean
 * 把当前类标识为一个切面供容器读取。@Aspect注解上不支持指定initial和destroy方法
 */
@Aspect
//@Aspect("testComponentBean") //声明一个bean。bean名称默认为类名（首字母小写），value属性值指定其bean名称（不支持多个），其中value可以省略。
//@Aspect(value = "myTestComponentBean") 
public class TestAspect { }
```

### <a id="a_componentScan">七、@ComponentScan 扫描注解</a>
@ComponentScan [org.springframework.context.annotation.ComponentScan](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/ComponentScan.html)

@Filter [org.springframework.context.annotation.ComponentScan$Filter](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/ComponentScan.Filter.html)

FilterType [org.springframework.context.annotation.FilterType](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/FilterType.html)

TypeFilter [org.springframework.core.type.filter.TypeFilter](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/core/type/filter/TypeFilter.html)

AnnotationTypeFilter [org.springframework.core.type.filter.AnnotationTypeFilter](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/core/type/filter/AnnotationTypeFilter.html)

```
1、配置用于@ Configuration类的组件扫描指令。提供与Spring XML <context:component-scan>元素并行的支持。
2、可以指定basePackageClasses()或basePackages()（或其别名 value()）来定义要扫描的特定软件包。如果未定义特定的包，则将从声明此批注的类的包中进行扫描。
4、请注意，该<context:component-scan>元素有一个 annotation-config属性; 然而，这个注解不是。这是因为在几乎所有使用的情况下@ComponentScan，@Autowired都会假设默认注释配置处理。
而且，在使用时AnnotationConfigApplicationContext，注释配置处理器总是被注册，这意味着任何试图在该@ComponentScan级别禁用它们的尝试都 将被忽略
```

7.1、@ComponentScan 常用属性说明：

```
1、basePackages()：用于指定要扫描注释组件的软件包的类型安全替代方法。如果不需要其他属性，则允许更简洁的注释声明即省略basePackages。value() 是该属性的别名（并且与之互斥）
2、includeFilters()：公共抽象  ComponentScan.Filter[]、指定哪些类型有资格进行组件扫描。
3、excludeFilters()：公共抽象  ComponentScan.Filter[]、指定哪些类型不符合组件扫描条件。
4、lazyInit：指定扫描的bean是否应注册以进行延迟初始化。默认是false; 根据true需要切换此项。
```

7.2、FilterType 枚举值说明：

```
1、ANNOTATION：筛选标记有给定注释的候选项。
2、ASSIGNABLE_TYPE：筛选可分配给定类型的候选项（常用）。
3、ASPECTJ：筛选与指定的AspectJ类型模式表达式匹配的候选项。
4、REGEX：过滤与给定的正则表达式模式匹配的候选项。<br/>
5、CUSTOM：使用给定的自定义TypeFilter实现过滤候选项 。
```

7.3、@ComponentScan 使用示例：

```Java
package com.mutistic.annotation;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;

import com.mutistic.annotation.beans.AnnotationBeansConfig;
import com.mutistic.annotation.id.IDConfig;
import com.mutistic.annotation.register.TestAspect;
import com.mutistic.annotation.register.TestController;
import com.mutistic.annotation.register.TestRepositoryDao;

/**
 * bean组件扫描 引导@Configuration类
 * 开启组件扫描
 */
@Configuration
// @ComponentScan("com.mutistic.annotation") // @ComponentScan 配置用于 @Configuration类的组件扫描指令 可以指定用于定义要扫描的特定包
// @ComponentScan(value = {"com.mutistic.annotation.beans", "com.mutistic.annotation.id"}) // @ComponentScan 通过 value属性或者basePackages属性  可以指定多个需要扫描的包
@ComponentScan(basePackages="com.mutistic.annotation"
	,includeFilters = @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {TestAspect.class}) // @ComponentScan 通过 includeFilters 属性 可以指定导入bean(类型具体参考 FilterType)
	,excludeFilters = @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {AnnotationBeansConfig.class, IDConfig.class}) // @ComponentScan 通过 excludeFilters 属性 可以指定忽略bean(类型具体参考 FilterType)
) 
public class AnnotationScan { }
```

### <a id="a_beanPostProcessor">八、BeanPostProcessor bean后置处理器</a>
BeanPostProcessor：[org.springframework.beans.factory.config.BeanPostProcessor](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html)

```
1、允许自定义修改新bean实例的工厂挂钩，例如检查标记接口或用代理包装它们。
2、ApplicationContexts可以在其bean定义中自动检测BeanPostProcessor bean，并将它们应用于随后创建的任何bean。工厂允许对后处理程序进行程序注册，适用于通过该工厂创建的所有bean。
3、通常，通过标记接口等来填充bean的后置处理器将实现postProcessBeforeInitialization(java.lang.Object, java.lang.String)，
4、而用代理来包装bean的后置处理器通常将实现postProcessAfterInitialization(java.lang.Object, java.lang.String)
```

8.1、方法说明：<br/>
8.1.1、postProcessAfterInitialization(java.lang.Object bean, java.lang.String beanName)

```
调用顺序：bean 属性设置（依赖配置） > postProcessAfterInitialization > bean init （即判定：init也是在bean 属性设置之后执行）
1、在任何bean初始化回调（如InitializingBean's afterPropertiesSet 或自定义init方法）之前，  将此BeanPostProcessor应用于给定的新bean实例。
2、这个bean已经被填充了属性值。返回的bean实例可能是原始包装，或代理对象。	
3、默认实现返回给定的bean本身。不可返回null（如返回null导致后续处理时丢失传入的bean，且不会调用后续的BeanPostProcessors）。
```

8.1.2、postProcessBeforeInitialization(java.lang.Object bean, java.lang.String beanName)

```
调用顺序：bean init  > postProcessBeforeInitialization
1、在 Bean初始化回调（如InitializingBean's afterPropertiesSet 或自定义init方法）之后，将此BeanPostProcessor应用于给定的新bean实例。
2、这个bean已经被填充了属性值。返回的bean实例可能是原始包装。
3、在FactoryBean的情况下，FactoryBean实例和由FactoryBean创建的对象（从Spring 2.0开始）都将调用此回调函数。
4、后处理器可以通过相应的bean instanceof FactoryBean检查决定是应用于FactoryBean还是应用于已创建的对象或两者。
5、InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation(java.lang.Class<?>, java.lang.String)与所有其他BeanPostProcessor回调相比，
	此回调也将在由方法触发的短路之后 调用。
6、默认实现返回给定的bean本身。不可返回null（如返回null导致后续处理时丢失传入的bean，且不会调用后续的BeanPostProcessors）。
```

### <a id="a_applicationContextAware">九、Aware 之 ApplicationContextAware</a>
9.1、Aware：[org.springframework.beans.factory.Aware](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/Aware.html)<br/>

```
1、公共接口Aware
2、标记超级接口，表示一个bean有资格通过一个回调式方法被Spring容器通知一个特定的框架对象。
3、实际的方法签名由各个子接口确定，但通常应由一个只接受一个参数的void返回方法组成。
4、请注意，仅实现不Aware提供默认功能。相反，处理必须明确完成，例如在 BeanPostProcessor。
5、参阅ApplicationContextAwareProcessor 和AbstractAutowireCapableBeanFactory 用于处理的示例 Aware接口回调
```

9.2、ApplicationContextAware：[org.springframework.context.ApplicationContextAware](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/ApplicationContextAware.html)<br/>
方法说明：<br/>
setApplicationContext(ApplicationContext applicationContext)

```
1、设置该对象运行的ApplicationContext。通常，这个调用将用于初始化对象。
2、在正常bean属性的设置之后但在init回调（如InitializingBean.afterPropertiesSet() 自定义init方法）之前调用。
3、之后被调用ResourceLoaderAware.setResourceLoader(org.springframework.core.io.ResourceLoader)， ApplicationEventPublisherAware.setApplicationEventPublisher(org.springframework.context.ApplicationEventPublisher)
并且 MessageSourceAware，如果适用
```

9.3、ApplicationContext：[org.springframework.context.ApplicationContext](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html)<br/>
9.3.1、通过@Autowired或@Resource注解自动注入 ApplicationContext

```Java
package com.mutistic.annotation.processor;
import javax.annotation.Resource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Component;
/**
 * 通过@Autowired或@Resource注解自动注入
 */
@Component
public class TestApplicationContextAware {
	@Autowired // 使用 spring  @Autowired 自动注入bean
//	@Resource  // 使用 JSR-250 @Resource:javax.annotation.Resource 自动注入bean
	private ApplicationContext applicationContextByAuto;
}
```

9.3.2、通过 ApplicationContextAware.setApplicationContext()接口方法注入 ApplicationContext

```Java
package com.mutistic.annotation.processor;
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;
/**
 * 通过 ApplicationContextAware.setApplicationContext()接口方法注入 
 */
@Component
public class TestApplicationContextAware implements ApplicationContextAware {
	private ApplicationContext applicationContextByImpl;
	
	/**
	 * 通过 ApplicationContextAware.setApplicationContext()接口方法注入  
	 * @param applicationContext
	 * @throws BeansException
	 * @see org.springframework.context.ApplicationContextAware#setApplicationContext(org.springframework.context.ApplicationContext)
	 */
	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		/**
		 * spring在bean初始化之后判断是否是 实现了ApplicationContextAware接口，如是则通过 setApplicationContext 方法将 ApplicationContext 注入到bean中。
		 */
		this.applicationContextByImpl = applicationContext;
	}
}
```

9.3.3、通过spring4.3的新特性 构造函数 自动注入 ApplicationContext

```Java
package com.mutistic.annotation.processor;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Component;
/**
 * 通过spring4.3的新特性 构造函数 自动注入
 */
@Component
public class TestApplicationContextAware {
	private ApplicationContext applicationContextByCtor;
	/**
	 * 3、通过spring4.3的新特性 构造函数 自动注入
	 */
	public TestApplicationContextAware(ApplicationContext applicationContextByCtor) {
	// public TestRegisterContext(ApplicationContext applicationContextByCtor, TestController testController) {
		/**
		 * 通过构造函数自动注入的局限性：
		 * 1、本类只能有一个构造函数，存在多个的话则会调去默认构造函数，则会导致 ApplicationContext无法实现自动注入。
		 * 2、构造函数参数个数无要求，但是参数要求spring容器中有对应bean，bean的可以是一个或多个。
		 */
		this.applicationContextByCtor = applicationContextByCtor;
	}
}
```

### <a id="a_beanFactoryPostProcessor">十、BeanFactoryPostProcessor 上下文的基础工厂Bean后置处理器</a>
10.1、BeanFactoryPostProcessor：[org.springframework.beans.factory.config.BeanFactoryPostProcessor](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html)<br/>

```
1、公共接口BeanFactoryPostProcessor。只会执行一次。
2、允许自定义修改应用程序上下文的bean定义，调整上下文的基础bean工厂的bean属性值。
3、应用程序上下文可以在其bean定义中自动检测BeanFactoryPostProcessor bean，并在创建任何其他bean之前应用它们。
4、对于定制配置文件非常有用，这些文件针对系统管理员，覆盖应用程序上下文中配置的bean属性。
5、请参阅PropertyResourceConfigurer及其具体实现，了解解决此类配置需求的开箱即用解决方案。
6、BeanFactoryPostProcessor可能与bean定义交互并修改，但永远不会bean实例。这样做可能会导致bean过早实例化，违反容器并导致意想不到的副作用。如果需要bean实例交互，请考虑实现 BeanPostProcessor
```

TestBeanFactoryPostProcessor.class：

```Java
package com.mutistic.annotation.factory;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.stereotype.Component;
import com.mutistic.utils.CommonConstant;

/**
 * 实现 上下文的基础bean BeanFactoryPostProcessor 接口
 * 实现 BeanFactoryPostProcessor.postProcessBeanFactory() 方法
 */
@Component
public class TestBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
	/**
	 * bean初始化后，修改应用程序上下文的内部bean工厂
	 * @param beanFactory
	 * @throws BeansException
	 * @see org.springframework.beans.factory.config.BeanFactoryPostProcessor#postProcessBeanFactory(org.springframework.beans.factory.config.ConfigurableListableBeanFactory)
	 */
	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		CommonConstant.printPref("自定义实现类重写 BeanFactoryPostProcessor.postProcessBeanFactory", beanFactory);
		
		CommonConstant.printPref("自定义实现类 从 ConfigurableListableBeanFactory 中获取Spring bean的数量", beanFactory.getBeanDefinitionCount());
		for(String name : beanFactory.getBeanDefinitionNames()) {
			CommonConstant.printPref("自定义实现类 从 ConfigurableListableBeanFactory 中获取Spring bean的name", name);
		}
	}
}
```

10.2、BeanFactoryPostProcessor 执行顺序

```
1、BeanDefinitionRegistryPostProcessor.postProcessBeanDefinitionRegistry()
2、BeanDefinitionRegistryPostProcessor.postProcessBeanFactory()
3、BeanFactoryPostProcessor.postProcessBeanFactory() 
4、Bean 的依赖配置
5、BeanPostProcessor.postProcessBeforeInitialization()
6、Bean 的初始化方法 
7、BeanPostProcessor.postProcessAfterInitialization()
```

10.3、使用BeanDefinitionRegistryPostProcessor的动态注册bean（BeanFactoryPostProcessor的子类）
BeanDefinitionRegistryPostProcessor[org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/support/BeanDefinitionRegistryPostProcessor.html)<br/>

ConfigurableListableBeanFactory[org.springframework.beans.factory.config.ConfigurableListableBeanFactory](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/config/ConfigurableListableBeanFactory.html)<br/>

BeanDefinitionRegistry[org.springframework.beans.factory.support.BeanDefinitionRegistry](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/support/BeanDefinitionRegistry.html)<br/>

BeanDefinitionBuilder[org.springframework.beans.factory.support.BeanDefinitionBuilder](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/support/BeanDefinitionBuilder.html)<br/>


```
1、扩展到标准BeanFactoryPostProcessor SPI，允许在正常的BeanFactoryPostProcessor检测开始之前注册更多的bean定义。
2、特别是，BeanDefinitionRegistryPostProcessor可以注册更多的bean定义，然后定义BeanFactoryPostProcessor实例。
3、BeanDefinitionRegistryPostProcessor 可以拿到ConfigurableListableBeanFactory，BeanDefinitionRegistry对象。
	BeanDefinitionRegistry 可以动态注册bean
	BeanDefinitionBuilder 获取Bean的引用(class)，注入属性，后用于BeanDefinitionRegistry
```

TestBeanDefinitionRegistryPostProcessor.class：

```Java
package com.mutistic.annotation.factory;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor;
import org.springframework.stereotype.Component;
import com.mutistic.utils.CommonConstant;

/**
 * 演示 BeanDefinitionRegistryPostProcessor 的自定义实现类 的动态注册bean
 * BeanDefinitionRegistryPostProcessor 继承至 BeanFactoryPostProcessor
 */
@Component
public class TestBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {
	/**
	 * bean初始化后，修改应用程序上下文的内部bean工厂
	 * @param beanFactory
	 * @throws BeansException
	 * @see org.springframework.beans.factory.config.BeanFactoryPostProcessor#postProcessBeanFactory(org.springframework.beans.factory.config.ConfigurableListableBeanFactory)
	 */
	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		CommonConstant.printPref("自定义实现类重写 BeanFactoryPostProcessor.postProcessBeanFactory", beanFactory);
		
		CommonConstant.printPref("自定义实现类 从 ConfigurableListableBeanFactory 中获取Spring bean的数量", beanFactory.getBeanDefinitionCount());
		for(String name : beanFactory.getBeanDefinitionNames()) {
			CommonConstant.printPref("自定义实现类 从 ConfigurableListableBeanFactory 中获取Spring bean的name", name);
		}
	}

	/**
	 * 标准初始化后，修改应用程序上下文的内部bean定义注册表
	 * @param registry
	 * @throws BeansException
	 * @see org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor#postProcessBeanDefinitionRegistry(org.springframework.beans.factory.support.BeanDefinitionRegistry)
	 */
	@Override
	public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
		CommonConstant.printPref("自定义实现类重写 BeanDefinitionRegistryPostProcessor.postProcessBeanDefinitionRegistry", registry);
		for (int i = 0; i < 10; i++) {
			// 通过 BeanDefinitionBuilder 获取Bean的引用(class)，注入属性
			BeanDefinitionBuilder bdb = BeanDefinitionBuilder.rootBeanDefinition(TestRegistryBean.class);
			bdb.addPropertyValue("property", "注入属性值："+i);
			registry.registerBeanDefinition("beanName"+i, bdb.getBeanDefinition());
			CommonConstant.printPref("通过 BeanDefinitionRegistry和BeanDefinitionBuilder实现动态注册bean", registry);
		}
	}
}
```

10.4、通过AnnotationConfigApplicationContext.registerBeanDefinition() 直接动态注册bean

```Java
package com.mutistic.annotation.factory;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import com.mutistic.utils.CommonConstant;
/**
 * 通过AnnotationConfigApplicationContext.registerBeanDefinition() 直接动态注册bean
 */
public class MainByFactoryProcessor {
	public static void main(String[] args) {
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(TestRegistryBean.class);
		
		for (int i = 0; i < 10; i++) {
			// 通过 BeanDefinitionBuilder 获取Bean的引用(class) (可不注入属性)
			context.registerBeanDefinition("beanName"+i, BeanDefinitionBuilder.rootBeanDefinition(TestRegistryBean.class).getBeanDefinition());
			CommonConstant.printPref("通过 AnnotationConfigApplicationContext.registerBeanDefinition() 和BeanDefinitionBuilder实现动态注册bean：", i);
		}
		CommonConstant.println();
		context.getBeansOfType(TestRegistryBean.class).values().forEach(System.out::println); // java8流模式打印输入
		
		context.close();
	}
}
```


---
<a id="a_down"></a>  
<a href="#a_top">Top</a> 
<a href="#a_catalogue">Catalogue</a>  

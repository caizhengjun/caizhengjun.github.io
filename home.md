## Spring 注解开发

## 1. @Bean

```java
public class MainTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        Person person = (Person) context.getBean("person");
        System.out.println("person = " + person);
    }
}

```

```java
public class Person {

    private String name;
    private Integer age;
}
```

## 2. @Configuration

```java
@Configuration
public class MainConfig {

    @Bean
    public Person person() {
        return new Person("lisi", 27);
    }
}
```

```java

public class MainTest {
    public static void main(String[] args) {
//        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
////        Person person = (Person) context.getBean("person");
////        System.out.println("person = " + person);

        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        Person person = (Person) applicationContext.getBean("person");
        System.out.println("person = " + person);
    }
}
```

## 3. @ComponentScan

```java
@Configuration
//@ComponentScan(value = {"com.nangjing"})
@ComponentScan(value = "com.nangjing",excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = Controller.class)
})
public class MainConfig {

    @Bean
    public Person person() {
        return new Person("lisi", 27);
    }
}
```

## 4. 自定义过滤规则

```java
public enum FilterType {

	/**
	 * Filter candidates marked with a given annotation.
	 * @see org.springframework.core.type.filter.AnnotationTypeFilter
	 */
	ANNOTATION,

	/**
	 * Filter candidates assignable to a given type.
	 * @see org.springframework.core.type.filter.AssignableTypeFilter
	 */
	ASSIGNABLE_TYPE,

	/**
	 * Filter candidates matching a given AspectJ type pattern expression.
	 * @see org.springframework.core.type.filter.AspectJTypeFilter
	 */
	ASPECTJ,

	/**
	 * Filter candidates matching a given regex pattern.
	 * @see org.springframework.core.type.filter.RegexPatternTypeFilter
	 */
	REGEX,

	/** Filter candidates using a given custom
	 * {@link org.springframework.core.type.filter.TypeFilter} implementation.
	 */
	CUSTOM

}
```

### 4.1 实现TypeFilter接口

```java
public class MyTypeFilter implements TypeFilter {
    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {

        // 扫描类的类信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        // 类路径
        Resource resource = metadataReader.getResource();

        String className = classMetadata.getClassName();

        System.out.println("className = " + className);

        if (className.contains("er")) {
            return true;
        }

        return false;
    }
}

```

## 5. 组件作用域

```java
/**
	 * Specifies the name of the scope to use for the annotated component/bean.
	 * <p>Defaults to an empty string ({@code ""}) which implies
	 * {@link ConfigurableBeanFactory#SCOPE_SINGLETON SCOPE_SINGLETON}.
	 * @since 4.2
	 * @see ConfigurableBeanFactory#SCOPE_PROTOTYPE
	 * @see ConfigurableBeanFactory#SCOPE_SINGLETON
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_SESSION
	 * @see #value
	 */
```

## 6. 懒加载

```java

@Configuration
public class MainConfig2 {

  // prototype
  // singleton
  // request
  // session
  @Bean
  @Scope
  @Lazy
  public Person person() {
    System.out.println("给容器添加person");
    return new Person("蔡正峻", 27);
  }
}
```

## 7. @Condition 按条件注册

```java
/**
 * 判断操作系统
 */
public class LinuxConditional implements Condition {
  @Override
  public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata){
    Environment environment = context.getEnvironment();
    String property = environment.getProperty("os.name");
    if (property.contains("Mac OS X")) {
      return true;
    }
    return false;
  }
}
```

```java
/**
 * 判断操作系统
 */
public class WindowsConditional implements Condition {
  @Override
  public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata){
    Environment environment = context.getEnvironment();
    String property = environment.getProperty("os.name");
    if (property.contains("Windows")) {
      return true;
    }
    return false;
  }
}
```

## 8. @Import 快速导入

```java
@Configuration
@Import(Color.class)
public class MainConfig3 {


    @Conditional(WindowsConditional.class)
    @Bean("Bill")
    public Person person1() {
        return new Person("Bill", 62);
    }


    @Conditional(LinuxConditional.class)
    @Bean("Linux")
    public Person person2() {
        return new Person("Linux", 48);
    }
}
```

![image-20200411232340403](/Users/caizhengjun/Library/Application Support/typora-user-images/image-20200411232340403.png)

## 9. ImportSelector

```java
/**
 * 自定义逻辑返回导入组件
 */
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {

        return new String[]{"com.nangjing.bean.Color","com.nangjing.bean.Red"};
    }
}

```

## 10. ImportBeanDefinitionRegistrar

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

        registry.registerBeanDefinition("yellow",new RootBeanDefinition(Yellow.class));
    }
}

```

## 11. FactoryBean

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

        registry.registerBeanDefinition("yellow",new RootBeanDefinition(Yellow.class));
    }
}
```

```java
    @Test
    public void testImport() {
        ApplicationContext context = new AnnotationConfigApplicationContext(MainConfig3.class);
        String[] names = context.getBeanDefinitionNames();

        for (String name : names) {
            System.out.println(name);
        }
        System.out.println("====>");
        Object colorFactoryBean1 = context.getBean("colorFactoryBean");
        System.out.println(colorFactoryBean1);

        Object colorFactoryBean2 = context.getBean("&colorFactoryBean");
        System.out.println(colorFactoryBean2);
    }
```

![image-20200411235924100](/Users/caizhengjun/Library/Application Support/typora-user-images/image-20200411235924100.png)

## 12. Bean 生命周期

```java
public class Car {
    public Car() {
        System.out.println("car constructor...");
    }
    public void init(){
        System.out.println("car init...");
    }
    public void destroy(){
        System.out.println("car destroy...");
    }
}
```

```java
@Configuration
public class MyConfigOfLifeCycle {

  @Bean(initMethod = "init",destroyMethod = "destroy")
  public Car car() {
    return new Car();
  }
}
```

```java
@Test
public void Bean生命周期() {
  ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MyConfigOfLifeCycle.class);
}
```

```java
@Test
public void Bean生命周期() {
  AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MyConfigOfLifeCycle.class);
  applicationContext.close();
}
```

![image-20200412001020734](/Users/caizhengjun/Library/Application Support/typora-user-images/image-20200412001020734.png)

### 12.1 多实例

```java
@Configuration
public class MyConfigOfLifeCycle {

    @Scope("prototype")
    @Bean(initMethod = "init",destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }
}
```

```java
@Test
public void Bean生命周期() {

  AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MyConfigOfLifeCycle.class);

  applicationContext.getBean("car");

  applicationContext.close();
}
// GC 回收
```

![image-20200412001443911](/Users/caizhengjun/Library/Application Support/typora-user-images/image-20200412001443911.png)

## 13. 生命周期 InitializingBean, DisposableBean

```java
@Component
public class Cat implements InitializingBean, DisposableBean {
    @Override
    public void destroy() throws Exception {
        System.out.println("销毁");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("属性设置");
    }
}
```

```java
@Configuration
@ComponentScan(value = {"com.nangjing.bean"})
public class MyConfigOfLifeCycle {

//    @Bean(initMethod = "init",destroyMethod = "destroy")
//    public Car car() {
//        return new Car();
//    }
}
```

```java
@Test
public void Bean生命周期() {

AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MyConfigOfLifeCycle.class);

//applicationContext.getBean("car");

applicationContext.close();

}
```

## 14. JSR250

```java
@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PostConstruct {
}
```

```java
@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PreDestroy {
}
```

## 15. BeanPostProcessor 后置处理器

```java
public interface BeanPostProcessor {

	/**
	 * Apply this BeanPostProcessor to the given new bean instance <i>before</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 * if {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 */
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

	/**
	 * Apply this BeanPostProcessor to the given new bean instance <i>after</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * <p>In case of a FactoryBean, this callback will be invoked for both the FactoryBean
	 * instance and the objects created by the FactoryBean (as of Spring 2.0). The
	 * post-processor can decide whether to apply to either the FactoryBean or created
	 * objects or both through corresponding {@code bean instanceof FactoryBean} checks.
	 * <p>This callback will also be invoked after a short-circuiting triggered by a
	 * {@link InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation} method,
	 * in contrast to all other BeanPostProcessor callbacks.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 * if {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 * @see org.springframework.beans.factory.FactoryBean
	 */
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
}
```

## 16. BeanPostProcessor 原理

## 17. Spring底层使用 BeanPostProcessor

## 18. @Value 属性赋值

## 19. @PropertySource

```java
@Configuration
@PropertySource(value = {"classpath:/person.properties"})
public class MainConfigOfPropertyValue {

    @Bean
    public Person person() {
        return new Person();
    }

}
```

```java

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Person {

    @Value("caizhengjun")
    private String name;
    @Value("#{20-2}")
    private Integer age;

    @Value("${person.nickname}")
    private String nickName;

    public Person(String bill, int i) {
    }
}
```

## 20. 自动装配 @Autowired @Qualifier @Primary

## 21. @Resource(JSR250) @Inject(JSR330)

## 22. 自动装配 构造器 方法 位置


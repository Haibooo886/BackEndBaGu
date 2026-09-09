# 1. Spring中@Autowired和@Resource注解有什么区别？

Spring框架中，@Autowired和@Resorce都是用来实现依赖注入的注解，有以下区别：
## **来源不同**
@Autowired是Spring框架提供的注解，@Resource是Java EE的JSR-250规范的一部分，由Java本身提供。
## **注入方式**
@Autowired默认是通过类型(beType)进行注入。如果容器中存在多个相同类型的实例，它还可以与@Qualifier注解一起使用，通过指定bean的id来注入特定的实例。@Resource默认是通过名称(byName)进行注入。如果未指定名称，则会尝试通过类型进行匹配。
## **属性**
@Autowired可以不指定任何属性，仅通过类型自动装配。@Resource可以指定一个名为name的属性，该属性表示要注入的bean的名称。

# 2. Bean的生命周期
1. 创建阶段（实例化 Bean）：当使用构造函数或者工厂方法创建Bean对象时，就进入了创建阶段。
2. 属性设置阶段：在Bean对象创建后，通过setter方法设置Bean的各个属性。
3. 初始化阶段：当Bean的属性设置完成后，会触发初始化回调方法，进行一些额外的初始化工作。
-实现了各种 Aware 通知的方法，如 BeanNameAware、BeanFactoryAware、
ApplicationContextAware 的接口方法
-执行 BeanPostProcessor 初始化前置方法
-执行 @PostConstruct 初始化方法，依赖注入操作之后被执行
-执行自己指定的 init-method 方法
-执行 BeanPostProcessor 初始化后置方法
4. 使用阶段：在初始化完成后，Bean对象处于可用状态，可以供应用程序使用。
5. 销毁阶段：当Bean对象不再需要时，会触发销毁回调方法，进行资源释放等清理工作，销毁容器的各种方法，如 @PreDestroy、DisposableBean 接口方法、destroy-method
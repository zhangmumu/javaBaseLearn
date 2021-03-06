###责任链模式
**责任链模式（Chain of Responsibility Pattern）为请求创建了一个接收者对象的链。
这种模式给予请求的类型，对请求的发送者和接收者进行解耦。这种类型的设计模式属于行为型模式。
在这种模式中，通常每个接收者都包含对另一个接收者的引用。如果一个对象不能处理该请求，
那么它会把相同的请求传给下一个接收者，依此类推。[引用w3c对责任链的描述](https://www.runoob.com/design-pattern/chain-of-responsibility-pattern.html)**   
在实际的开发中我们也会遇见，我们需要对外部提供一个接口，中间可能会结果很多非业务的处理方法（日志记录，权限校验，
敏感数据清洗....） 但是对于业务来说是透明的。每个处理器都是独立的，不应该存在耦合关系，才可以让我们随意的去拼接。  
![责任链-01](https://img04.sogoucdn.com/app/a/100520146/ce9ee410725183b6ef51c6699976802b)  
代码大概实现关系依赖关系  
![责任链-02](https://img01.sogoucdn.com/app/a/100520146/8a877941bbec7c59436444d5484b430d)  
首先我们需要定义一个接口RequestPlugin,后面所有的插件需要去实现这个接口就行
```
/**
 * 插件接口定义
 */
public interface RequestPlugin {

    /**
     * 路由
     */
    void interceptor(InterceptorChainWrapper routeChainWrapper);

    /**
     * 是否启用
     */
    boolean enable();
}
```
每个插件处理的功能都是独立的。但是插件之间可能存在排序关系，先加载执行谁后执行谁，比如请求入口应该保留最原始的参数，所以日志的插件一般是放在第一个。
这个时候我们每个插件是不是就要有个排序，所以这里定义了一个注解，通过order的数值大小进行排序。
```
/**
 * 插件注解
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface PluginAnno {

    int order() default Ordered.HIGHEST_PRECEDENCE;

    String name();
}

```
这里我实现了三个插件去构建一个插件的责任链:日志处理插件，解析处理插件，权限校验插件；
```
/**
 * 日志处理
 */
@PluginAnno(order = 1, name = "LogSavePlugin")
public class LogSavePlugin implements RequestPlugin {

    @Override
    public void interceptor(InterceptorChainWrapper routeChainWrapper) {
        System.out.println("日志插件 ： LogSavePlugin");
        routeChainWrapper.interceptor();
    }

    @Override
    public boolean enable() {
        return true;
    }
}


/**
 * 解析处理
 */
@PluginAnno(order = 2, name = "ParseHandlePlugin")
public class ParseHandlePlugin implements RequestPlugin {

    @Override
    public void interceptor(InterceptorChainWrapper routeChainWrapper) {
        System.out.println("解析处理插件 ： ParseHandlePlugin");
        routeChainWrapper.interceptor();
    }

    @Override
    public boolean enable() {
        return false;
    }
}


/**
 * 权限校验
 */
@PluginAnno(order = 3, name = "AuthorCheckPlugin")
public class AuthorCheckPlugin implements RequestPlugin {

    @Override
    public void interceptor(InterceptorChainWrapper routeChainWrapper) {
        System.out.println("权限插件 ： AuthorCheckPlugin");
        routeChainWrapper.interceptor();
    }

    @Override
    public boolean enable() {
        return true;
    }
}

```
注入三个插件，将他们排序。
```
/**
 * 配置插件
 */
@Configuration
public class RequestPluginConfig {

    private List<RequestPlugin> requestPlugins;

    /**
     * 注入相关处理器
     * 对处理器
     */
    public RequestPluginConfig(List<RequestPlugin> requestPlugins) {
        this.requestPlugins = requestPlugins.stream().sorted(Comparator.comparingInt(o -> o.getClass().getAnnotation(PluginAnno.class).order())).collect(Collectors.toList());
    }

    public InterceptorChainWrapper createChainWrapper() {
        return new InterceptorChainWrapper(requestPlugins);
    }
}
```
实际启用插件调用类和测试
```
/**
 * 插件调用链
 */
public class InterceptorChainWrapper {
    private final AtomicInteger atomicInteger = new AtomicInteger(-1);

    private List<RequestPlugin> requestPlugins;

    public InterceptorChainWrapper(List<RequestPlugin> requestPlugins) {
        this.requestPlugins = requestPlugins;
    }

    /**
     * 实际触发
     */
    public void interceptor() {
        if (atomicInteger.incrementAndGet() == requestPlugins.size()) {
            return;
        }
        RequestPlugin plugin = requestPlugins.get(atomicInteger.get());
        if (!plugin.enable()) {
            interceptor();
            return;
        }
        plugin.interceptor(this);
    }
}


/**
* 模拟调用
**/
public class StudyApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(StudyApplication.class, args);
        RequestPluginConfig requestPluginConfig = context.getBean(RequestPluginConfig.class);
        requestPluginConfig.createChainWrapper().interceptor();
    }

}
```
我们看了上面实现方案，我们再来看看实际的框架时怎么实现的，这里来搂下mybatis拦截器,看看它和我们实现的有什么不同。
先看看他们的关系图

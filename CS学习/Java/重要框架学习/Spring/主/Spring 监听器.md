## 一, 是什么
一种特殊的类, 它们帮助开发者监听 web 中特定的事件, 比如 ServletContext, HttpSession, ServletRequest 的创建和销毁；变量的创建、销毁等等。

当Web容器启动后，Spring的监听器会启动监听，监听是否创建ServletContext的对象，如果发生了创建ServletContext对象这个事件 (当web容器启动后一定会生成一个ServletContext对象，所以监听事件一定会发生)，ContextLoaderListener类会实例化并且执行初始化方法，将spring的配置文件中配置的bean注册到Spring容器中

## 二, 观察者模式

观察者模式 一种 行为设计模式 它用于在对象之间建立 一对多 的 依赖关系. 在该模式中, 当一个对象的状态发生变化时, 它会自动通知其他依赖对象 (称为观察者), 使它们能够自动更新

工作原理如下:
1. 主题对象维护一个 `观察者列表`, 并提供方法用于 `添加` 和 `删除` 观察者
2. 当主题的状态发生变化时, 它会`遍历`观察者`列表`, 并 `调用` 每个观察者的 `通知方法`
3. 观察者收到通知后, 根据通知进行相应的更新操作

![[Pasted image 20240529220811.png]]


### 观察者模式Demo
一个观察者模式demo包含一下部分
- 观察者实体
- 主题实体

所以我们先写个观察者接口

```java
import java.util.ArrayList;
import java.util.List;

interface Observer {
    void update();
}

```

创建两个观察者类

```java
// 具体观察者A
class ConcreteObserverA implements Observer {
    @Override
    public void update() {
        System.out.println("ConcreteObserverA收到更新通知");
    }
}

// 具体观察者B
class ConcreteObserverB implements Observer {
    @Override
    public void update() {
        System.out.println("ConcreteObserverB收到更新通知");
    }
}

```

然后定义主题接口

```java
interface Subject {
    void registerObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers();
}
```

构建一个具体主题

```java
// 具体主题
class ConcreteSubject implements Subject {
	// 观察者列表
    private List<Observer> observers = new ArrayList<>();
	// 注册观察者
    @Override
    public void registerObserver(Observer observer) {
        observers.add(observer);
    }
	// 移除观察者
    @Override
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }
	// 调用观察者的通知方法
    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update();
        }
    }
	// 触发
    public void doSomething() {
        System.out.println("主题执行某些操作...");
        notifyObservers(); // 执行操作后通知观察者
    }
}

```

测试代码

```java
public class ObserverPatternDemo {
    public static void main(String[] args) {
        // 创建主题和观察者
        ConcreteSubject subject = new ConcreteSubject();
        Observer observerA = new ConcreteObserverA();
        Observer observerB = new ConcreteObserverB();

        // 注册观察者
        subject.registerObserver(observerA);
        subject.registerObserver(observerB);

        // 执行主题的操作，触发通知
        subject.doSomething();
    }
}
```

## 三, Spring 监听器的应用
### 1. 新建监听器
#### 1.1 实现 ApplicationListener 接口

一个简单的demo

```java
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextRefreshedEvent;
import org.springframework.stereotype.Component;

@Component
public class MyContextRefreshedListener implements ApplicationListener<ContextRefreshedEvent> {

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        System.out.println("应用程序上下文已刷新");
        // 在这里可以执行一些初始化操作
    }
}
```

1. 我们创建了一个名为MyContextRefreshedListener的监听器，它实现了ApplicationListener接口，这意味着这个监听器监听的事件类型是ContextRefreshedEvent，并重写了 onApplicationEvent 方法。该方法在应用程序上下文被刷新时触发。

2. 使用 @Component 注解将该监听器声明为一个Spring管理的组件，这样Spring会自动将其纳入到应用程序上下文中，并在适当的时候触发监听

#### 1.2 @EventListener

或者 在通知方法上 加 注解 不用 继承接口

```java
@Component
public class MyListener {
	@EventListener(ContextRefreshedEvent.class)
	public void methodA(ContextRefreshedEvent event) {
 		System.out.println("应用程序上下文已刷新");
        // 在这里可以执行一些初始化操作
	}
}


```

1. 在一个现有的类的某个方法上，加上@EventListener(ContextRefreshedEvent.class)，Spring会在加载这个类时，为其创建一个监听器，这个监听器监听的事件类型是ContextRefreshedEvent，当此事件发生时，将触发执行该方法methodA。


3. 我们可以在这个类中写上多个方法，每个方法通过注解监听着不同的事件类型，`这样我们就仅需使用一个类，却构建了多个监听器`


### 内置的事件类型

除了该事件，Spring还内置了一些其他的事件类型，分别在以下情况下触发

1. ContextRefreshedEvent：
   当应用程序上下文被刷新时触发。这个事件在ApplicationContext初始化或刷新时被发布，适用于执行初始化操作和启动后的后续处理。例如，初始化缓存、预加载数据等。

2. ContextStartedEvent：
   当`应用程序上下文启动时触发`。这个事件在调用ApplicationContext的start()方法时被发布，适用于在应用程序启动时执行特定的操作。例如，启动定时任务、启动异步消息处理等。

3. ContextStoppedEvent：
   当`应用程序上下文停止时触发`。这个事件在调用ApplicationContext的stop()方法时被发布，适用于在应用程序停止时执行清理操作。例如，停止定时任务、关闭数据库连接等。

4. ContextClosedEvent：
   当`应用程序上下文关闭时触发`。这个事件在调用ApplicationContext的close()方法时被发布，适用于在应用程序关闭前执行最后的清理工作。例如，释放资源、保存日志等。

5. RequestHandledEvent：
   在Web应用程序中，`当一个HTTP请求处理完成后`触发。这个事件在Spring的DispatcherServlet处理完请求后被发布，适用于记录请求日志、处理统计数据等。

6. ApplicationEvent：
   这是一个`抽象的基类`，可以用于定义`自定义`的应用程序事件。你可以创建自定义事件类，继承自ApplicationEvent，并定义适合你的应用场景的事件类型。


### 3. 自定义事件与监听器Demo
开始手写spring事件
#### 3.1 构建两个自定义事件
```java
// 事件A
public class CustomEventA extends ApplicationEvent {
    private String message;

    public CustomEventA(Object source, String message) {
        super(source);
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
// 事件B
public class CustomEventB extends ApplicationEvent {
    private String message;

    public CustomEventB(Object source, String message) {
        super(source);
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}

```

#### 3.2 构建监听
我们选用@EventListener注解来实现监听

```java
@Component
public class MyListener {

	@EventListener(CustomEventA.class)
	public void methodA(CustomEventA event) {
 		System.out.println("========我监听到事件A了:" + event.getMessage());
        // 在这里可以执行一些其他操作
	}

	@EventListener(CustomEventB.class)
	public void methodB(CustomEventB event) {
 		System.out.println("========我监听到事件B了:" + event.getMessage());
        // 在这里可以执行一些其他操作
	}
}

```

#### 3.3 发布事件

```java
@Component
public class CustomEventPublisher implements ApplicationContextAware, ApplicationListener<ContextRefreshedEvent> {
    
    private ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
    
    // 利用容器刷新好的消息为触发，发布两条自定义的事件
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        CustomEventA eventA = new CustomEventA(applicationContext , "我是AAAA");
        CustomEventB eventB = new CustomEventB(applicationContext , "我是BBBB");
        applicationContext.publishEvent(eventA);// 上下文对象也可以发布事件
        applicationContext.publishEvent(eventB);
    }
}
```

## Spirng 监听原理
前面讲到观察者模式, 它的模型是由 观察者实体 和 主题实体 构成, 而 Spring 的监听器模式则结合了Spring本身的特征, 也就是容器化, 在 Spring 中, 监听器实体全部存放在 上下文 对象中, 事件也是 通过上下文对像来进行发布, 具体模型如下

![[Pasted image 20240529225912.png|700]]

们不难看到，虽说是通过ApplicationContext发布的事件，但其并不是自己进行事件的发布，而是引入了一个处理器—— EventMulticaster，直译就是 **事件多播器**，它负责在大量的监听器中，针对每一个要广播的事件，找到事件对应的监听器，然后调用该监听器的响应方法，图中就是调用了监听器1、3、6。

> [!tip]
>  **只有在某类事件第一次广播时，EventMulticaster才会去做遍历所有监听器的事，当它针对该类事件广播过一次后，就会把对应监听器保存起来了，最后会形成一个缓存Map，下一次就能直接找到这些监听器**

```java
final Map<ListenerCacheKey, ListenerRetriever> retrieverCache = new ConcurrentHashMap<>(64);
```

### 2. @EventListener 原理
直接实现监听器接口，然后注册成Bean，这种方式比较好理解，因为我们自己写的实现类就是监听器。但是使用 @EventListener 时，监听器又是怎么产生呢？我们以上面的【自定义事件与监听器Demo】为例，来看一下关键代码。

我们知道，在生成流程中，会对每个Bean都使用PostProcessor来进行加工，而其中就有这么一个类EventListenerMethodProcessor，这个类会在Bean实例化后进行一系列操作

将该 **bean后处理器** 那块, 遍历 该 bean 的所有 被 @EventListener 标注的方法, 并生成监听器, 然后把监听器窜存入容器中

```java
private void processBean(final String beanName, final Class<?> targetType) {
          ......省略前面代码
	// Non-empty set of methods
	ConfigurableApplicationContext context = this.applicationContext;
	Assert.state(context != null, "No ApplicationContext set");
	List<EventListenerFactory> factories = this.eventListenerFactories;
	Assert.state(factories != null, "EventListenerFactory List not initialized");
	// 遍历该Bean中有EventListener注解的方法，此例中即methodA、methodB
	for (Method method : annotatedMethods.keySet()) {
	    // 遍历监听器工厂，这类工厂是专门用来创建监听器的，此处起作用的是默认工厂DefaultEventListenerFactory
		for (EventListenerFactory factory : factories) {
		    // DefaultEventListenerFactory是永远返回true的
			if (factory.supportsMethod(method)) {
				Method methodToUse = AopUtils.selectInvocableMethod(method, context.getType(beanName));
				// 利用该Bean名、Bean类型、方法来创建监听器
				ApplicationListener<?> applicationListener =
						factory.createApplicationListener(beanName, targetType, methodToUse);
				if (applicationListener instanceof ApplicationListenerMethodAdapter) {
					((ApplicationListenerMethodAdapter) applicationListener).init(context, this.evaluator);
				}
				// 把监听器存入容器
				context.addApplicationListener(applicationListener);
				break;
			}
		}
	}
	......省略后面代码
}

```

如上，遍历Bean每个带@EventListener注解的方法，然后利用DefaultEventListenerFactory开始创建监听器，实际上这些监听器类型都是一个适配器类——ApplicationListenerMethodAdapter，只是因为这些监听器具体的参数不一样，所以可以监听不同的事件，做不同的响应

```java
public class DefaultEventListenerFactory implements EventListenerFactory, Ordered {
	// 省略其余代码
	@Override
	public ApplicationListener<?> createApplicationListener(String beanName, Class<?> type, Method method) {
		// 可以看到，每次都是返回一个新对象，所以我们在MyListener里的两个方法都加了@EventListener，其实就会返回两个监听器
		return new ApplicationListenerMethodAdapter(beanName, type, method);
	}
}
```


## 同步和异步

### 1. 默认同步通知
其实，==因为spring默认的多播器没有设置执行器，所以默认采用的是第一种情况，即哪个线程发起的事件，则由哪个线程去通知监听器们==，关键代码如下所示
```java
	// SimpleApplicationEventMulticaster.java
	public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
		ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
		Executor executor = getTaskExecutor();
		for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
			if (executor != null) {
				// 调用线程池异步执行
				executor.execute(() -> invokeListener(listener, event));
			}
			else {
				// 默认走此分支，由发出事件的线程来执行
				invokeListener(listener, event);
			}
		}
	}
	
	@Nullable
	protected Executor getTaskExecutor() {
		return this.taskExecutor;
	}

```
我们可以看到，对于每个监听器的调用是同步还是异步，取决于多播器内部是否含有一个执行器，如果有则交给执行器去执行，如果没有，只能让来源线程一一去通知了。
### 2. 异步通知设置
两种方式，一种是在多播器创建时内置一个线程池，使多播器能够调用自身的线程池去执行事件传播。另一种是不再多播器上做文章，而是在每个监视器的响应方法上标注异步@Async，毫无疑问，第一种才是正道，我们来看看如何做到。其实有多种方法，我们这里说两种。

第一种，直接自定义一个多播器，然后顶替掉Spring自动创建的多播器
```java
@Configuration
public class EventConfig {
    @Bean("taskExecutor")
    public Executor getExecutor() {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(10,
                15,
                60,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue(2000));
        return executor;
    }
    
    // 这其实就是spring-boot自动配置的雏形，所谓的自动配置其实就是通过各种配置类，顶替原有的简单配置
    
    @Bean(AbstractApplicationContext.APPLICATION_EVENT_MULTICASTER_BEAN_NAME)
    // 优先注入 我们自定义的线程池对象
    public ApplicationEventMulticaster initEventMulticaster(@Qualifier("taskExecutor") Executor taskExecutor) {
        SimpleApplicationEventMulticaster simpleApplicationEventMulticaster = new SimpleApplicationEventMulticaster();
        simpleApplicationEventMulticaster.setTaskExecutor(taskExecutor);
        return simpleApplicationEventMulticaster;
    }
}
```


第二种，为现成的多播器设置设置一个线程池
创建一个配置类, 实现 ApplicationContextAware 接口, 在 bean 工厂后处理器的方法后, 会执行该方法, 为 多播器 配置一个线程池对象

```java
@Component
public class WindowsCheck implements ApplicationContextAware {
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        SimpleApplicationEventMulticaster caster = (SimpleApplicationEventMulticaster)applicationContext
                .getBean(AbstractApplicationContext.APPLICATION_EVENT_MULTICASTER_BEAN_NAME);
        ThreadPoolExecutor executor = new ThreadPoolExecutor(10,
                15,
                60,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue(2000));
        caster.setTaskExecutor(executor);
    }
}

```
当然，这里推荐第一种，第二种方法会在spring启动初期的一些事件上，仍采用同步的方式。直至被注入一个线程池后，其才能使用线程池来响应事件。而第一种方法则是官方暴露的位置，让我们去构建自己的多播器。























### BeanFactory与ApplicationContext的关系
1. BeanFactory是Spring的早期接口，称为Spring的Bean工厂，ApplicationContext是后期更高级接口，称之为 Spring 容器
2. ApplicationContext在BeanFactory基础上对功能进行了扩展，例如:监听功能、国际化功能等。BeanFactory的API更偏向底层，所以 ApplicationContext 的很多API都是对 BeanFactory API 的封装
3. `Bean创建的主要逻辑和功能都被封装在BeanFactory中`, ApplicationContext不仅继承了BeanFactory，而且还维护了一个Bean工厂属性
4. Bean的初始化时机不同，原始BeanFactory是在首次调用getBean时才进行Bean的创建，而ApplicationContext则是容器一创建就将Bean都实例化并初始化好
![[Pasted image 20240407085553.png|left|500]]


### BeanFactory 继承体系
BeanFactory是核心接口 , 项目运行过程中肯定有具体实现参与，这个具体实现就是DefaultlistableBeanFactory, 而ApplicationContext内部维护的Beanfactory的实现类也是它


### ApplicationContext的继承体系

只在Spring基础环境下，即只导入spring-context坐标时，此时ApplicationContext的继承体系
![[Pasted image 20240407091257.png|left|400]]

`只`在Spring基础环境下，常用的三个ApplicationContext作用如下:

| 实现类                                | 功能                               |
| ---------------------------------- | -------------------------------- |
| ClassPathXmlApplicationContext     | 加载类路径下的xml配置的ApplicationContext  |
| FileSystemXmlApplicationContext    | 加载磁盘路径下的xml配置的ApplicationContext |
| AnnotationConfigApplicationContext | 加载注解配置类的ApplicationContext       |

## 基于xml的Spring应用


### Spring开发中主要是对Bean的配置，Bean的常用配置一览如下

| xml配置方式                                    | 功能描述                                            |
| ------------------------------------------ | ----------------------------------------------- |
| \<bean id="" class=""\>                    | Bean的id和全限定名配置                                  |
| \<bean name="">                            | 通过name设置Bean的别名，通过别名也能直接获取到Bean实例               |
| \<bean scope="">                           | Bean的作用范围，BeanFactory作为容器时取值singleton和prototype |
| \<bean lazy-init="">                       | Bean的实例化时机，是否延迟加载。BeanFactory作为容器时无效            |
| \<bean init-method="">                     | Bean实例化后自动执行的初始化方法，method指定方法名                  |
| \<bean destroy-method="">                  | Bean实例销毁前的方法，method指定方法名                        |
| \<bean autowire="byType">                  | 设置自动注入模式，常用的有按照类型byType，按照名字byName              |
| \<bean factory-bean="" factory-method=""/> | 指定哪个工厂Bean的哪个方法完成Bean的创建                        |

### Bean的别名设置
getBean("xxx") , 里的xxx为beanName
beanName 默认为id, 无id则为第一个name(别名), 无别名则为全限定类名


### Bean的范围配置
默认情况下，单纯的Spring环境Bean的作用范围有两个:Singleton和Prototype
- Singleton:单例，默认值，Spring容器创建的时候，就会进行Bean的实例化，并存储到容器内部的单例池中, 每次getBean时都是从单例池中获取相同的Bean实例:
- Prototype:原型，Spring容器初始化时不会创建Bean实例，当调用getBean时才会实例化Bean，每次getBean都会创建一个新的Bean实例。而且创建了也`不会放进单例池`


### Bean的延迟加载

lazy-init 只对ApplicationContext生效


### Beand的初始化和销毁方法配置
\<bean init-method="xxx"\> 将xxx方法名填入即可 或实现 InitializingBean 接口, 重写afterPrgpertiesSet()方法, 完成一些Bean的初始化操作(该方法会比xml设置的init方法先执行)

构造方法比init方法先执行
destory方法在关闭容器的时候会被执行


### Bean的实例化配置
Spring的实例化方式主要有如下两种:
- 构造方式实例化: 底层通过无参构造方法对Bean进行实例化
- 工厂方式: 底层Bean工厂通过调用我们自己写的工厂方法来对Bean实例化
怎么做呢?

自己写构造方法
在xml文件\<Bean\>标签里写
\<constructor-arg\>的标签  注意: 这个标签不只是构造方法使用, 工厂方式的方法也可使用

name-参数名   value-参数值













---
###### ctrl + h 可以以大纲的方式查看继承体系
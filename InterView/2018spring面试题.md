# spring

##1.springmvc的原理？

方式一：

初始化过程：

* 1.web.xml配置一个DispatcherServleat(实现Awaer接口)，能够得到ApplicationContext  
* 2.默认加载IOC容器（ApplicationContext  ）
* 3.开始扫描mvc的配置（扫描controller/requestmapping等）view的配置，插件（拦截器/转换器/视图解析器）

> HandlerMapping/HandlerAdapters都是通过扫描注解得到结果

* 4.解析成HandlerMapping的list，主要是保存了url和具体的执行方的对应关系

等待用户请求过程：

* 1.从浏览器输入url
* 2.统一拦截，扫描handlerMapping，如果找不到，返回404/500
* 3.DispatcherServleat接收到请求，从之前初始化保存的数据中找到请求url对应的方法，然后调用
* 4.把调用结果输出



方式二：

* `DispatcherServlet`是 springmvc 中的前端控制器(front controller),负责接收` request `并将 request 转发给对应的处理组件.
* `HanlerMapping `是 springmvc 中完成 url 到 controller 映射的组件.DispatcherServlet 接收 request, 然后从 HandlerMapping 查找处理 request 的 controller.
* Cntroller 处理 request,并返回` ModelAndView `对象,Controller 是 springmvc 中负责处理 request 的组件(类似于 struts2 中的 Action),ModelAndView 是封装结果视图的组件.
* 视图解析器解析 ModelAndView 对象并返回对应的视图给客户端.

一期第三阶段 待补充。。。原理源码部分一定要看。



## springmvc的优化？



* 1.controller 如果能保持单例,尽量使用单例,这样可以减少创建对象和回收对象的开销.也就是说,如果 controller 的类变量和实例变量可以以方法形参声明的尽量以方法的形参声明,不要以类变量和实例变量 声明,这样可以避免线程安全问题.
* 2.处理 request 的方法中的形参务必加上@RequestParam 注解,这样可以避免 springmvc 使用 asm 框 架读取 class 文件获取方法参数名的过程.即便 springmvc 对读取出的方法参数名进行了缓存,如果不要读 取 class 文件当然是更加好.
* 3.阅读源码的过程中,发现 springmvc 并没有对处理 url 的方法进行缓存,也就是说每次都要根据请求 url 去匹配 controller 中的方法 url,如果把 url 和 method 的关系缓存起来,会不会带来性能上的提升呢?有 点恶心的是,负责解析 url 和 method 对应关系的 ServletHandlerMethodResolver 是一个 private 的内部类, 不能直接继承该类增强代码,必须要该代码后重新编译.当然,如果缓存起来,必须要考虑缓存的线程安全 问题.



## 1.spring的ioc原理？





Ioc容器初始化的步骤：

* 1.初始化的入口在容器实现中的 refresh()调用来完成

* 2.对 bean 定义载入 IOC 容器使用的方法是 `loadBeanDefinition`,其中的大致过程如下:通过 `ResourceLoader` 来完成资源文件位置的定位，`DefaultResourceLoader` 是默 认的实现，同时上下文本身就给出了 `ResourceLoader` 的实现，可以从`类路径，文件系统,URL `等方式来 定位资源位置。

  ​

  如果是 `XmlBeanFactory` 作为 IOC 容器，那么需要为它指定 bean 定义的资源，也就是说 bean定义文件时通过抽象成 `Resource`来被IOC容器处理的，容器通过`BeanDefinitionReader`来完成 `定义信息的解析和 Bean 信息的注册`,往往使用的是` XmlBeanDefinitionReader `来解析` bean 的 xml 定义 文件`-实际的处理过程是`委托给 BeanDefinitionParserDelegate`来完成的，从而得到bean的定义信息， 这些信息在 Spring 中使用 `BeanDefinition` 对象来表示-这个名字可以让我们想到` loadBeanDefinition`,`RegisterBeanDefinition `这些相关方法-他们都是为处理`BeanDefinitin`服务的， 容器解析得到` BeanDefinition`IoC 以后，需要把它在 IOC 容器中注册，这由 IOC 实现 `BeanDefinitionRegistry`接口来实现。

  ​

  注册过程就是在IOC容器内部维护的一`HashMap`来保存得到的` BeanDefinition `的过程。这个 HashMap 是 IOC 容器持有 bean 信息的场所，以后对 bean 的操作都 是围绕这个 HashMap 来实现的.

  ​

* 3.然后我们就可以通过` BeanFactory `和 `ApplicationContext `来享受到 SpringIOC 的服务了,在使用 IOC 容器的时候，我们注意到除了少量粘合代码，绝大多数以正确 IOC 风格编写的应用程序代码完全不用关心如何到达工厂，因为容器将把这些对象与容器管理的其他对象钩在一起。基本的策略是把工厂放到已知的地方，最好是放在对预期使用的上下文有意义的地方，以及代码将实际需要访问工厂的地方。Spring 本身提供了对声明式载入 web 应用程序用法的应用程序上下文,并将其存储在 ServletContext 中的框架实现。



在使用 SpringIOC 容器的时候我们还需要区别两个概念:

* `BeanFactory` 和 `FactoryBean`，其中 `BeanFactory `指的是 IOC 容器的编程抽象，比如` ApplicationContext`，`XmlBeanFactory `等，这些都是 IOC 容器的具体表现，需要使用什么样的容器由客 户决定,但 Spring 为我们提供了丰富的选择。
* `FactoryBean `只是一个可以在 IOC 而容器中被管理的一个 bean,是对各种处理过程和资源使用的抽象,FactoryBean 在需要时产生另一个对象，而不返回 FactoryBean 本身,我们可以把它看成是一个抽象工厂，对它的调用返回的是工厂生产的产品。
* 所有的 `FactoryBean `都实现特殊的 `org.springframework.beans.factory.FactoryBean` 接口，当使用容器中 `FactoryBean `的时候，该容器不会返回 `FactoryBean 本身`,而是返回其`生成的对象`。
* Spring 包括了大部 分的通用资源和服务访问抽象的` FactoryBean `的实现，其中包括:对 `JNDI 查询的处理`，对`代理对象的处 理`，对`事务性代理的处理`，对 `RMI 代理的处理`等，这些我们都可以看成是具体的工厂,看成是 Spring 为 我们建立好的工厂。也就是说 Spring 通过使用抽象工厂模式为我们准备了一系列工厂来生产一些特定 的对象,免除我们手工重复的工作，我们要使用时只需要在 IOC 容器里配置好就能很方便的使用了.



## 2.spring的aop原理？



AOP可以通过预编译方式和运行期动态代理实现在不修改源代码的情况下给程序动态统一添加功能的一种技术。

> 我们现在做的一些非业务，如:日志、事务、安全等都会写在业务代码中(也即是说，这些非业务类横切于业务类)，但这些 代码往往是重复，复制——粘贴式的代码会给程序的维护带来不便，AOP 就实现了把这些业务需求与系统需求分开来做。这 种解决的方式也称代理机制。



应用场景：权限认证，日志，事务，懒加载，上下文处理，错误跟踪（异常捕获机制），缓存



***通知(Advice)类型:***

* 前置通知(Before advice):在某连接点(JoinPoint)之前执行的通知，但这个通知不能阻止连接点前的执行。 ApplicationContext 中在<aop:aspect>里面使用<aop:before>元素进行声明。例如，TestAspect 中的 doBefore 方法。
* 后置通知(After advice):当某连接点退出的时候执行的通知(不论是正常返回还是异常退出)。ApplicationContext 中在<aop:aspect>里面使用<aop:after>元素进行声明。例如，ServiceAspect 中的 returnAfter 方法，所以 Teser 中调用 UserService.delete 抛出异常时，returnAfter 方法仍然执行。
* 返回后通知(After return advice):在某连接点正常完成后执行的通知，不包括抛出异常的情况。ApplicationContext 中在<aop:aspect>里面使用<after-returning>元素进行声明。
* 环绕通知(Around advice):包围一个连接点的通知，类似 Web 中 Servlet 规范中的 Filter 的 doFilter 方法。可以在 方法的调用前后完成自定义的行为，也可以选择不执行。ApplicationContext 中在<aop:aspect>里面使用<aop:around> 元素进行声明。例如，ServiceAspect 中的 around 方法。
* 抛出异常后通知(After throwing advice):在方法抛出异常退出时执行的通知。ApplicationContext 中在<aop:aspect> 里面使用<aop:after-throwing>元素进行声明。例如，ServiceAspect 中的 returnThrow 方法。



## Spring的DI？

DI就是指对象是被动接受依赖类而不是自己主动去找，换句话说就是指对象不是从容器中查找它依赖的类，而是在容器实例化对象的时候主动将它依赖的类注入给它。

> 依赖注入、依赖查找，Spring 不仅保存自己创建的对象，而且保存对象与对象之间的关系。

> 注入即赋值，主要三种方式构造方法、set 方法、直接赋值。

​	

**依赖注入发生的时间:**

> 当Spring IOC容器完成了Bean定义资源的定位、载入和解析注册以后，IOC容器中已经管理类Bean 定义的相关数据，但是此时 IOC 容器还没有对所管理的 Bean 进行依赖注入，依赖注入在以下两种情况 发生:

* (1).用户第一次通过 getBean 方法向 IOC 容索要 Bean 时，IOC 容器触发依赖注入。
* (2).当用户在 Bean 定义资源中为<Bean>元素配置了 lazy-init 属性，即让容器在解析注册 Bean 定义时 进行预实例化，触发依赖注入。

## 3.spring的事务？

**事务特点:**

> 事务是恢复和并发控制的基本单位。
>
> 事务应该具有 4 个属性:原子性、一致性、隔离性、持久性。这四个属性通常称为 ACID 特性。 

* 原子性(atomicity)。一个事务是一个不可分割的工作单位，事务中包括的诸操作要么都做，要么都 不做。 
* 一致性(consistency)。事务必须是使数据库从一个一致性状态变到另一个一致性状态。一致性与原 子性是密切相关的。 
* 隔离性(isolation)。一个事务的执行不能被其他事务干扰。即一个事务内部的操作及使用的数据对 并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。 
* 持久性(durability)。持久性也称永久性(permanence)，指一个事务一旦提交，它对数据库中数据 的改变就应该是永久性的。接下来的其他操作或故障不应该对其有任何影响。



**spring事务的基本原理**
Spring 事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring 是无法提供事务功 能的。对于纯 JDBC 操作数据库，想要用到事务，可以按照以下步骤进行:

```
1. 获取连接 Connection con = DriverManager.getConnection() 
2. 开启事务 con.setAutoCommit(true/false);
3. 执行 CRUD
4. 提交事务/回滚事务 con.commit() / con.rollback();
5. 关闭连接 conn.close();
   
```

使用 Spring 的事务管理功能后，我们可以不再写步骤 2 和 4 的代码，而是由 Spirng 自动完成。 那 么 Spring 是如何在我们书写的 CRUD 之前和之后开启事务和关闭事务的呢?解决这个问题，也就可以 从整体上理解 Spring 的事务管理实现原理了。下面简单地介绍下，注解方式为例子

* 1.配置文件开启注解驱动，在相关的类和方法上通过注解@Transactional 标识。
* 2.spring 在启动的时候会去解析生成相关的 bean，这时候会查看拥有相关注解的类和方法，并且 为这些类和方法生成代理，并根据@Transaction 的相关参数进行相关配置注入，这样就在代理中为我们把相关的事务处理掉了(开启正常提交事务，异常回滚事务)。
* 3.真正的数据库层的事务提交和回滚是通过 binlog 或者 redo log 实现的。



**Spring 事务的传播属性**
所谓 spring 事务的传播属性，就是定义在存在多个事务同时存在的时候，spring 应该如何处理这些事 务的行为。这些属性在 TransactionDefinition 中定义.

* PROPAGATION_REQUIRED:支持当前事务，如果当前没有事务，就新建一个事 务。这是最常见的选择，也是 Spring 默认的事务 的传播。

* PROPAGATION_REQUIRES_NEW:新建事务，如果当前存在事务，把当前事务挂起。新建的事务将和被挂起的事务没有任何关系，是两个独立的事务，外层事务失败回滚之后，不能回滚内层事务执行的结果，内层事务失败抛出异常，外层事务捕获，也可以不处理回滚操作.

* PROPAGATION_SUPPORTS:支持当前事务，如果当前没有事务，就以非事务方式执行。

* PROPAGATION_MANDATORY:支持当前事务，如果当前没有事务，就抛出异常。

* PROPAGATION_NOT_SUPPORTED:以非事务方式执行操作，如果当前存在事务，就把

  当前事务挂起.

* PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。

* PROPAGATION_NESTED：如果一个活动的事务存在，则运行在一个嵌套的事 务中。如果没有活动事务，则按 REQUIRED 属性执 行。它使用了一个单独的事务，这个事务拥有多个 可以回滚的保存点。内部事务的回滚不会对外部事 务造成影响。它只对 DataSourceTransactionManager 事务管理器起效。



**数据库隔离级别**

| 隔离级别             | 隔离级别的值 | 导致的问题                                    |
| ---------------- | ------ | ---------------------------------------- |
| Read-Uncommitted | 0      | 导致脏读                                     |
| Read-Committed   | 1      | 避免脏读，允许不可重复读和幻读(默认的)                     |
| Repeatable-Read  | 2      | 避免脏读，不可重复读，允许幻读                          |
| Serializable     | 3      | 串行化读，事务只能一个一个执行，避免了脏读、不可重复读、幻读。执行效率慢，使用时慎重 |

> * 脏读:一事务对数据进行了增删改，但未提交，另一事务可以读取到未提交的数据。如果第一个事务这
>
> 时候回滚了，那么第二个事务就读到了脏数据。
>
> * 不可重复读:一个事务中发生了两次读操作，第一次读操作和第二次操作之间，另外一个事务对数据进
>
> 行了修改，这时候两次读取的数据是不一致的。
>
> * 幻读:第一个事务对一定范围的数据进行批量修改，第二个事务在这个范围增加一条数据，这时候第一
>
> 个事务就会丢失对新增数据的修改。



**Spring 中的隔离级别**

| 常量                         | 解释                                       |
| -------------------------- | ---------------------------------------- |
| ISOLATION_DEFAULT          | 这 是 个 PlatfromTransactionManager 默 认 的 隔离级别，使用数据库默认的事务隔离级别。另外 四个与 JDBC 的隔离级别相对应。 |
| ISOLATION_READ_UNCOMMITTED | 这是事务最低的隔离级别，它充许另外一个事务可以看到这个事务未提交的数据。这种隔离级别会产生脏读，不可重复读和幻像读。 |
| ISOLATION_READ_COMMITTED   | 保证一个事务修改的数据提交后才能被另外一个 事务读取。另外一个事务不能读取该事务未提交的数据。 |
| ISOLATION_REPEATABLE_READ  | 这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻像读。          |
| ISOLATION_SERIALIZABLE     | 这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行。         |



spring传播属性（嵌套）：

* PROPAGATION_REQUIRED(spring 默认)
* PROPAGATION_REQUIRES_NEW
* PROPAGATION_SUPPORTS
* PROPAGATION_NESTED



一期，第四章待补充。。。。



## 4.spring作用域？

spring容器中的bean可以分为5个范围。

* singleton：这种bean范围是默认的，这种范围确保不管接收到多少请求，每个容器中只有一个bean的实力，单例的模式由bena factory自身来维护。

* prototype：原型范围与单例范围相反，为每一个bean请求提供一个实例。

* request：在请求bean范围内会为每一个来自客户端的网络请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。

* session：与请求范围类似，确保每个session中有一个bean的实例，在session过期后，bean会随之失效。

* global-session：global-session和portlet应用相关。当你的应用部署在portlet容器中工作时，它包含很多portlet。如果你想要声明让所有的portlet共用全局的存储变量的话，那么这全局变量需要存储在global-session中。

  > 全局作用域与servlet中的session作用域效果相同。

## 5.动态代理（CGLib 与 JDK）、优缺点、性能对比、如何选择？

* cglib  asm（自动生成一个类，去继承原来类），更优的字节码重组方案

* jdk 内置（必须要实现接口，因为生成代理对象之后要强制转型）

  > Spring 提供了两种方式来生成代理对象: JDKProxy 和 Cglib，具体使用哪种方式生成由 AopProxyFactory 根据 AdvisedSupport 对象的配置来决定。默认的策略是如果目标类是接口，则使用 JDK 动态代理技术，否则使用 Cglib 来生成代 理。

## 6.Spring中循环注入是什么意思，可不可以解决，如何解决；

利用递归+缓存



## 7.spring怎么保证bean不被回收？

* IOC容器，在map中，会一致持有对象的引用
* Map又是单例的，map本身不会被回收

## 8.@auware 和@resource区别？



## 9.不同数据源的事务如何处理？

在spring中，事务是不支持跨数据源

换句话说，一个事务不能操作两个数据库

> Datasource是connection，当创建语句集的时候开启事务

解决：使用中间件，做一些消息同步，利用分布式式锁去实现分布式事务。



## 10.spring如何保证controller并发的安全？

spring5的新特性就开始考虑这个问题了

如果对并发有要求，推荐使用springboot。






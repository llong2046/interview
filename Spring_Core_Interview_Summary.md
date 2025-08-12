# Spring 核心面试要点

### 一、 核心思想：IoC 与 AOP

*   **IoC (Inversion of Control - 控制反转)**
    *   **定义**: 一种设计原则，将原本由程序代码直接操控的对象创建、依赖关系维护等控制权，交给外部容器来管理。IoC是目标，DI是实现手段。
    *   **核心思想**: 程序本身不创建依赖对象，而是被动地等待IoC容器来创建并注入。这实现了组件间的解耦，提高了代码的灵活性和可维护性。

*   **DI (Dependency Injection - 依赖注入)**
    *   **定义**: IoC最常见的实现方式。容器在创建Bean时，会主动将其所依赖的其他Bean注入到对应的属性中。
    *   **注入方式**: 
        1.  **构造器注入**: 通过构造函数的参数注入。官方推荐，能保证依赖在对象构造时就已完备。
        2.  **Setter注入**: 通过setter方法注入。更灵活，但无法保证所有依赖都被注入。
        3.  **字段注入**: 通过`@Autowired`直接在字段上注入。代码简洁，但通用性差，不推荐在业务代码中使用。

*   **AOP (Aspect-Oriented Programming - 面向切面编程)**
    *   **定义**: 一种编程范式，旨在将横切关注点（Cross-cutting Concerns）与业务逻辑主体进行分离。
    *   **核心思想**: 将日志、事务、安全等通用功能（切面），动态地织入到业务逻辑（连接点）中，而无需修改业务代码本身。
    *   **关键术语**: **Aspect(切面)**, **Join Point(连接点)**, **Pointcut(切点)**, **Advice(通知)**, **Weaving(织入)**。

```
助记关键词: 容器管理, 被动接收, 功能织入, 业务解耦
助记/类比:
IoC/DI: 你（程序）想喝可乐（依赖），不用自己跑去造可乐（new对象），而是直接告诉管家（Spring容器）你要可乐，管家会把可乐递到你手上（注入）。
AOP: 大楼里的安保系统（切面）。每个员工（业务方法）进出办公室时，安保系统都会自动完成开门、记录日志、监控（通知），员工只需专注于自己的工作，无需关心安保细节。
```

### 二、 Bean 深度解析

*   **Bean的定义**
    *   **定义**: 被Spring IoC容器初始化、组装和管理的对象。本质上就是一个由容器管理的Java对象。

*   **Bean的生命周期 (Lifecycle)**
    1.  **实例化 (Instantiation)**: Spring容器根据配置创建Bean的实例。
    2.  **属性填充 (Populate Properties)**: 进行依赖注入，填充所有属性。
    3.  **初始化 (Initialization)**: 执行Aware接口（如`BeanNameAware`）、Bean后置处理器（`postProcessBeforeInitialization`）、`@PostConstruct`注解、`InitializingBean`接口的`afterPropertiesSet`方法、自定义的`init-method`，最后是Bean后置处理器（`postProcessAfterInitialization`）。
    4.  **使用 (In Use)**: Bean处于可用状态，响应业务请求。
    5.  **销毁 (Destruction)**: 容器关闭时，执行`@PreDestroy`注解、`DisposableBean`接口的`destroy`方法、自定义的`destroy-method`。

*   **Bean的作用域 (Scope)**
    *   **singleton (单例)**: **默认作用域**。在整个容器中，该Bean只有一个实例。
    *   **prototype (原型)**: 每次请求（注入或`getBean()`）时，都会创建一个新的Bean实例。
    *   **request**: 每次HTTP请求创建一个新实例，仅适用于Web应用。
    *   **session**: 每个HTTP Session创建一个新实例，仅适用于Web应用。

```
助记关键词: 容器管理对象, 实例化->初始化->销毁, 单例/原型
助记/类比:
Bean生命周期: 就像一个员工入职到离职。公司（容器）先给你办好入职手续（实例化），发电脑配权限（属性填充），进行岗前培训（初始化），然后你开始正式工作（使用），最后办理离职交接（销毁）。
Bean作用域: 公司的员工类型。大部分是正式员工（singleton），全公司就这一个人；有些是项目外包人员（prototype），每个新项目都招一批新的人来干。
```

### 三、 Spring Boot 核心机制

*   **核心优势：约定优于配置 (Convention over Configuration)**
    *   **思想**: Spring Boot提供了大量默认配置，旨在让开发者尽可能少地进行配置，快速启动和开发项目。

*   **自动装配 (Auto-Configuration)**
    *   **原理**: Spring Boot会检查应用的classpath下有哪些依赖。如果检测到特定依赖（如`spring-webmvc`），它就会根据预设的条件（`@ConditionalOn...`）自动配置相关的功能（如`DispatcherServlet`）。
    *   **核心**: 核心逻辑在`spring-boot-autoconfigure.jar`包中，通过`@EnableAutoConfiguration`注解启动。

*   **起步依赖 (Starters)**
    *   **定义**: 一组方便的依赖描述符。它将构建特定类型应用所需的通用依赖聚合在一起。
    *   **作用**: 开发者只需引入一个starter（如`spring-boot-starter-web`），即可获得包括内嵌Tomcat、Spring MVC、Jackson等所有相关依赖，无需手动管理版本和依赖关系。

```
助记关键词: 约定优于配置, 条件装配, 依赖套餐
助记/类比:
Spring Boot: 如果说Spring Framework是给你一堆乐高积木，让你自由发挥；那么Spring Boot就是给你一个成套的乐高跑车模型，包含了所有需要的零件和一本极简的说明书（自动装配），让你能快速拼出跑车。
起步依赖(Starters): 乐高套装里按步骤分好的零件包，比如“车轮包”、“底盘包”，你只需要按需引入即可。
```

### 四、 Spring MVC 工作原理

*   **核心组件：DispatcherServlet**
    *   **定义**: 前端控制器，是Spring MVC的核心。所有进入应用的HTTP请求都首先由它接收和分发。

*   **请求处理流程**
    1.  请求到达 `DispatcherServlet`。
    2.  `DispatcherServlet` 查询 `HandlerMapping`，找到处理该请求的 `Controller` 方法。
    3.  `DispatcherServlet` 将请求交给 `HandlerAdapter`，由它来实际调用 `Controller` 方法。
    4.  `Controller` 方法执行业务逻辑，返回一个 `ModelAndView` 对象或由 `@ResponseBody` 标记的结果。
    5.  如果返回 `ModelAndView`，`ViewResolver` 会解析视图名，找到对应的 `View`。
    6.  `DispatcherServlet` 渲染视图，并将最终的HTTP响应返回给客户端。

*   **常用注解**
    *   `@Controller` / `@RestController`: 标记一个类为控制器。`@RestController`是`@Controller`和`@ResponseBody`的组合。
    *   `@RequestMapping` / `@GetMapping` / `@PostMapping`: 映射请求URL到具体的处理方法。
    *   `@RequestParam` / `@PathVariable` / `@RequestBody`: 用于从请求中提取参数。

```
助记关键词: 前端控制器, 请求分发, 组件协同, 注解驱动
助记/类比:
DispatcherServlet: 餐厅的总管。客人（HTTP请求）来了，总管先查预订本（HandlerMapping）找到负责的包间服务员（Controller）。然后让传菜员（HandlerAdapter）通知服务员上菜。服务员处理完后，将菜品（ModelAndView）交给总管，总管安排上菜（渲染视图），最后送到客人面前（HTTP响应）。
```

### 五、 事务管理

*   **核心注解: `@Transactional`**
    *   **定义**: 提供声明式事务管理。只需在方法或类上添加此注解，Spring就会通过AOP为此方法自动开启和管理事务。

*   **事务的ACID属性**
    *   **原子性 (Atomicity)**: 事务中的所有操作，要么全部成功，要么全部失败回滚。
    *   **一致性 (Consistency)**: 事务执行前后，数据库从一个一致性状态转移到另一个一致性状态。
    *   **隔离性 (Isolation)**: 一个事务的执行不能被其他事务干扰。
    *   **持久性 (Durability)**: 一个事务一旦提交，它对数据库中数据的改变就是永久性的。

*   **事务传播行为 (Propagation)**
    *   **定义**: 当一个带有事务的方法调用另一个带有事务的方法时，事务该如何传递。
    *   **`REQUIRED` (默认)**: 如果当前存在事务，则加入该事务；如果不存在，则创建一个新的事务。
    *   **`REQUIRES_NEW`**: 总是创建一个新的事务。如果当前存在事务，则将当前事务挂起。

*   **事务隔离级别 (Isolation)**
    *   **定义**: 控制并发事务之间数据的可见性程度。
    *   **目的**: 防止**脏读**、**不可重复读**、**幻读**等并发问题。

```
助记关键词: 声明式事务, ACID, 事务传播, 并发隔离
助记/类比:
@Transactional: 就像给一项重要任务（比如转账）盖上一个“要么全部做完，要么一步都别做”的印章。AOP是执行这个印章规则的公证人。
事务传播: 你（一个事务）正在处理一个任务，需要调用同事（另一个事务方法）帮忙。`REQUIRED`就是你俩在同一张申请表上签字；`REQUIRES_NEW`是你的同事另外拿了一张新申请表，他签他的，你签你的，互不影响。
```

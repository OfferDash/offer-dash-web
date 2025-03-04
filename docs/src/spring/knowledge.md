# Spring 高频考点

## Spring 基础

****

### Spring IOC

****

**传统程序设计**：对象的创建、依赖管理由应用程序代码直接控制：
  ```java
public interface T {}                  // 接口定义

public class A implements T {}         // 实现类A
public class B implements T {}         // 实现类B

public class Main {
    T a = new A();  // 直接依赖具体实现类A
    T b = new B();  // 直接依赖具体实现类B
}
  ```
**IoC模式**：控制权反转至容器（如Spring），由容器负责对象的生命周期和依赖注入。例如：
  ```java
public interface T {}

@Component("a")  // 标记为Spring Bean，并指定名称"a"
public class A implements T {}

@Component("b")  // 标记为Spring Bean，并指定名称"b"
public class B implements T {}

@Component
public class Main {
    @Autowired
    @Qualifier("a")  // 指定注入名为"a"的Bean（对应类A）
    private T a;

    @Autowired
    @Qualifier("b")  // 指定注入名为"b"的Bean（对应类B）
    private T b;
}
  ```

**IoC的三大核心优势**：
1. 解耦与抽象：  
   - 使用者无需关注实现细节：一个接口可能有不同实现类，通过依赖接口注入而非依赖具体类。
   - 符合开闭原则：替换实现只需修改配置，无需改动业务代码。
2. 资源高效利用：
   - 单例模式：Spring默认以单例管理Bean，避免重复创建对象，节省内存。
   - 懒加载：通过`@Lazy`延迟初始化，优化启动性能。
3. 隔离变化影响：  
   - Bean修改透明化：若Bean的接口不变，内部逻辑修改（如性能优化）不会影响调用方。

****

### Spring AOP

****

AOP 是面向切面编程，将对多个对象产生影响的公共行为和逻辑抽取出来进行复用，降低耦合。

**AOP 底层原理**
- 代理模式：
  - JDK 动态代理：动态代理只能对实现了接口的类生成代理，而不能针对类。
  - CGLIB 代理：基于继承，适用于无接口的类（无法代理 final 类/方法，因为 final 修饰的无法继承）。
  - 默认策略：Spring 优先使用 JDK 代理，若无接口则切换至 CGLIB。可通过 `@EnableAspectJAutoProxy(proxyTargetClass=true)` 强制使用 CGLIB。
- 织入时机：在 Bean 初始化阶段（`postProcessAfterInitialization`），通过 `AbstractAutoProxyCreator` 创建代理对象。
- 性能对比：CGLIB 创建代理较慢但执行快，JDK 代理反之。

**内部调用失效**
- 问题：同类中方法互相调用时，AOP 增强可能不生效。
- 解决方案：
  - 通过 `AopContext.currentProxy()` 获取代理对象。
  - 将方法拆分到不同类中。

****

### Bean 生命周期

****

以下是 **Bean 生命周期**的四个阶段及其对应接口与配置方法的表格总结：

| **阶段**          | **接口/配置方法**               | **具体工作**                                                                 |
|-------------------|-------------------------------|-------------------------------------------------------------------------|
| **实例化** (Instantiation) | 构造函数、工厂方法               | 容器通过构造函数或工厂方法创建 Bean 的实例。                                |
| **属性赋值** (Populate)   | `Aware` 接口（如 `BeanNameAware`、`BeanFactoryAware` 等） | 注入依赖的属性值，并处理 `Aware` 接口的回调（例如设置 Bean 名称、BeanFactory 等）。 |
| **初始化** (Initialization) | `InitializingBean` 接口的 `afterPropertiesSet()` 方法 | 属性赋值完成后调用，执行自定义初始化逻辑。                                    |
|                   | `init-method` 配置的自定义方法    | 在 Bean 配置中指定的初始化方法（例如 XML 中的 `init-method` 或 `@Bean(initMethod)`）。 |
| **销毁** (Destruction)    | `DisposableBean` 接口的 `destroy()` 方法 | 容器销毁 Bean 前调用，执行自定义销毁逻辑。                                    |
|                   | `destroy-method` 配置的自定义方法 | 在 Bean 配置中指定的销毁方法（例如 XML 中的 `destroy-method` 或 `@Bean(destroyMethod)`）。 |

**执行顺序**：
   - 初始化阶段：先执行 `afterPropertiesSet()`，再执行 `init-method`。
   - 销毁阶段：先执行 `destroy()`，再执行 `destroy-method`。

除了上述接口，还可以通过 `BeanPostProcessor` 在 Bean 生命周期前后插入自定义逻辑（例如 `postProcessBeforeInitialization` 和 `postProcessAfterInitialization`）。

****

### 三级缓存机制

****

**循环依赖**：

- 循环依赖其实就是循环引用，也就是两个或两个以上的 Bean 互相持有对方，比如 A 依赖于 B，B 依赖于 A，形成了闭环。 
- 循环依赖在 Spring 中允许存在，可以依据三级缓存已经解决了大部分的循环依赖 。

| **缓存名称**              | **存储内容**                                                                 |
|--------------------------|----------------------------------------------------------------------------|
| **一级缓存**（`singletonObjects`） | 缓存已经经历了完整的生命周期，已经初始化完成的 Bean 对象          |
| **二级缓存**（`earlySingletonObjects`） | 实例化完成但未初始化完成的早期 Bean 对象（未完成属性赋值和初始化）           | 
| **三级缓存**（`singletonFactories`） | 存储 `ObjectFactory` 对象工厂，提供一个匿名内部类，用于创建二级缓存中的对象 | 

**三级缓存执行过程**：
   - 创建 Bean A 时，先实例化 A 对象（未初始化），将 A 的 `ObjectFactory` 存入三级缓存。
   - 若 A 依赖 Bean B，则创建 B；此时 B 又依赖 A，会从三级缓存中获取 A 的 `ObjectFactory`，生成 A 的早期代理对象，并将该对象存入二级缓存。
   - B 完成初始化后，A 继续完成属性赋值和初始化，最终将完整的 A 存入一级缓存，并清除二、三级缓存。

****

### Spring 事务

****

Spring 支持**编程式事务**管理和**声明式事务**管理两种：

- 编程式事务控制：
  - 需要使用 TransactionTemplate 来进行实现。
  - 这种方式实现对业务代码有侵入性，因此在项目中很少被使用到。
- 声明式事务管理：
  - 声明式事务管理建立在 AOP 之上
  - 通过 AOP 功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中。
  - 目标方法开始之前加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。
  - 通过 @Transactional 注解的方式使用，无侵入性。

**Spring 事务传播机制**：

| **传播机制**          | **行为描述**                                                                 | **适用场景**                                                                 |
|-----------------------|----------------------------------------------------------------------------|----------------------------------------------------------------------------|
| **REQUIRED**（默认）  | 当前有事务则加入，没有则新建一个事务。                                           | 大多数场景（如订单创建后更新库存）。                                           |
| **REQUIRES_NEW**      | 总是新建事务，暂停当前事务（如果存在）。                                          | 独立业务（如日志记录需独立提交）。                                             |
| **SUPPORTS**          | 当前有事务则加入，没有则以非事务执行。                                           | 查询方法（可适应有无事务环境）。                                               |
| **NOT_SUPPORTED**     | 以非事务执行，暂停当前事务（如果存在）。                                          | 非核心操作（如发送通知）。                                                     |
| **MANDATORY**         | 强制要求存在事务，否则抛出异常。                                                | 必须依赖外部事务（如子方法需在事务中执行）。                                       |
| **NEVER**             | 强制要求无事务，否则抛出异常。                                                 | 禁止事务操作（如纯查询方法）。                                                  |
| **NESTED**            | 嵌套事务，外层事务回滚会导致内层回滚，内层回滚不影响外层。                           | 复杂业务分层（如订单主表与明细表操作）。                                         |

**Spring 事务失效的场景**：

| **场景**               | **原因**                                                                 | **解决方案**                                                                 |
|------------------------|-------------------------------------------------------------------------|----------------------------------------------------------------------------|
| **异常被捕获未抛出**   | 事务内部的异常被 `try-catch` 捕获后未重新抛出，Spring 无法感知异常。          | 在 `catch` 块中使用 `throw` 抛出异常。                                         |
| **抛出检查异常**       | Spring 默认只回滚非检查异常（`RuntimeException` 和 `Error`）。               | 添加 `@Transactional(rollbackFor = Exception.class)`。                        |
| **方法非 `public`**    | CGLIB 代理无法代理 `private` 或非 `public` 方法。                            | 将事务方法改为 `public`。                                                    |
| **非代理对象调用**     | 直接调用 `this.方法()` 或子类调用未继承 `@Transactional` 的方法。               | 通过代理对象调用（如注入自身 Bean 或调用父类方法 `super.方法()`）。              |
| **数据库引擎不支持**   | 如 MySQL 的 MyISAM 引擎不支持事务。                                          | 改用 InnoDB 引擎。                                                          |
| **自调用问题**         | 同一类中方法 A 调用方法 B（B 有 `@Transactional`），事务失效。                  | 通过代理对象调用（如注入自身 Bean 或拆分方法到不同类）。                         |

AspectJX
==================================

 一个基于AspectJ并在此基础上扩展出来可应用于Android开发平台的AOP框架，可作用于java源码，class文件及jar包，同时支持kotlin的应用。 
 AOP （Aspect-Oriented Programming），面向切面编程，是一种编程思想。AOP 以切面（aspect）为基础，切面是一种新的模块化机制，用来描述分散在对象、类或函数中的横切关点（crosscutting concern）。
 
## 最近更新

#### v2.0.4

#### v2.0.2
* 解决无法识别kotlin写的Aspect文件的Bug

#### v2.0.0
* 支持Instant Run编译
* 废弃 `includeJarFilter`和`excludeJarFilter`两个配置命令，改为`include`和 `exclude`配置命令，通过包名(package)路径关键字匹配，可过滤class文件和库文件(jar)




## 如何使用 --写给自己，原文档请转到原作者文档

* **第一步：插件引用**

a.在项目根目录的build.gradle里依赖**AspectJX**

```
 dependencies {
        classpath 'com.hujiang.aspectjx:gradle-android-plugin-aspectjx:2.0.4'
 }
```

b.或者直接使用product/gradle-android-plugin-aspectjx-2.0.0.jar，然后记得依赖jar包
`注意`: 

*如果用在module,`compile 'org.aspectj:aspectjrt:1.8.+'` 必须添加到包含有AspectJ代码的module. [可以参考Demo](https://github.com/HujiangTechnology/AspectJX-Demo/blob/master/library/build.gradle)

* **第二步：在app项目的build.gradle里应用插件**

```
apply plugin: 'android-aspectjx'
//或者这样也可以
apply plugin: 'com.hujiang.android-aspectjx'
```

* **AspectJX配置**

**AspectJX**默认会处理所有的二进制代码文件和库，为了提升编译效率及规避部分第三方库出现的编译兼容性问题，**AspectJX**提供`include`,`exclude`命令来过滤需要处理的文件及排除某些文件(包括class文件及jar文件)。
> 2.0.0版本的 `include`,`exclude`通过package路径匹配class文件及jar文件：

**支持**

```
aspectjx {
//排除所有package路径中包含`android.support`的class文件及库（jar文件），支持通配符写法，`*`和`**`匹配**
//通常加入 include 你想处理的模块即可，加快编译
//需要写出全类名模式，不要只写你的第三方库名，可以使用通配符匹配
	exclude 'android.support'
}

aspectjx {
//关闭AspectJX功能，默认是true，
	enabled false
}
```
#####################################################################################################################################
**使用方法：**
1，侵入式用法，一般会使用自定义注解，以此作为选择切入点的规则。
**第一步，自定义一个注解，例如
```

@Target({TYPE, METHOD, CONSTRUCTOR}) @Retention(CLASS)
public @interface DebugLog {
 String value() default "功能x";
}

 在处理的类中使用你自定义的注解@DebugLog
 
```
**第二步，切入点处理
```
@Aspect
public class Hugo {
  private static volatile boolean enabled = true;

  @Pointcut("within(@hugo.weaving.DebugLog *)")
  public void withinAnnotatedClass() {} // @DebugLog 修饰的类、接口的 Join Point

  // synthetic 是内部类编译后添加的修饰语，所以 !synthetic 表示非内部类的

  @Pointcut("execution(!synthetic * *(..)) && withinAnnotatedClass()")
  public void methodInsideAnnotatedType() {} // 执行 @DebugLog 修饰的类、接口中的方法，不包括内部类中方法
  

  @Pointcut("execution(!synthetic *.new(..)) && withinAnnotatedClass()")
  public void constructorInsideAnnotatedType() {} // 执行 @DebugLog 修饰的类中的构造函数，不包括内部类的构造函数

  @Pointcut("execution(@hugo.weaving.DebugLog * *(..)) || methodInsideAnnotatedType()")
  public void method() {} // 执行 @DebugLog 修饰的方法，或者 @DebugLog 修饰的类、接口中的方法

  @Pointcut("execution(@hugo.weaving.DebugLog *.new(..)) || constructorInsideAnnotatedType()")
  public void constructor() {} // 执行 @DebugLog 修饰的构造函数，或者 @DebugLog 修饰的类中的构造函数

  ...

  @Around("method() || constructor()")
  public Object logAndExecute(ProceedingJoinPoint joinPoint) throws Throwable {
    enterMethod(joinPoint); // 打印切入点方法名、参数列表

    long startNanos = System.nanoTime();
    Object result = joinPoint.proceed(); // 调用原来的方法
    long stopNanos = System.nanoTime();
    long lengthMillis = TimeUnit.NANOSECONDS.toMillis(stopNanos - startNanos);

    exitMethod(joinPoint, result, lengthMillis); // 打印切入点方法名、返回值、方法执行时间

    return result;
  }
Join Point
Join Point 表示连接点，即 AOP 可织入代码的点，下表列出了 AspectJ 的所有连接点：



Join Point
说明

Method call
方法被调用

Method execution
方法执行


Constructor call
构造函数被调用


Constructor execution
构造函数执行

Field get
读取属性

Field set
写入属性


Pre-initialization
与构造函数有关，很少用到


Initialization
与构造函数有关，很少用到


Static initialization
static 块初始化


Handler
异常处理


Advice execution
所有 Advice 执行



Pointcuts
Pointcuts 是具体的切入点，可以确定具体织入代码的地方，基本的 Pointcuts 是和 Join Point 相对应的。



Join Point
Pointcuts syntax




Method call
call(MethodPattern)


Method execution
execution(MethodPattern)


Constructor call
call(ConstructorPattern)


Constructor execution
execution(ConstructorPattern)


Field get
get(FieldPattern)


Field set
set(FieldPattern)


Pre-initialization
initialization(ConstructorPattern)


Initialization
preinitialization(ConstructorPattern)


Static initialization
staticinitialization(TypePattern)


Handler
handler(TypePattern)


Advice execution
adviceexcution()

除了上面与 Join Point 对应的选择外，Pointcuts 还有其他选择方法：



Pointcuts synatx
说明


within(TypePattern)
符合 TypePattern 的代码中的 Join Point


withincode(MethodPattern)
在某些方法中的 Join Point


withincode(ConstructorPattern)
在某些构造函数中的 Join Point


cflow(Pointcut)
Pointcut 选择出的切入点 P 的控制流中的所有 Join Point，包括 P 本身


cflowbelow(Pointcut)
Pointcut 选择出的切入点 P 的控制流中的所有 Join Point，不包括 P 本身


this(Type or Id)调用注解类所在对象


target(Type or Id)注解所在的对象



args(Type or Id, ...)
方法或构造函数参数的类型


if(BooleanExpression)
满足表达式的 Join Point，表达式只能使用静态属性、Pointcuts 或 Advice 暴露的参数、thisJoinPoint 对象



Pointcut 表达式还可以 ！、&&、|| 来组合，!Pointcut 选取不符合 Pointcut 的 Join Point，Pointcut0 && Pointcut1 选取符合 Pointcut0 和 Pointcut1 的 Join Point，Pointcut0 || Pointcut1 选取符合 Pointcut0 或 Pointcut1 的 Join Point。
上面 Pointcuts 的语法中涉及到一些 Pattern，下面是这些 Pattern 的规则，[]里的内容是可选的：



Pattern
规则
链接：https://www.jianshu.com/p/691acc98c0b8







```















## 常见问题

* 问：**AspectJX**是否支持`*.aj`文件的编译?

答：
不支持。目前**AspectJX**仅支持annotation的方式，具体可以参考[支持kotlin代码织入的AspectJ Demo](https://github.com/HujiangTechnology/AspectJ-Demo)

* 问：编译时会出现`can't determine superclass of missing type**`及其他编译错误怎么办

答：大部分情况下把出现问题相关的class文件或者库（jar文件）过滤掉就可以搞定了


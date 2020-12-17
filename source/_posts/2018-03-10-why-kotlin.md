---
title: 为什么使用 Kotlin
date: 2018-03-10 22:58:40
categories:
- Android
tags:
---

本文主要讲述一些个人推荐使用 Kotlin 的理由。

<!-- more -->

# 1 JetBrains 亲儿子，Google 干儿子

Kotlin 由大名鼎鼎的 JetBrains 开发维护，并且得到了 Google 的官方认可，可谓是 JetBrains 的亲儿子，Google 的干儿子，据说 Google 还在最新的 Android P 上，在虚拟机层对 Kotlin 进行了专门的性能优化。

# 2 Kotlin 和 Java 完全兼容

Kotlin 和 Java 一样都是基于 JVM 的语言，所以他们俩其实就是亲兄弟，自然也不存在什么兼容性问题，Java 的 API 在 Kotlin 里会被自动转换成 Kotlin 语法，反过来 Kotlin 的 API 在 Java 里也会自动转换成 Kotlin 语法，所以你完全不需要担心什么兼容性问题。

关于两个语言是如何相互转换的，我猜测都是通过字节码进行解析然后转换的，仅仅是猜测而已。

# 3 跟 setter 和 getter 说再见

据说我们这辈子撸过最多的代码是 setter 和 getter？因为你得为数据封装类的每一个字段写 setter 和 getter，机智一点的会使用 IDE 生成代码，而 Kotlin 则一句话就搞定了。

比如我们要写一个 Person 的数据封装类，用 Java 的话我们会这样写： 

```java
public class Person {

    private String name;
    private String addr;
    private int age;

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setAddr(String addr) {
        this.addr = addr;
    }

    public String getAddr() {
        return addr;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public int getAge() {
        return age;
    }

}
```

这段 Java 代码看起来是不是又臭又长，我们换 Kotlin 写个 Person 类看看：

```kotlin
data class Person(var name: String, var addr: String, var age: Int)
```

什么！这就完了吗？没错这就完了，你需要的 setter 和 getter 方法 Kotlin 都已经为你自动生成了！

# 4 参数很多看着懵逼？

如果一个方法有很多参数，在调用的时候难免会出现看不出每个参数是什么意思的情况，尤其是传递数字的时候，比如我们有下面一个方法：

```java
public Rectange createRectange(int x, int y, int width, int height) {}
```

我们在调用的时候这样写：

```java
createRectangle(20, 30, 40, 50);
```

什么鬼！对 API 不熟悉的人根本没办法一眼看出这个方法调用传递的参数都是什么意思！我们换 Kotlin 写个看看：

```java
fun createRectangle(x: Int, y: Int, width: Int, height: Int) {}
```

Kotlin 允许我们在调用方法的时候显式指定参数的名称，如下所示：

```kotlin
createRectangle(x = 20, y = 30, width = 40, height = 50)
```

怎么样！是不是看起来清晰多了，一眼就看出在坐标为 (20, 30) 的位置创建一个宽 40，高 50 的矩形。

# 5 跟方法重载说再见

如果一个方法有 N 个参数，并且有些参数是可选的，你是不是要为它写好几个重载版本，当让你也可以用 Builder 解决该问题，不管哪一种方式我都要多写很多代码啊！在 Kotlin 中你可以给每一个参数指定默认值，具有默认值的参数可以在调用的时候选填，是不是很爽！

我们来看下面这个 Java 方法：

```java
public void foo() {
    foo(0, 0, 0)
}

public void foo(int a) {
    foo(a, 0, 0)
}

public void foo(int a, int b) {
    foo(a, b, 0)
}

public void foo(int a, int b, int c) {
    this.a = a;
    this.b = b;
    this.c = c;
}
```

再来看看 Kotlin 的写法：

```kotlin
fun foo(a: Int = 0, b: Int = 0, c: Int = 0) {}
```

搞定了！我们来看看这方法怎么调用：

```kotlin
foo()
foo(a = 1)
foo(b = 2)
foo(c = 3)
foo(a = 1, b = 2)
foo(a = 1, b = 2, c = 3)
```

# 6 用 as 代替 () 进行类型转换

如果你是 Android 开发者，相信你一定对 `findViewById()` 方法不陌生吧，在获取每一个 View  的时候都需要进行类型转换，如下所示：

```java
Button button = (Button) findViewById(R.id.button);
ImageView imageView = (ImageView) findViewById(R.id.image_view);
```

这样的类型转换方式本身没什么问题，但是 Kotlin 有更优雅的类型转换关键字 `as`：

```kotlin
Button button = findViewById(R.id.button) as Button;
ImageView imageView = findViewById(R.id.image_view) as Button;
```

两种方式比较起来，个人更喜欢 Kotlin 的方式，原因如下：

*   用 `as` 代替 `()` 更加具有可读性
*   敲代码的时候打 `as` 比打 `()` 更快ヾ(｡｀Д´｡)

# 7 让方法重写或实现更明显

我们来看下面这段代码：

```java
public class A extends B {
  
    public void foo() {
      
    }
  
}
```

如果我告诉你 A 的 `foo()` 方法是重写 B 的 `foo()` 方法，你会怎么想？反正我是想骂人了，标记重写方法的 `@Override` 被狗吃了吗？这样的写法虽然编译器不会报错，但是却不利于代码的阅读，很容易让人误以为 `foo()` 方法是 A 自己新定义的。在 Kotlin 中则不允许你这样写，它要求你必须显式声明某一个方法是重写父类或接口的：

```kotlin
class A: B() {

    override fun foo() {
    
    }
  
}
```

有了 `override` 这个关键字，我们一看就知道这是一个重写的方法。

# 8 支持类的方法扩展

Kotlin 支持对一类进行方法扩展，注意这个扩展并不是通过继承，也就是它可以扩展一个 final 类。例如我们可以扩展 String 类，添加一个 `log(tag: String)` 方法，让它可以在日志系统里面输出自己的内容：

```Kotlin
fun String.log(tag: String) {
    Log.d(tag, this)
}
```

然后我们就可以这么用：

```Kotlin
val myString: String = 'test'
myString.log(TAG)
```

上面的 String 扩展很无聊，我们来一个逼格高一些的，例如我们可以扩展 SharedPreferences 类的 `edit()` 方法，让它支持更便捷的便捷方式：

```Kotlin
fun SharedPreferences.edit(action: SharedPreferences.Editor.() -> Unit) {
    val editor = edit()
    action(editor)
    editor.apply()
}
```

然后我们就可以这么用：

```Kotlin
val sp = getSharedPreferences("name",  Context.MODE_PRIVATE)
sp.edit({
    putInt("key1", 0)
    putLong("key2, 1)
    putString("key3", "value")
})
```

没错，你不需要调用 `apply()` 或者 `commit()` 方法，因为我们的扩展方法里面做好了，你要做的就是存储数据。

# 9 妈妈再也不用担心我空指针了

空指针异常是我们经常遇到的问题，为什么会出现空指针？要嘛是忘记进行判空处理，要嘛就是忘记赋值。当然在开发过程中可以使用 @Nullable 注解来对可能为空的变量或方法返回值进行注解，但是这样当你没有进行判空的时候，也仅仅是收到 IDE 的一个警告信息而已，照样能够正常编译。

在 Kotlin 的语法里，你必须显示声明成员变量、方法参数、方法返回值是否可以为空，当你在使用一个可能为空的成员变量时没有进行判空处理，编译器直接就会报错，而不是形同虚设的警告。下面是一段简单的代码，通过 `?` 关键字声明成员变量 `name` 可能为空，然后我们在使用的时候必须判空：

```
var name: String? = null

if (name != null) {
    // do something
}
```

# 10 一句话实现单例模式

当例模式是我们经常用到的一个设计模式，其目的就是为了让全局只有一个对象实例，在 Java 里面我们会这么写：

```
public class Singleton {
    public static final Singleton INSTANCE = new Singleton();
    private Singleton() {}
}
```
在 Kotlin 里面则更加简单，只需要通关键字 `object` 就可以生命一个类为单例：

```
object Singleton {}
```

# 11 让你的代码仅在模块内可见

有过 SDK 开发经验的人应该会遇到这样一个问题，在开发过程中我们经常会将类按照功能进行分包，例如 A 属于com.xxx.pkga，B 属于 com.xxx.pkgb，但是我们又希望 A 的 `foo()` 方法可以被 B 访问，但是不被 SDK 之外的其他人访问，这种情况在 Java 里面是无解的，要嘛你声明 `foo()` 方法为 `public`，要嘛你把 A 和 B 放在一个包下，并声明 `foo()` 方法为 `default`，但是这两种做法都违背了我们的意愿。在 Kotlin 里面，专门提供了 `internal` 关键字，用于声明一个类或方法只有模块内部可见，所谓的模块在 Android Studio 里面就是一个 Gradle Module，而我们的 SDK 通常都是一个 Module，这就完美解决了上面描述的问题了。

# 12 比 // TODO 更狠的 TODO()

你是不是经常写了个 `// TODO` 注释然后放在那边几百年不动它？好吧这不能怪你，因为它只是一段注释而已，不会主动提示你**“快来把我解决掉！”**，而 Kotlin 给我提供了一个更给力的方式 `TODO()` 方法，用它在代码中标记那些你日后会去处理的逻辑，并且当你跑到这段没有完成逻辑处理的代码时，`TODO()` 会这样告诉你**“来啊！干掉我啊！不然我就让你崩溃！”**：

```java
fun foo() {
    TODO("来啊！干掉我啊！不然我就让你崩溃！")
}
```

其内部实现原理也很简单，就是给你抛异常，但是起码我不用自己写啦，而且它默认就是引入的：

```kotlin
/**
 * Always throws [NotImplementedError] stating that operation is not implemented.
 */
@kotlin.internal.InlineOnly
public inline fun TODO(): Nothing = throw NotImplementedError()
```
# 13 更加灵活的字符串定义 

用 Java 拼接字符串是不是很恶心？比如我们要拼接一串 SQL 查询语句，用 Java 会这么写：

```java
String table = ...
String id = ...
String email = ...
String age = ...
String sql = "SELECT * FROM " + table + " WHERE id=" + id + " AND name=" + name + " AND email=" + email + " AND age=" + age;  
```

使用一堆的 `+` 拼接起这个查询语句，它的可读性真是不敢恭维，而且写起来也不方便，我们用 Kotlin 的字符串模板试试：

```kotlin
val table = ...
val id = ...
val email = ...
val age = ...
val sql = "SELECT * FROM $table WHERE id=$id AND name=$name AND email=$email AND age=$age"
```

在字符串中的 `$table`、`$id`、`$name`、`$email` 和 `$age` 会被自动替换为同名变量的值，再也不需要使用一堆 `+` 去拼接字符串啦，如果你想让语句看起来更好看些，可以这么写：

```kotlin
val table = ...
val id = ...
val email = ...
val age = ...
val sql = """
          SELECT * FROM $table 
          WHERE id=$id 
          AND name=$name 
          AND email=$email 
          AND age=$age
          """
```

在 Kotlin 中字符串除了用 `"字符串"` 表示之外，还可以用 `"""字符串"""` 表示，这种方式让你在定义字符串的时候随意换行并且保留格式。

持续更新中...
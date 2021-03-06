---
title: 如何提高代码的可读性
date: 2017-05-16 22:43:28
categories:
- Android
tags:
---

“任何一个傻瓜都能写出计算机可以理解的代码，唯有写出人类容易理解的代码，才是优秀的程序员。”，这句话出自《重构》这本书，我个人很赞同这句话，原因如下：

*  代码不仅仅是人类和计算机沟通的语言，它也是建立在程序员之间的桥梁，两个程序员在沟通的时候，任何富有表达力的言语都不如直接阅读对方一段代码。
*  代码也是公司的一笔特殊财富，因为它不可能永远被同一个程序员维护，如果代码的可读性很差的话，很可能导致这笔财富无法传承下来，前功尽弃。
*  具有良好可读性的代码能让功能的扩展和BUG的修复更顺利，这一点应该很容易理解，增加新功能、修改某个BUG都需要你首先理解代码。

所以，提高代码可读性是很有必要的，本文将介绍个人在实践中认为能够提高代码可读性的方法，希望对大家有所帮助。

# 1 重视代码规范

重视代码规范是提高代码可读性最基本和最简单的方式，每一门语言，每一个公司或组织都会有自己的代码规范，制定这些规范的目的就是为了让大家能够更容易阅读和理解代码。例如当我们看到某一个变量名称前面带有 `m` 前缀时，我们就知道它是一个成员变量（member）；当我们看到某个方法以 `on` 开头时，我们就知道它是一个回调方法；当我们看到某个变量是由大写字母和下划线组成时，我们就知道它是一个静态常量。诸如此类的规范有很多，只要我们遵循规范，我们的代码可读性起码会有一个最低的保障。

# 2 多写注释

注释本应作为代码不可分割一部分，因为它是对代码最直观最详细的说明，你可以把设计思路、用法和注意事项都写在注释里面，这样无论是对你自己还是对别人都是有好处的，它让你能在很久没有接触代码的时候快速回忆起当初的想法，能让阅读源码的人更快理解你的思路，让使用的人更清楚用法。

例如我们有一个设置年龄的方法叫 `setAge(int age)`，从方法名称上我们就可以知道它的作用，但是其内部还做了一些其他处理，这些处理是方法命无法体现的，所以加上一段注释说明就很有必要了：

```java
/**
 * 设置用户的年龄，当 age < 0 的时候会设置 age = 0，当 age > 100 的时候会设置 age = 100。
 * 另外你可以通过 {@link #getAge()} 方法获取用户的年龄。
 *
 * @param age 用户年龄[0, 100]
 * @see #getAge()
 */
public void setAge(int age) {
    if (age < 0) {
        age = 0;
    } else if (age > 100) {
        age = 100;
    }
    this.age = age;
}
```

# 3 重写注释

当我们重写某个方法并且修改了方法的逻辑导致它的行为与原有的注释描述不一致的时候，我们应该对注释也进行重写，以确保注释内容和代码逻辑一致。例如上面的提到的 `setAge(int age)` 方法，我们对它进行重写，改成接收任意数值的年龄，此时原有的注释内容就需要修改，因为它已经不适用于现在的逻辑：

```java
/**
 * 设置用户的年龄，另外你可以通过 {@link #getAge()} 方法获取用户的年龄。
 *
 * @param age 任意数值的用户年龄
 * @see #getAge()
 */
public void setAge(int age) {
    this.age = age;
}
```

# 4 使用注解

注解可以用于替换一些简短的注释描述，并且提高变量、方法和类的可读性，最典型的例子就是 `@Override` 注解，它告诉我们被注解的方法是对父类或者接口方法的重写或实现，相比于注释“这是一个重写方法”，注解 `@Override` 要更简单快捷得多。例如下面的代码我们实现了 `OnClickListener` 接口的 `onClick(View view)` 方法：

```java
public class MyListener implements View.OnClickListener {
    
    // 实现 onClick 方法。
    @Override
    public void onClick(View view) {
    
    }
    
}
```

大多数人只会使用一些现有的注解，而很少自己创建注解，其实我们可以自己创建一些有用的注解来提高代码的可读性，下面我们以 [EventBus](https://github.com/greenrobot/EventBus) 为例，说明自定义注解是如何提高代码可读性的。

EventBus 是一个应用在 Android 上的事件总线，我们可以使用它在任意地方发布事件，并且在任意地方注册并接收事件。EventBus 有三个弊端，一是我们经常搞不清楚某一个事件是来自哪里的，因为任何地方都可以发送同一个事件；二是我们同样也经常搞不清楚某个事件会在哪些地方被接收，因为任何地方都可以注册并接收事件；三是早期的 EventBus 事件接收方法要求你必须以 `onEventXXXX` 的方式命名，这样的命名其实和普通方法并没有太大的区别，我们需要一种方式让它更加凸显，以防它被误以为是普通方法而被删除。这时候我们可以利用注解让事件接收方法一眼就被认出来，并且还能看出有哪些地方会发出该事件，哪些地方会接收到我们发出的事件：

```java
/**
 * EventBus 事件注解，用于标识某个方法是 EventBus 方法，并且注明事件
 * 的接收方或发送方。
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Event {
    Class[] from() default {};
    Class[] to() default {};
}
```

现在我们假设 A 和 B 都可以发送一个名叫 `MyEvent` 的事件到 C 和 D，我们可以这样写：

```java
public class A {
    
    @Event(to = {C.class, D.class})
    public void sendEvent() {
        EventBus.getDefault().post(new MyEvent());
    }
    
}
```

```java
public class B {
    
    @Event(to = {C.class, D.class})
    public void sendEvent() {
        EventBus.getDefault().post(new MyEvent());
    }
    
}
```

```java
public class C {
    
    @Event(from = {A.class, B.class})
    public void onEventMainThread(MyEvent event) {
        // do something
    }
    
}
```

```java
public class D {
    
    @Event(from = {A.class, B.class})
    public void onEventMainThread(MyEvent event) {
        // do something
    }
    
}
```

从上面的代码，我们通过 `@Event` 注解一眼就认出来 EventBus 方法，并且还能知道 A 将事件发送到了 C 和 D，而 C 接收到的事件可能来自 A 或 B。

# 5 多用 @Nullable，慎用 @NonNull

【@Nullable】是一个用于标注某个变量、参数或方法返回值可能为空指针的注解，当我们看到该注解的时候就要小心了，它意味着如果我们不做判空处理的话，很可能出现空指针异常，并且在 Android Studio 上会以黄色背景警告我们。那么什么时候该使用该注解呢？我的原则只要某个变量、参数或方法返回值有一丝可能性为空，就使用该注解，这起码可以避免空指针异常的出现。

{% asset_img nullable.png %}

---

【@NonNull】是一个用于标注某个变量、参数和方法返回值不可能为空指针的注解，当我们看到该注解的时候，就可以认为被标记的变量、参数或方法返回值不可能为空，无需做判空处理，同时 Android Studio 也会在你进行判空处理的时候提醒你没有必要这样做。对于这个注解，我个人的原则是除非100%肯定不会空指针，否则绝对不用。

{% asset_img nonnull.png %}

# 6 使用 @MainThread 和 @WorkerThread

【@MainThread】是一个用于标注某个类或方法必须在主线程中使用的注解，当我们看到该注解的时候，就要注意当前的操作是否处于主线程，否则很有可能出错。例如我们有个方法用于更新 `TextView` 的内容，因为涉及到 UI 的更新操作，必须在主线程进行，所以我们可以这么写：

```java
@MainThread
public void updateTitle(String title) {
    mTvTitle.setText(title);
}
```

---

【@WorkerThread】是一个用于标注某个类或方法必须在子线程中使用的注解，常见的情况就是不适合在主线程中进行的耗时操作，当我们看到该注解的时候，应该创建一个子线程去使用被注解的类或方法。例如我们有个方法用于将 `Bitmap` 保存到 SD 卡中，涉及到 I/O 的操作理应在子线程中进行，所以我们可以这么写：

```java
@WorkerThread
public void saveImage(Bitmap image, String path) throw IOException {
    FileOutputStream outputStream = new FileOutputStream(path);
    image.compress(Bitmap.CompressFormat.JPEG, 100, outputStream);
}
```

# 7 使用 return 减少 if 嵌套

当我们的方法中有一系列业务逻辑是按顺序执行，并且每一个业务逻辑的执行前提是前一个业务逻辑执行成功时，我们可以考虑使用 `return` 关键字在业务逻辑执行失败时终止整个方法，而不是嵌套多层的 `if`。例如，我们有一个返回只为 `boolean` 类型的方法，它只有在方法内的所有业务逻辑都按顺序执行成功之后才返回 `true`： 

```java
public boolean func() {
    boolean isSuccess;
    isSuccess = doSomething1();
    if (isSuccess) {
        isSuccess = doSomething2();
        if (isSuccess) {
            isSuccess = doSomething3();
            if (isSuccess) {
                isSuccess = doSomething4();
                if (isSuccess) {
                    isSuccess = doSomething5();
                }
            }
        }
    }
    return isSuccess;
}
```

对于这种多重 `if` 嵌套的代码，我们可以在某个业务逻辑执行失败的时候 `return false` 来终止整个方法，这样做的好处是让人一看就知道里面的逻辑是从上到下按顺序执行的，更容易理解，具体写法如下：

```java
public boolean func() {
    boolean isSuccess;
    isSuccess = doSomething1();
    if (!isSuccess) {
        return false;
    }

    isSuccess = doSomething2();
    if (!isSuccess) {
        return false;
    }

    isSuccess = doSomething3();
    if (!isSuccess) {
        return false;
    }

    isSuccess = doSomething4();
    if (!isSuccess) {
        return false;
    }

    isSuccess = doSomething5();
    if (!isSuccess) {
        return false;
    }

    return isSuccess;
}
```

# 8 使用 Map 代替分支语句

当我们需要使用条件语句语句根据不同的条件筛选出对应的结果，并且条件很多导致语句很长时，可以考虑使用 `Map` 代替冗长的条件判断，例如有一个方法需要根据用户输入的索引值返回对应的字母（我们不考虑算法），我们以下三种写法：

**1 使用 if 语句**

```java
public char getLetter(int index) {
    if (index == 1) {
        return "A";
    } else if (index == 2) {
        return "B";
    } else if (index == 3) {
        return "C";
    } else if (index == 4) {
        return "D";
    }
    ...
}
```

使用 `if` 语句进行大量的条件筛选是最糟糕的设计，一方面代码编写麻烦，另一方面代码长度也是最长的。

---

**2 使用 switch 语句**

```java
public char getLetter(int index) {
    switch (index) {
        case 1: return "A";
        case 2: return "B";
        case 3: return "C";
        case 4: return "D";
        ...
    }
}
```

使用 `switch` 语句是最常见的方式，通过不同的 `case` 筛选出对应的结果，并且逻辑清晰，代码量也相对较少。

---

**3 使用 Map**

```java
private static final Map<Integer, String> LETTERS = new HashMap<>();

static {
    LETTERS.put(1, "A");
    LETTERS.put(2, "B");
    LETTERS.put(3, "C");
    LETTERS.put(4, "D");
    ...
}

public char getLetter(int index) {
    return LETTERS.get(index);
}
```

使用 `Map` 代替 `switch` 本质上没有太大的区别，但是有些时候它可以让我们省去一个方法，例如我们完全可以省去 `getLetter(int)` 方法而直接通过 `Map` 获取想要的值。

# 9 方法调用的语法糖

方法调用的语法糖要求一个或多个方法的调用能够形成具有良好可读性的语句，我们通过几个简单的例子看下什么样的方法调用能够形成可以阅读的语句：

**1 Android ValueAnimator 实例创建**

```java
ValueAnimator animator = ValueAnimator.ofInt(0, 100);
```

我们将 `ValueAnimator.ofInt` 拆解开来会发现它其实就是**“value animator of int”**，也就是**“整型类型的属性动画”**。

---

**2 Android AnimatorSet 动画播放顺序**

```java
AnimatorSet animatorSet = new AnimatorSet();
animatorSet.play(anim1).width(anim2).before(anim3);
```

AnimatorSet 是一个能够将多个动画组合在一起并且指定动画播放顺序的工具类，上面的方法调用方式很清晰的告诉我们**“在播放 anim3 之前先同时播放 anim1 和 anim2”**。

---

**3 Mockito 打桩方法**

```java
when(person.getAge()).thenReturn(20);
```

Mockito 是一个用于单元测试的框架，这里我们不探究它的用法，我们要看的是它的方法调用形式。上面的代码同样可以拆解成**“when person.getAge() then return 20”**，意思就是**“当调用person.getAge()方法的时候，返回20”**。

# 10 使用 Builder 代替构造方法

当某个类的构造方法有很多个参数或者有很多个重载版本时，我们应该考虑为这个类写一个 Builder，通过这个 Builder 创建配置并创建该类的实例。

假设我们有个类用来代表一个矩形，它的名字叫做 Rectangle，它的代码如下所示：

```java
public class Rectangle {

    private int mId;
    private int mWidth;
    private int mHeight;
    private int mStroke;// 边框宽度
    
    public Rectangle(int id) { ... }
    
    public Rectangle(int id, int width, int height) { ... }
    
    public Rectangle(int id, int width, int height, int stroke) { ... }

}
```

于是我们就可能看见这样的代码 `new Rectangle(1, 1, 1, 1)`，这样的代码可读性是很差的，因为我们无法一眼就看出这个矩形设置了哪些信息，还需要去查阅下相关的 API 文档。此外，当我们需要创建一个只需指定 `id` 和 `stroke` 的矩形的时候，我们就必须再写一个新的构造方法，当一个对象的属性较多的时候，构造方法的重载版本就可能变得非常的多，维护成本也随之提高。如果我们为 Rectangle 创建一个 Builder，通过 Builder 创建矩形实例的过程就会变得灵活而清晰很多：

```java
public class Rectangle {

    private int mId;
    private int mWidth;
    private int mHeight;
    private int mStroke;// 边框宽度
    
    private Rectangle(Builder builder) {
        mId = builder.id;
        mWidth = builder.width;
        mHeight = builder.height;
        mStroke = builder.stroke;
    }
    
    public static class Builder {
    
        private int id;
        private int width;
        private int height;
        private int stroke;// 边框宽度
        
        public Builder(int id) {
            this.id = id;
        }
        
        public Rectangle build() {
            return new Rectangle(this);
        }
        
        public Builder width(int width) {
            this.width = width;
            return this;
        }
        
        public Builder height(int height) {
            this.height = height;
            return this;
        }
        
        public Builder stroke(int stroke) {
            this.stroke = stroke;
            return this;
        }
            
    }

}
```

现在，我们创建 Rectangle 实例的过程就会变成下面的样子，它不仅让我们一眼就看出矩形设置了哪些属性，而且我们还可以自由组合这些属性，达到重载构造方法想要的结果：

```java
Rectangle.Builder builder1 = new Rectangle.Builder(1);
Rectangle rect1 = builder1.width(1).height(1).stroke(1).build();

Rectangle.Builder builder2 = new Rectangle.Builder(2);
Rectangle rect2 = builder2.width(1).height(1).build();

Rectangle.Builder builder3 = new Rectangle.Builder(3);
Rectangle rect3 = builder3.stroke(1).build();
```

# 11 使用 tools 优化布局预览

【tools】是 Android 里进行布局排版时的一个工具，它用于在布局预览的时候设置临时属性。我们经常需要一边编写布局代码一边查看预览以确保布局正确，现在假设我们有一个 `TextView` 用来显示标题，但是标题的内容是在运行的时候动态设置的，为了开发的时候方便预览，提高复杂布局代码的可读性，我们可能会这么写：

```xml
<TextView
    android:text="测试标题"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" />
```

你一定想着等后面确定布局没问题的时候再删掉测试文案，但是事实上你很有可能忘记删除，这时候我们就可以利用 tools 这样写：

```xml
<TextView
    tools:text="测试标题"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" />
```

通过 tools 设置的文案只会在预览的时候生效，不会影响到实际运行的情况，这样我们可以毫无顾忌地随便填写测试文案了。tools 支持的属性还有很多，例如：

* tools:background
* tools:visibility
* tools:checked
* tools:src

关于更多 tools 的用法，大家可以到网上查找。

# 12 使用 isInEditMode() 优化布局预览

`isInEditMode()` 方法是用于判断 View 当前是否处于 IDE 布局编辑（预览）状态，只有在编辑状态下才会返回 `true`，当我们编写只有在运行时才能看到绘制效果的自定义 View 的时候，可以使用 `isInEditMode()` 方法让 View 在布局预览的时候就看到运行时的大概样子。例如我们通过 View 实现一个圆形不断放大缩小的动画时，正常情况我们只有在程序运行的时候才能看到动画效果，在布局预览的时候是空白一片的，我们可以通过 `isInEditMode()` 方法在编辑的时候先绘制一个圆形，让开发者大概知道这个动画 View 会是什么样子，虽然它是静止的，但是也好过一片空白：

```java
public class AnimationView extends View {
    
    @Override
    public void onDraw(Canvas canvas) {
        if (isInEditMode()) {
            // 编辑状态下绘制一个圆形，让开发者大概知道圆形的大小。
            canvas.drawCircle(centerX, centerY, radius, paint);
        } else {
            // 运行时刷新画面。
        }
    }
    
}
```

我们在布局预览的时候就会看到一个圆形被绘制出来，并且在程序正在运行的时候不会绘制该圆形：

{% asset_img preview-in-edit-mode.png %}

> 提示：实际开发中，`isInEditMode()` 方法可以用在 View 的任何地方用于设置预览数据。

# 13 使用内部类对代码进行分类

内部类有些时候可以起到类似包的代码分类功能，通过内部类对某个类内部的代码进行功能划分，可以让与该类相关的代码更具有可读性。

例如我们有个日志打印工具类，里面有两种类型的方法，一种是打印用户可见的日志，另一种则是打印只有开发者可见的日志，我们可以这样写：

```java
public class Logger {
    
    // 用户可见的日志。
    public class User {
        public static void d(String tag, String msg) {
            // 打印日志
        }
    }
    
    // 开发者可见的日志。
    public class Developer {
        public static void d(String tag, String msg) {
            // 打印日志
        }
    }
    
}
```

我们在调用日志打印方法时通过内部类的名称一眼就可以看出当前打印的日志是用户可见还是开发者可见的：

```java
Logger.User.d(TAG, "打印用户可见的日志");
Logger.Developer.d(TAG, "打印开发者可见的日志");
```

类似的，我们也可以使用内部类对常量进行分类，例如我们有一个多媒体数据库叫 MediaStore，其内部分为图像数据表和视频数据表，每个表都有自己的字段，我们需要定义常量来对应这些字段，所以可以这样写：

```java
public class MediaStore {
    
    // 图像数据表
    public class ImageTable {
        public static final String NAME = "image_name";
        public static final String PATH = "image_path";
    }
    
    // 视频数据表
    public class VideoTable {
        public static final String NAME = "video_name";
        public static final String PATH = "video_path"; 
    }
    
}
```

我们在需要使用这些常量的时候就可以通过内部类区分不同类型的常量了：

```java
MediaStore.ImageTable.NAME// 图像名称字段
MediaStore.VideoTable.NAME// 视频名称字段
```
# kotlin学习笔记

## 属性委托

### java实现委托

```java
interface Behavior {  
    void doSomething();  
}  
  
class RealBehavior implements Behavior {  
    @Override  
    public void doSomething() {  
        System.out.println("Doing something real");  
    }  
}  
  
class DelegatingClass implements Behavior {  
    private final Behavior delegate;  
  
    public DelegatingClass(Behavior delegate) {  
        this.delegate = delegate;  
    }  
  
    @Override  
    public void doSomething() {  
        delegate.doSomething(); // 委托给delegate对象  
    }  
}  
  
// 使用  
Behavior realBehavior = new RealBehavior();  
Behavior delegatingBehavior = new DelegatingClass(realBehavior);  
delegatingBehavior.doSomething(); // 输出: Doing something real
```

### 什么是属性委托？

在kotlin中，属性委托是一种将属性的访问（获取和设置）转发给另一个对象（通常是某个字段或另一个类的实例）的机制。这样，当你访问或修改这个属性时，实际上是在访问或修改背后的那个对象。

### 为什么需要属性委托？

属性委托允许我们实现一些设计模式，如懒加载（Lazy Initialization）、可观察属性（Observable Properties）、验证属性（Validated Properties）等，而不需要在每次访问属性时都重复编写相同的代码。

### 如何实现属性委托？

kotlin提供了`by`关键字来实现属性委托。你可以将属性的访问委托给另一个对象或委托属性（Delegate Property）。这个委托对象或属性需要提供一些方法来处理属性的获取（`getValue`）和设置（`setValue`，对于var属性）。

```kotlin
class Delegate(private val value: String) {  
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String = value  
    operator fun setValue(thisRef: Any?, property: KProperty<*>, newValue: String) {  
        // 这里可以添加设置值前的验证或其他逻辑  
        println("Setting $property.name to $newValue")  
        // 但在这个例子中，我们不改变实际的值，因为它是在构造函数中定义的  
    }  
}  
  
class Example {  
    var delegatedProperty: String by Delegate("Initial value")  
}  
  
fun main() {  
    val example = Example()  
    println(example.delegatedProperty) // 输出: Initial value  
    // setValue在这里不会被实际调用，因为Delegate类的实现中没有改变value的值  
    // 但它展示了如何在设置值时执行代码  
}

```

## Kotlin 对 Jetpack Compose 的支持
### 高阶函数和lambda表达式

Kotlin 支持[高阶函数](https://kotlinlang.org/docs/reference/lambdas.html)，即接收其他函数作为参数的函数。Compose 在此方法的基础上构建而成。例如，[`Button`](https://developer.android.google.cn/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#Button(kotlin.Function0,androidx.compose.ui.Modifier,kotlin.Boolean,androidx.compose.foundation.interaction.MutableInteractionSource,androidx.compose.material.ButtonElevation,androidx.compose.ui.graphics.Shape,androidx.compose.foundation.BorderStroke,androidx.compose.material.ButtonColors,androidx.compose.foundation.layout.PaddingValues,kotlin.Function1)) 可组合函数提供了一个 `onClick` lambda 参数。该参数的值是一个函数，当用户点击按钮时，按钮会调用该函数：

```kotlin
Button(
    // ...
    onClick = myClickFunction
)
// ...
```

高阶函数与 lambda 表达式（即求得的值是函数的表达式）自然配对。如果您只需要该函数一次，则不必在其他位置进行定义以将其传递给高阶函数，而只需使用 lambda 表达式在该位置定义该函数。前面的示例假设在其他位置定义了 `myClickFunction()`。但是，如果您只在此处使用该函数，则使用 lambda 表达式以内嵌方式定义该函数会更简单：

```kotlin
Button(
    // ...
    onClick = {
        // do something
        // do something else
    }
) { /* ... */ }
```

#### 尾随lambda

Kotlin 提供了一种特殊语法来调用最后一个参数为 lambda 的高阶函数。如果您要将一个 lambda 表达式作为该参数传递，您可以使用尾随 lambda 语法。您应将 lambda 表达式放在圆括号后面，而不是将其放在圆括号内。这是 Compose 中的一种常见情况，因此您需要熟悉代码是什么样子的。

例如，所有布局的最后一个参数（如 [`Column()`](https://developer.android.google.cn/reference/kotlin/androidx/compose/foundation/layout/package-summary?hl=zh-cn#Column(androidx.compose.ui.Modifier,androidx.compose.foundation.layout.Arrangement.Vertical,androidx.compose.ui.Alignment.Horizontal,kotlin.Function1)) 可组合函数）均为 `content`，它是一个发出子界面元素的函数。假设您想要创建一个包含三个文本元素的列，并且需要应用某种格式设置。以下代码行得通，但它非常繁琐：

```kotlin
Column(
    modifier = Modifier.padding(16.dp),
    content = {
        Text("Some text")
        Text("Some more text")
        Text("Last text")
    }
)
```

由于 `content` 参数是函数签名中的最后一个参数，并且我们要将其值作为 lambda 表达式传递，因此我们可以将其从圆括号中取出：

```kotlin
Column(modifier = Modifier.padding(16.dp)) {
    Text("Some text")
    Text("Some more text")
    Text("Last text")
}
```

这两个示例的含义完全相同。大括号定义传递给 `content` 参数的 lambda 表达式。

事实上，如果您要传递的唯一一个参数是该尾随 lambda，也就是说，如果最后一个参数是 lambda，并且您不会传递其他任何参数，则您可以完全省略圆括号。例如，假设您不需要将修饰符传递给 `Column`。您可以像下面这样编写代码：

```kotlin
Column {
    Text("Some text")
    Text("Some more text")
    Text("Last text")
}
```

此语法在 Compose 中十分常见，尤其是对于诸如 `Column` 之类的布局元素。最后一个参数是一个 lambda 表达式，它用于定义元素的子元素，这些子元素在函数调用后在大括号中指定。

## 定义构造函数

### 默认构造函数

默认构造函数不含形参。定义默认构造函数的做法如以下代码段所示：

```kotlin
class SmartDevice constructor() {
    ...
}
```

Kotlin 旨在简化代码，因此，如果构造函数中没有任何注解或可见性修饰符，您可以移除 `constructor` 关键字。如果构造函数中没有任何形参，您还可以移除圆括号，如以下代码段所示：

```kotlin
class SmartDevice {
    ...
}
```

Kotlin 编译器会自动生成默认构造函数。您不会在自己的代码中看到自动生成的默认构造函数，因为编译器会在后台进行添加。

### 定义形参化构造函数

```kotlin
class SmartDevice(val name: String, val category: String) {
    
}
```

---

```kotlin
class SmartTvDevice(deviceName: String, deviceCategory: String) :
    SmartDevice(name = deviceName, category = deviceCategory) {
}
```

`SmartTvDevice` 的 `constructor` 定义没有指定属性是可变的还是不可变的，这意味着，`deviceName` 和 `deviceCategory` 形参只是 `constructor` 形参，而不是类属性。您无法在类中使用这些形参，只能将其传递给父类构造函数。

## 类与类之间的关系

### IS-A关系（继承）

如果在 `SmartDevice` 父类和 `SmartTvDevice` 子类之间指定 IS-A 关系，即表示 `SmartDevice` 父类可以执行的操作，`SmartTvDevice` 子类也可执行。这种关系是单向的，因此可以说每个智能电视“都是”智能设备，但不能说每个智能设备“都是”智能电视。IS-A 关系的代码表示形式如以下代码段所示：

```kotlin
// Smart TV IS-A smart device.
class SmartTvDevice : SmartDevice() {
}
```

请不要只为了实现代码的可重用性而使用继承。在做出决定之前，请检查这两个类彼此是否相关。如果两者表现出某种关系，请检查是否确实符合 IS-A 关系的定义。不妨问问自己，“我可以说子类是父类吗？”例如，Android“是”操作系统。

### HAS-A关系（组合）

HAS-A 关系是指定两个类之间的关系的另一种方式。例如，您可能要使用住宅中的智能电视。在这种情况下，智能电视和住宅之间存在某种关系。住宅中包含智能设备，即住宅“拥有”智能设备。两个类之间的 HAS-A 关系也称为“组合”。

```kotlin
// The SmartHome class HAS-A smart TV device.
class SmartHome(val smartTvDevice: SmartTvDevice) {
    fun turnOnTv() {
        smartTvDevice.turnOn()
    }

    fun turnOffTv() {
        smartTvDevice.turnOff()
    }
}
```

## Kotlin的可见性修饰符

Kotlin默认的可见性是public

### 为属性指定可见性修饰符

语法：

```kotlin
modifier var name: dataType =  initialValue
```

也可以将可见性修饰符设置为 setter 函数

```kotlin
var name: dataType =  initialValue
	modifier set(value) {
        body
    }
```

不会在 `set()` 函数中执行任何操作或检查，只需将 `value` 形参赋给 `field` 变量时，您可以省略 `set()` 函数的圆括号和主体：

```kotlin
var name: dataType =  initialValue
	modifier set

//===================================
open class SmartDevice(val name: String, val category: String) {

    ...

    var deviceStatus = "online"
        protected set

    ...
}
```

### 为方法指定可见性修饰符

语法：

```kotlin
modifier fun name() {
    body
}
```

### 为构造函数指定可见性修饰符

语法：

```kotlin
class name modifier constructor(parameters) {
    body
}
```

### 为类指定可见性修饰符

语法：

```kotlin
modifier class name {
    body
}
```



|   修饰符    | 可在相同类中访问 | 可在子类中访问 | 可在相同模块中访问 | 可在模块之外访问 |
| :---------: | ---------------- | -------------- | ------------------ | ---------------- |
|  `private`  | √                | ×              | ×                  | ×                |
| `protected` | √                | √              | ×                  | ×                |
| `internal`  | √                | √              | √                  | ×                |
|  `public`   | √                | √              | √                  | √                |


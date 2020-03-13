# Kotlin基本语法

## 函数定义

表达式作为函数体，返回类型自动推断

```kotlin
fun sum(a:Int, b:Int) = a + b
// public方法则必须明确写出返回值类型（貌似不需要）
public fun sum(a:Int, b:Int):Int = a + b
```

无返回值的函数

```kotlin
// 无论是否为public方法，若返回Unit类型，则都可省略
fun printSum(a:Int, b:Int): Unit {
    print(a + b)
}
```

可变长参数函数，可用`vararg`关键字进行标识

```kotlin
fun vars(vararg v:Int) {
    for(vt in v) {
        print(vt)
    }
}
```

lambda(匿名函数)

```kotlin
fun main(args: Array<String>) {
    val sumLamdba = (Int, Int) -> Int = {x, y -> x+y}
    println(sumLamdba(1, 2))
}
```

## 定义常量与变量

- 可变变量定义使用`var`关键字
- 不可变变量定义使用`val`关键字
- 如果不在声明时初始化变量的值，则必须提供变量类型

## NUll检查机制

Kotlin的空安全设计对于声明可为空的参数，在使用时要进行空判断处理。处理有两种方式：字段后加 `!!` 像Java一样抛出空异常，另一种字段后加 `?` 可不做处理返回null或配合 `?:` 做空判断处理、

```kotlin
// 不做处理返回null
val age1 = age?.toInt()
// age为空返回-1
val age2 = age?.toInt() ?: -1
```

## 区间

区间表达式由具有操作符形式 `..` 的rangeTo函数辅以`in`与`!in`形成。

```kotlin
for(i in 1..4) print(i)		// 输出“1234”
for(i in 4..1) print(i)		// 没有输出

// 使用step指定步长
for(i in 1..4 step 2) print(i)	// 输出“13”
for(i in 4 downTo 1 step 2) print(i)	// 输出“42”

// 使用until函数排除结束元素
for(i in 1 until 10) print(i)	// 输出i in [1, 10)
```

# Kotlin基本数据类型

## 字面常量

- Kotlin中字符不属于数值类型，是一个独立的数据类型。
- 长整型一大写字符`L`结尾，Floats使用`f`或者`F`结尾
- 16进制以`0x`开头，2进制以`0b`开头，不支持8进制
- 数字常量间可使用下划线连接提高可读性，例如: `1_000_000`

## 比较两个数字

Kotlin中没有基础数据类型，只有封装的数字类型，每定义一个变量，Kotlin都会自动封装成一个对象。因此，包括数字类型，在做比较时，就有比较数据大小和比较两个对象是否相同的区别了。Kotlin中，使用`===`比较对象的地址，使用`==`比较值的大小。

## 类型转化

由于不同的表示方式，较小的类型并不是较大类型的子类型，故不能进行隐式转换。

```kotlin
val b: Byte = 1 	// 正确，字面值是静态检测的
val i: Int = b		// 错误
val i: Int = b.toInt()	// 显示转换
```

有些情况下也是可以进行自动类型转换的，前提是可以根据上下文环境推断出正确的数据类型而且数学操作符会做相应的重载。

```kotlin
val l = 1L + 3 		// Long + Int -> Long
```

## 字符

和Java不同，Kotlin中的Char不能直接和数字操作。

```kotlin
fun decimalDigitValue(c: Char): Int {
    if(c !in '0'..'9')
    	throw IllegalArgumentException("Out of range")
    return c.toInt() - '0'.toInt()
}
```

## 数组

数组类用Array实现，并且有一个size属性及get和set方法，并且使用`[]`做了重载，因此可以直接通过下标获取。

数组创建的两种方式：一种是使用函数`arrayOf()`；另一种是使用工厂函数。

```kotlin
fun main(args: Array<String>) {
    val a = arrayOf(1, 2, 3)	// [1, 2, 3]
    val b = Array(3, {i -> (i * 2)})	// [0, 2, 4]
}
```

## 字符串

- Kotlin支持`'''`括起来的字符串，支持多行字符串
- String可通过`trimMargin()`方法来删除多余的空白。默认`|`作为边界前缀，但我们可以选择其他字符并作为参数传入。例如:`trimMargin(">")`


## 量

- 变量：`var`

- 不可变量：`val`
  - 变量和不可变量在字节码文件中是相同的，不可变量并不是`final`类型。
  - 不可变量不能有`set`方法，但在类内部是可以改变的
  - 效率高于java：如果`val`量长期不会被用到的时候，`gc`是会回收的。

- 常量：`const var`
  - 相当于`public static final`
  - 只有全局保证一定不会变化的量才会使用`const var`

```kotlin
const val a:String = "456789"

fun main(args: Array<String>) {

//    val hello = Hello()
//    println(hello.string)
//    hello.string = "Want"
//    println(hello.string)

    val people = People(1985)
    println(people.age)
    people.thenextYear()
    println(people.age)
}

class People(var birth:Int) {
    val age:Int
        get() {
            return Calendar.getInstance()
                    .get(Calendar.YEAR) - birth
        }

    fun thenextYear() {
        birth = birth - 2
    }
}

class Hello {
    // String?与String为两个类型，String?为可空类型
    var string:String? = null
        get() {
            return field + "hello"
        }
        set(value) {
            field = value + "set"
        }

    val string2:String = "789"
        get() {
            return field
        }
}
```

## 空安全实现

- 在编译期通过静态代码检查

- 引用对象前做判空操作
  - 局部变量会有上下文空预判（java没有）

## 单例模式

- 饿汉式

  ```kotlin
  object KSingleton
  ```

- 懒汉式

  ```kotlin
  class KSingleton private constructor() {
      // 伴生对象，没有静态内部类
      companion object {
          private var instance:KSingleton? = null
              get() {
                  if (field == null) {
                      field = KSingleton()
                  }
                  return field
              }
  
          fun get():KSingleton {
              return instance!!
          }
      }
  }
  ```

- 线程安全的懒汉式

  ```kotlin
  class KSingleton private constructor() {
      companion object {
          private var instance:KSingleton? = null
              get() {
                  if (field == null) {
                      field = KSingleton()
                  }
                  return field
              }
  
          @Synchronized
          fun get():KSingleton {
              return instance!!
          }
      }
  }
  ```

- 双重锁

  ```kotlin
  class KSingleton private constructor() {
      companion object {
          val instance:KSingleton by lazy(mode = LazyThreadSafetyMode.SYNCHRONIZED) {
              KSingleton()
          }
      }
  }
  ```

- 静态内部类

  ```kotlin
  class KSingleton private constructor() {
      companion object {
          val instance = SingtonHolder.holder
      }
      
      private object SingtonHolder {
          val holder = KSingleton
      }
  }
  ```
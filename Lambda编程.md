# Lambda编程

Lambda表达式的语法结构：{参数名1: 参数类型, 参数名2: 参数类型 -> 函数体}

函数体中可以编写任意行代码，并且最后一行代码会自动作为Lambda表达式的返回值。

maxBy就是一个普通的函数而已，只不过它接收的是一个Lambda类型的参数，并且会
在遍历集合时将每次遍历的值作为参数传递给Lambda表达式。maxBy函数的工作原理是根据
我们传入的条件来遍历集合，从而找到该条件下的最大值

```kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape", "Watermelon")
val lambda = { fruit: String -> fruit.length }
val maxLengthFruit = list.maxBy(lambda)
```

## 简化过程

首先，我们不需要专门定义一个lambda变量，而是可以直接将lambda表达式传入maxBy函数
当中，因此第一步简化如下所示：

val maxLengthFruit = list.maxBy({ fruit: String -> fruit.length })

然后Kotlin规定，当Lambda参数是函数的最后一个参数时，可以将Lambda表达式移到函数括
号的外面，如下所示：
val maxLengthFruit = list.maxBy() { fruit: String -> fruit.length }
接下来，如果Lambda参数是函数的唯一一个参数的话，还可以将函数的括号省略：
val maxLengthFruit = list.maxBy { fruit: String -> fruit.length }
这样代码看起来就变得清爽多了吧？但是我们还可以继续进行简化。由于Kotlin拥有出色的类型
推导机制，Lambda表达式中的参数列表其实在大多数情况下不必声明参数类型，因此代码可
以进一步简化成：
val maxLengthFruit = list.maxBy { fruit -> fruit.length }
最后，当Lambda表达式的参数列表中只有一个参数时，也不必声明参数名，而是可以使用it
关键字来代替，那么代码就变成了：
val maxLengthFruit = list.maxBy { it.length }

# 标准函数

## 标准函数with、run和apply

### with

with函数接收两个参数：第一个参数可以是一个任意类型的对象，第二个参数是一个Lambda表达式。with函数会在Lambda表达式中提供第一个参数对象的上下文，并使用Lambda表达式中的最后一行代码作为返回值返回。

```kotlin
 val result = with(obj) {
 // 这里是obj的上下文
 "value" // with函数的返回值
}
```

它可以在连续调用同一个对象的多个方法时让代码变得更加精简。

首先我们给with函数的第一个参数传入了一个StringBuilder对象，那么接下来整个Lambda表达式的上下文就会是这个StringBuilder对象。于是我们在Lambda表达式中就不用调用builder.append()和builder.toString()方法了，而是可以直接调用append()和toString()方法。Lambda表达式的最后一行代码会作为with函数的返回值返回，最终我们将结果打印出来。

```kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape")
val result = with(StringBuilder()) {
 append("Start eating fruits.\n")
 for (fruit in list) {
 append(fruit).append("\n")
 }
 append("Ate all fruits.")
 toString()
}
println(result)
```

### apply

apply函数和run函数也是极其类似的，都要在某个对象上调用，并且只接收一个Lambda参数，也会在Lambda表达式中提供调用对象的上下文，但是apply函数无法指定返回值，而是会自动返回调用对象本身。

```kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape")
val result = StringBuilder().apply {
 append("Start eating fruits.\n")
 for (fruit in list) {
 append(fruit).append("\n")
 }
 append("Ate all fruits.")
}
println(result.toString())
```

# 高阶函数

如果一个函数接收另一个函数作为参数，或者返回值的类型是另一个函数，那么该函数就称为高阶函数

## 函数类型

(String, Int) -> Unit

->左边的部分就是用来声明该函数接收什么参数的，多个参数之间使用逗号隔开，如果不接收任何参数，写一对空括号就可以了。而->右边的部分用于声明该函数的返回值是什么类型，如果没有返回值就使用Unit，它大致相当于Java中的void。

## 用途

高阶函数允许让函数类型的参数来决定函数的执行逻辑。即使是同一个高阶函数，只要传入不同的函数类型参数，那么它的执行逻辑和最终的返回结果就可能是完全不同的。

在num1AndNum2()函数中，我们没有进行任何具体的运算操作，而是将num1和num2参数传给了第三个函数类型参数，并获取它的返回值，最终将得到的返回值返回。

```kotlin
fun num1AndNum2(num1: Int, num2: Int, operation: (Int, Int) -> Int): Int {
 val result = operation(num1, num2)
 return result
}

fun plus(num1: Int, num2: Int): Int {
 return num1 + num2
}
fun minus(num1: Int, num2: Int): Int {
 return num1 - num}


fun main() {
 val num1 = 100
 val num2 = 80
 val result1 = num1AndNum2(num1, num2, ::plus)

 val result2 = num1AndNum2(num1, num2, ::minus)
 println("result1 is $result1")
 println("result2 is $result2")
}




```

第三个参数使用了::plus和::minus这种写法。这是一种函数引用方式的写法，表示将plus()和minus()函数作为参数传递给num1AndNum2()函数。



## 使用Lambda表达式写法

```kotlin
fun main() {
 val num1 = 100
 val num2 = 80
 val result1 = num1AndNum2(num1, num2) { n1, n2 ->
 n1 + n2
 }
 val result2 = num1AndNum2(num1, num2) { n1, n2 ->
 n1 - n2
 }
 println("result1 is $result1")
 println("result2 is $result2")
}
```



[TOC]

# Android多线程编程

## Android异步消息处理机制

Android中的异步消息处理主要由4个部分组成：Message、Handler、MessageQueue和
Looper。

01. Message
Message是在线程之间传递的消息，它可以在内部携带少量的信息，用于在不同线程之间
传递数据。
02. Handler
Handler主要用于发送和处理消息。发送消息一般
是使用Handler的sendMessage()方法、post()方法等，而发出的消息经过一系列地辗
转处理后，最终会传递到Handler的handleMessage()方法中。
03. MessageQueue
MessageQueue是消息队列的意思，它主要用于存放所有通过Handler发送的消息。这部
分消息会一直存在于消息队列中，等待被处理。每个线程中只会有一个MessageQueue对
象。
04. Looper
Looper是每个线程中的MessageQueue的管家，调用Looper的loop()方法后，就会进入
一个无限循环当中，然后每当发现MessageQueue中存在一条消息时，就会将它取出，并
传递到Handler的handleMessage()方法中。每个线程中只会有一个Looper对象。

### 异步消息处理流程

首先在主线程当中创建一个Handler对象，并重写
handleMessage()方法。当子线程中需要进行UI操作时，就创建一个Message对象，并
通过Handler将这条消息发送出去。之后这条消息会被添加到MessageQueue的队列中等待被
处理，而Looper则会一直尝试从MessageQueue中取出待处理消息，最后分发回Handler的
handleMessage()方法中。由于Handler的构造函数中传入了
Looper.getMainLooper()，所以此时handleMessage()方法中的代码也会在主线程中运
行。

![image-20220714112726306](C:\Users\Clariceliu\AppData\Roaming\Typora\typora-user-images\image-20220714112726306.png)



```kotlin
class MainActivity : AppCompatActivity() {
 val updateText = 1
 val handler = object : Handler(Looper.getMaininLooper()) {
 override fun handleMessage(msg: Message) {
 // 在这里可以进行UI操作，如果发现Message的what字段的值等于updateText，就将TextView显示的内容改成“Nice to meet you”。
 when (msg.what) {
 updateText -> textView.text = "Nice to meet you"
 }
 }
 }
    
 override fun onCreate(savedInstanceState: Bundle?) {
 super.onCreate(savedInstanceState)
 setContentView(R.layout.activity_main)
 changeTextBtn.setOnClickListener {
 thread {
//创建了一个Message（android.os.Message）对象，并将它的what字段的值指定为updateText
 val msg = Message()
 msg.what = updateText
 handler.sendMessage(msg) // 将Message对象发送出去
 }
 }
 }
}

```



### AsyncTask

AsyncTask是一个**抽象类**，所以如果我们想使用
它，就必须创建一个子类去继承它。在继承时我们可以为AsyncTask类指定3个泛型参数，这3
个参数的用途如下。
**Params**。在执行AsyncTask时需要传入的参数，可用于在后台任务中使用。
**Progress**。在后台任务执行时，如果需要在界面上显示当前的进度，则使用这里指定的泛
型作为进度单位。
**Result**。当任务执行完毕后，如果需要对结果进行返回，则使用这里指定的泛型作为返回
值类型。

```kotlin
class DownloadTask : AsyncTask<Unit, Int, Boolean>() {
 ...
}
```

这里我们把AsyncTask的第一个泛型参数指定为Unit，表示在执行AsyncTask的时候不需要传
入参数给后台任务。第二个泛型参数指定为Int，表示使用整型数据来作为进度显示单位。第三
个泛型参数指定为Boolean，则表示使用布尔型数据来反馈执行结果。



当然，目前我们自定义的DownloadTask还是一个空任务，并不能进行任何实际的操作，我们
还需要重写AsyncTask中的几个方法才能完成对任务的定制。经常需要重写的方法有以下4个。
01. **onPreExecute()**
这个方法会在后台任务开始执行之前调用，用于进行一些界面上的初始化操作，比如显示
一个进度条对话框等。
02. **doInBackground(Params...)**
这个方法中的所有代码都会在子线程中运行，我们应该在这里去处理所有的耗时任务。任
务一旦完成，就可以通过return语句将任务的执行结果返回，如果AsyncTask的第三个泛
型参数指定的是Unit，就可以不返回任务执行结果。注意，在这个方法中是不可以进行UI
操作的，如果需要更新UI元素，比如说反馈当前任务的执行进度，可以调用
publishProgress (Progress...)方法来完成。
03. **onProgressUpdate(Progress...)**
当在后台任务中调用了publishProgress(Progress...)方法后，
onProgressUpdate (Progress...)方法就会很快被调用，该方法中携带的参数就是
在后台任务中传递过来的。在这个方法中可以对UI进行操作，利用参数中的数值就可以对
界面元素进行相应的更新。
04. **onPostExecute(Result)**
当后台任务执行完毕并通过return语句进行返回时，这个方法就很快会被调用。返回的数
据会作为参数传递到此方法中，可以利用返回的数据进行一些UI操作，比如说提醒任务执
行的结果，以及关闭进度条对话框等。

```kotlin
class DownloadTask : AsyncTask<Unit, Int, Boolean>() {
 override fun onPreExecute() {
 progressDialog.show() // 显示进度对话框
 }
    
    //这个方法里的代码都是在子线程中运行的，因而不会影响主线程的运行。
 override fun doInBackground(vararg params: Unit?) = try {
 while (true) {
 val downloadPercent = doDownload() // 这是一个虚构的方法,计算当前的下载进度并返回
 publishProgress(downloadPercent)
 if (downloadPercent >= 100) {
 break
 }
 }
 true
 } catch (e: Exception) {
 false
      }
 override fun onProgressUpdate(vararg values: Int?) {
 // 在这里更新下载进度
 progressDialog.setMessage("Downloaded ${values[0]}%")
 }
 override fun onPostExecute(result: Boolean) {
 progressDialog.dismiss()// 关闭进度对话框
 // 在这里提示下载结果
 if (result) {
 Toast.makeText(context, "Download succeeded", Toast.LENGTH_SHORT).show()
 } else {
 Toast.makeText(context, " Download failed", Toast.LENGTH_SHORT).show()
 }
 }
}
```

简单来说，使用AsyncTask的诀窍就是，在doInBackground()方法中执行具体的耗时任务，
在onProgressUpdate()方法中进行UI操作，在onPostExecute()方法中执行一些任务的收
尾工作。

如果想要启动这个任务，只需DownloadTask().execute()，也可以给execute()方法传入任意数量的参数，这些参数将会传递到DownloadTask的doInBackground()方法当中。



## 协程

可以简单地理解成一种轻量级的线程。线程是非常重量级的，它需要依靠操作系统的调度才能实现不同线程之间的切换。而使用协程却可以仅在编程语言的层面就能实现不同协程之间的切换，从而大
大提升了并发编程的运行效率。

协程允许我们在单线程模式下模拟多线程编程的效果，代码执行时的挂起与恢复完
全是由编程语言来控制的，和操作系统无关。

### 协程的基本用法

开启一个协程最简单的方式就是使用Global.launch函数。GlobalScope.launch函数可以创建一个协程的作用域，这样传递给launch函数的代码块（Lambda表达式）就是在协程中运行的了

```kotlin
fun main() {
 GlobalScope.launch {
 println("codes run in coroutine scope")
 }
}

```

Global.launch函数每次创建的都是一个顶层协程，这种协程当应用程序运行结束
时也会跟着一起结束。



让应用程序在协程中所有代码都运行完了之后再结束：**runBlocking函数**

```kotlin
fun main() {
 runBlocking {
 println("codes run in coroutine scope")
 //delay()函数是一个非阻塞式的挂起函数，它只会挂起当前协程，并不会影响其他协程的运行。而Thread.sleep()方法会阻塞当前的线程，这样运行在该线程下的所有协程都会被阻塞。注意，delay()函数只能在协程的作用域或其他挂起函数中调用。
 delay(1500)
 println("codes run in coroutine scope finished")
 }
}

```

runBlocking函数同样会创建一个协程的作用域，但是它可以保证在协程作用域内的所有代码
和子协程没有全部执行完之前一直阻塞当前线程。需要注意的是，runBlocking函数通常只应
该在测试环境下使用，在正式环境中使用容易产生一些性能上的问题。



创建多个协程:**launch**

### suspend

使用suspend关键字，可以将任意函数声明成挂起函数，而挂起函数之间都是可以互相调用的
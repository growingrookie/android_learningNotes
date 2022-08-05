# Intent

Intent是Android程序中各组件之间进行交互的一种重要方式，它不仅可以指明当前组件想要执
行的动作，还可以在不同组件之间传递数据。Intent一般可用于启动Activity、启动Service以
及发送广播等场景

## 显式Intent

Intent有多个构造函数的重载，其中一个是Intent(Context packageContext, Class<? cls)。这个构造函数接收两个参数：第一个参数Context要求提供一个启动Activity的上下 文；第二个参数Class用于指定想要启动的目标Activity

```kotlin
button1.setOnClickListener {
 val intent = Intent(this, SecondActivity::class.java)
 startActivity(intent)
}
```

## 隐式Intent

隐式Intent并不明确指出想要启动哪一个Activity，而是指定了一系列更为抽象的action和category等信息，然后交由系统去分析这个Intent，并找出合适的Activity去启动

通过在AndroidManifest.xml的<activity>标签下配置<intent-filter>的内容，可以指定当前Activity能够响应的action和category

```kotlin
<activity android:name=".SecondActivity" >
 <intent-filter>
 <action android:name="com.example.activitytest.ACTION_START" />
 <category android:name="android.intent.category.DEFAULT" />
 </intent-filter>
</activity>
```

在<action>标签中指明了当前Activity可以响应com.example.activitytest.ACTION_START这个action，而<category>标签则包含了一些附加信息，更精确地指明了当前Activity能够响应的Intent中还可能带有的category。只有<action>和<category>中的内容同时匹配Intent中指定的action和category时，这个
Activity才能响应该Intent

## 向下一个Activity传递数据

### putExtra

Intent中提供了一系列putExtra()方法的重载，可以把要传递的数据暂存在Intent中，在启动另一个Activity后把这些数据从Intent中取出。

putExtra()方法接收两个参数，第一个参数是键，用于之后从Intent中取值，第二个参数才是真正要传递的数据。

```kotlin
button1.setOnClickListener {
 val data = "Hello SecondActivity"
 val intent = Intent(this, SecondActivity::class.java)
 intent.putExtra("extra_data", data)
 startActivity(intent)
}
```

在SecondActivity中将传递的数据取出。调用的是父类的getIntent()方法，该方法会获取用于启动SecondActivity的Intent，然后调用getStringExtra()方法并传入相应的键值，就可以得到传递的数据了。

```kotlin
class SecondActivity : AppCompatActivity() {
 override fun onCreate(savedInstanceState: Bundle?) {
 super.onCreate(savedInstanceState)
 setContentView(R.layout.second_layout)
 val extraData = intent.getStringExtra("extra_data")
 Log.d("SecondActivity", "extra data is $extraData")
 }
}
```



## 返回数据给上一个Activity

### startActivityForResult

Activity类中还有一个用于启动Activity的startActivityForResult()方法，它期望在Activity销毁的时候能够返回一个结果给上一个Activity。

startActivityForResult()方法接收两个参数：第一个参数还是Intent；第二个参数是请
求码，用于在之后的回调中判断数据的来源。

```kotlin
button1.setOnClickListener {
 val intent = Intent(this, SecondActivity::class.java)
 startActivityForResult(intent, 1)
}
```

在SecondActivity中给按钮注册点击事件，并在点击事件中添加返回数据的逻辑

```kotlin
class SecondActivity : AppCompatActivity() {
 override fun onCreate(savedInstanceState: Bundle?) {
 super.onCreate(savedInstanceState)
 setContentView(R.layout.second_layout)
 button2.setOnClickListener {
//这个Intent仅仅用于传递数据
 val intent = Intent()
//把要传递的数据存放在Intent中
 intent.putExtra("data_return", "Hello FirstActivity")
 setResult(RESULT_OK, intent)
 finish()
 }
 }
}
```

setResult()方法专门用于向上一个Activity返回数据。setResult()方法接收两个参数：第一
个参数用于向上一个Activity返回处理结果，一般只使用RESULT_OK或RESULT_CANCELED这
两个值；第二个参数则把带有数据的Intent传递回去。最后调用了finish()方法来销毁当前
Activity。



由于我们是使用startActivityForResult()方法来启动SecondActivity的，在SecondActivity被销毁之后会回调上一个Activity的onActivityResult()方法，因此需要在FirstActivity中重写这个方法来得到返回的数据

```kotlin
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
 super.onActivityResult(requestCode, resultCode, data)
 when (requestCode) {
 1 -> if (resultCode == RESULT_OK) {
 val returnedData = data?.getStringExtra("data_return")
 Log.d("FirstActivity", "returned data is $returnedData")
 }
 }
}
```

onActivityResult()方法带有3个参数：第一个参数requestCode，即我们在启动Activity时传入的请求码；第二个参数resultCode，即我们在返回数据时传入的处理结果；第三个参数data，即携带着返回数据的Intent。由于在一个Activity中有可能调用startActivityForResult()方法去启动很多不同的Activity，每一个Activity返回的数据都
会回调到onActivityResult()这个方法中，因此我们首先要做的就是通过检查requestCode的值来判断数据来源。确定数据是从SecondActivity返回的之后，我们再通过resultCode的值来判断处理结果是否成功。最后从data中取值并打印出来，这样就完成了向上一个Activity返回数据的工作。

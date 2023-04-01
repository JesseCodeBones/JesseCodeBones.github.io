## Menu
首先在 res 目录下新建一个 menu 文件夹，右击 res 目录→New→Directory，输入文件夹名
menu，点击 OK。接着在这个文件夹下再新建一个名叫 main 的菜单文件，右击 menu 文件夹→
New→Menu resource file
文件名输入 main，点击 OK 完成创建。然后在 main.xml 中添加如下代码：

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"> 
 <item 
 android:id="@+id/add_item" 
 android:title="Add"/> 
 <item 
 android:id="@+id/remove_item" android:title="Remove"/> 
</menu>
```
接着重新回到 FirstActivity 中来重写 onCreateOptionsMenu()方法，重写方法可以使用 Ctrl+O 快捷键（Mac 系统是 control + O）然后在 onCreateOptionsMenu()方法中编写如下代码：
```java
public boolean onCreateOptionsMenu(Menu menu) { 
 getMenuInflater().inflate(R.menu.main, menu); 
 return true; 
}  
``` 
通过 getMenuInflater()方法能够得到 MenuInflater 对象，再调用它的 inflate()方法
就可以给当前活动创建菜单了。inflate()方法接收两个参数，第一个参数用于指定我们通过哪
一个资源文件来创建菜单，这里当然传入 R.menu.main。第二个参数用于指定我们的菜单项将
添加到哪一个 Menu 对象当中，这里直接使用 onCreateOptionsMenu()方法中传入的 menu 参数。
然后给这个方法返回 true，表示允许创建的菜单显示出来，如果返回了 false，创建的菜单将
无法显示。  
在 FirstActivity 中重写 onOptionsItemSelected()方法
```Kotlin
    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        if (item.itemId == R.id.menu_add){
            Toast.makeText(this, "you click the add button", Toast.LENGTH_SHORT).show()
        }
        if (item.itemId == R.id.menu_remove){
            Toast.makeText(this, "you click the remove button", Toast.LENGTH_SHORT).show()
        }
        return true
    }
```

## 销毁activity
Activity 类提供了一个 finish()
方法，我们在活动中调用一下这个方法就可以销毁当前活动了

## activity pass parameter

1. putExtra
2. startActivityForResult
3. bundle


## activity resume
应用中有一个Activity A，用户在Activity A的基础上启动了Activity B，Activity A就进入了
停止状态，这个时候由于系统内存不足，将Activity A回收掉了，然后用户按下Back键返回
Activity A，
Activity中还提供了一个onSaveInstanceState()回调方法，这个方法可以保证在
Activity被回收之前一定会被调用，因此我们可以通过这个方法来解决问题
onSaveInstanceState()方法会携带一个Bundle类型的参数，Bundle提供了一系列的方法
用于保存数据，比如可以使用putString()方法保存字符串，使用putInt()方法保存整型数
据，以此类推。每个保存方法需要传入两个参数，第一个参数是键，用于后面从Bundle中取
值，第二个参数是真正要保存的内容。

这里提醒一点，Intent还可以结合Bundle一起用于传递
数据。首先我们可以把需要传递的数据都保存在Bundle对象中，然后再将Bundle对象存放在
Intent里。到了目标Activity之后，先从Intent中取出Bundle，再从Bundle中一一取出数据。

## activity start mode

分别是standard、singleTop、
singleTask和singleInstance



对于single instance模式：
我们的程序中有一个Activity是允许其他程序调用的，如果想实现其他程序和我们的程序可以共享这个Activity的实
例，应该如何实现呢？使用前面3种启动模式肯定是做不到的，因为每个应用程序都会有自己的
返回栈，同一个Activity在不同的返回栈中入栈时必然创建了新的实例。而使用
singleInstance模式就可以解决这个问题，在这种模式下，会有一个单独的返回栈来管理这个
Activity，不管是哪个应用程序来访问这个Activity，都共用同一个返回栈，也就解决了共享
Activity实例的问题

## companion object
```kotlin
class SecondActivity : BaseActivity() {
 ...
 companion object {
 fun actionStart(context: Context, data1: String, data2: String) {
 val intent = Intent(context, SecondActivity::class.java)
 intent.putExtra("param1", data1)
 intent.putExtra("param2", data2)
 context.startActivity(intent)
 }
 }
}```

start:  
```kotlin
button1.setOnClickListener {
 SecondActivity.actionStart(this, "data1", "data2")
}```

如果想在kotlin中真正的使用静态方法，需要在companion object方法中加入@JvmStatic注解  

## kotlin顶层方法

```kotlin
fun doSomething() {
 println("do something")
}
```
顶层方法不属于类里面，可以直接用名称调用。
如果想在java中调用顶层方法，kotlin会创建一个和文件名一样的类，将顶层方法当作静态方法放在其中。  

## 布局
线性布局：  
android:gravity
用于指定文字在控件中的对齐方式，而android:layout_gravity用于指定控件在布局中的
对齐方式。

习LinearLayout中的另一个重要属性——android:layout_weight。这个属
性允许我们使用比例的方式来指定控件的大小

相对布局：  
android:layout_alignParentLeft、
android:layout_alignParentTop、android:layout_alignParentRight、
android:layout_alignParentBottom、android:layout_centerInParent
android:layout_toLeftOf
android:layout_toRightOf

帧布局：  
所有的控件都会默认摆放在布局的左上角

自定义控件


## LayoutInflater
`LayoutInflater``是根据布局资源文件来生成视图层级（包括子视图）的系统服务

```Kotlin
View view = LayoutInflater.from(MainActivity.this).inflate(R.layout.view_child, null);
frameLayout.addView(view);
```

## 自定义view group
1. 创建一个layout布局
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="JesseTest"
    android:id="@+id/jessetest"
    />
</LinearLayout>
```
2. 创建一个layout类
``` Kotlin
package com.example.myapplication

import android.content.Context
import android.util.AttributeSet
import android.view.LayoutInflater
import android.widget.Button
import android.widget.LinearLayout
import android.widget.Toast

class TitleLayout(context:Context, attrs:AttributeSet):LinearLayout(context, attrs) {

    init {
        val view =  LayoutInflater.from(context).inflate(R.layout.title, this)
        findViewById<Button>(R.id.jessetest).setOnClickListener {
            Toast.makeText(context, "jesse test", Toast.LENGTH_LONG).show()
        }
    }
}
```

3. 在其他的布局中使用自定义的布局
```xml
<com.example.myapplication.TitleLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    tools:ignore="MissingConstraints" />
```
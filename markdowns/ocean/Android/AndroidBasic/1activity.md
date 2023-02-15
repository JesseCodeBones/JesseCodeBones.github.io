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


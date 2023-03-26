# Kotlin language
## control flow
`    fun max(a:Int, b:Int) = if (a>b) a else b;`  
```kotlin
fun chose(a:Int, b:Int, c:Int):Int = when(a){
        0->b;
        1->c;
        else -> -1;
    }
```

```kotlin
for(i in 0..10){
    println(i)
}

for(i in 0 until 10 step 2){
    println(i)
}
for(i in 10 downTo 1){
    println(i)
}
```

## class
```kotlin
open class Person(val name:String){

}
class Student(name:String, age:Int):Person(name){
    constructor(name: String):this(name, 0){}
}

```
修饰符 Java Kotlin 修饰符 Java Kotlin
public 所有类可见 所有类可见（默认）
private 当前类可见 当前类可见
protected 当前类、子类、同一包路径下的类可见 当前类、子类可见
default 同一包路径下的类可见（默认） 无 internal 无 同一模块中的类可见  
当在一个类前
面声明了data关键字时，就表明你希望这个类是一个数据类，Kotlin会根据主构造函数中的参
数帮你将equals()、hashCode()、toString()等固定且无实际逻辑意义的方法自动生成，
从而大大减少了开发的工作量。  
在Kotlin中创建一个单例类的方式极其简单，只需要将class关键字改成object关键字即可。
现在我们尝试创建一个Kotlin版的Singleton单例类，右击com.example.helloworld包
→New→Kotlin File/Class，在弹出的对话框中输入“Singleton”，创建类型选择“Object”，点
击“OK”完成创建

## Lambda
{参数名1: 参数类型, 参数名2: 参数类型 -> 函数体}  
```kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape", "Watermelon")
val lambda = { fruit: String -> fruit.length }
val maxLengthFruit = list.maxBy(lambda)
```

然后Kotlin规定，当Lambda参数是函数的最后一个参数时，可以将Lambda表达式移到函数括
号的外面

val maxLengthFruit = list.maxBy() { fruit: String -> fruit.length }  
简化为:  
val maxLengthFruit = list.maxBy { it.length }

## string template
"hello, ${obj.name}. nice to meet you!"  
"hello, $name. nice to meet you!"

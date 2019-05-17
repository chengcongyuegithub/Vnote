# final和this逃逸
final域的重排序规则
对于final 域，编译器和处理器要遵守两个重排序规则：
在构造函数内对一个 final 域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序
```
对象构造函数内有final域，必须先用构造函数构造对象，再把对象赋给其他引用
```
初次读一个包含 final 域的对象的引用，与随后初次读这个 final 域，这两个操作之间不能重排序。
```
先读引用,在读引用中的final字段
```
```
public class FinalExample {
    int i;                            //普通变量
    final int j;                      //final变量
    static FinalExample obj;

    public void FinalExample () {     //构造函数
        i = 1;                        //写普通域
        j = 2;                        //写final域
    }

    public static void writer () {    //写线程A执行
        obj = new FinalExample ();
    }

    public static void reader () {       //读线程B执行
        FinalExample object = obj;       //读对象引用
        int a = object.i;                //读普通域
        int b = object.j;                //读final域
    }
}  
```
写 final 域的重排序规则可以确保：在引用变量为任意线程可见之前，该引用变量指向的对象的 final 域已经在构造函数中被正确初始化过了
final引用从构造函数中“溢出”
```
public class FinalReferenceEscapeExample {
final int i;
static FinalReferenceEscapeExample obj;

public FinalReferenceEscapeExample () {
    i = 1;                              //1写final域
    obj = this;                          //2 this引用在此“逸出”
}

public static void writer() {
    new FinalReferenceEscapeExample ();
}

public static void reader {
    if (obj != null) {                     //3
        int temp = obj.i;                 //4
    }
}

}
```
假设一个线程 A 执行 writer() 方法，另一个线程 B 执行 reader() 方法。
这里的操作2使得对象还未完成构造前就为线程 B 可见。即使这里的操作 2 是构造函数的最后一步，且即使在程序中操作 2 排在操作 1 后面
```
执行 read() 方法的线程仍然可能无法看到 final 域被初始化后的值
因为这里的操作 1 和操作 2 之间可能被重排序?
```
**在构造函数返回前，被构造对象的引用不能为其他线程可见，因为此时的 final 域可能还没有被初始化。
在构造函数返回后，任意线程都将保证能看到 final 域正确初始化之后的值**

```
如果在一个线程构造了一个不可变对象之后（对象仅包含final字段），就可以保证了这个对象被其他线程正确的查看(X)
没有保证this逃逸
```
形成this逃逸的情况
```
在本类中声明了一个自己的类
public class Test01
{
    static Test01 test01;
    //构造函数
    {
       test01=this;//或者
       //this01=new Test01() 
    }
}
```
这就会引起逃逸
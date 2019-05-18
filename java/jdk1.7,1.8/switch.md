# switch
## 题目
```
java7后关键字 switch 支不支持字符串作为条件：（）
支持(true)
不支持
```
## 解析
在Java7之前，switch只能支持 byte、short、char、int或者其对应的封装类以及Enum类型。在Java7中，呼吁很久的String支持也终于被加上了。

在switch语句中，表达式的值不能是null，否则会在运行时抛出NullPointerException。在case子句中也不能使用null，否则会出现编译错误。

同时，case字句的值是不能重复的。对于字符串类型的也一样，但是字符串中可以包含Unicode转义字符。重复值的检查是在Java编译器对Java源代码进行相关的词法转换之后才进行的。也就是说，有些case字句的值虽然在源代码中看起来是不同的，但是经词法转换之后是一样的，就会在成编译错误。比如：“男”和“\u7537”就是一个意思。

然后看一个源代码及反编译后的代码：

public class StringForSwitch {
    public void test_string_switch() {
        String result="";  
        switch ("doctor") {
        case "doctor":
            result = "doctor";
            break;
        default:
            break;
        }
    }
}
反编译后的，还原成大致的Java的代码如下：

public class StringForSwitch {
    public StringForSwitch() {
    }
    public void test_string_switch() {
        String result = "";
        String var2 = "doctor";
        switch("doctor".hashCode()) {
        case -1326477025:
            if(var2.equals("doctor")) {
                result = "doctor";
            }
        default:
           break;
        }
    }
}
可以看出，**字符串类型在switch语句中利用hashcode的值与字符串内容的比较来实现的；但是在case字句中对应的语句块中仍然需要使用String的equals方法来进一步比较字符串的内容，这是因为哈希函数在映射的时候可能存在冲突**。


```
# jvm参数
```
-Xmx：最大堆大小
-Xms：初始堆大小
-Xmn:年轻代大小
-XXSurvivorRatio：年轻代中Eden区与Survivor区的大小比值

```
```
-Xmx10240m -Xms10240m -Xmn5120m -XXSurvivorRatio=3
年轻代5120m， Eden：Survivor=3，Survivor区大小=1024m（Survivor区有两个，即将年轻代分为5份，每个Survivor区占一份），总大小为2048m。
-Xms初始堆大小即最小内存值为10240m
```

```
 -Xmx10240m：代表最大堆
 -Xms10240m：代表最小堆
 -Xmn5120m：代表新生代
 -XXSurvivorRatio=3：代表Eden:Survivor = 3    根据Generation-Collection算法(目前大部分JVM采用的算法)，一般根据对象的生存周期将堆内存分为若干不同的区域，一般情况将新生代分为Eden ，两块Survivor；    计算Survivor大小， Eden:Survivor = 3，总大小为5120,3x+x+x=5120  x=1024
新生代大部分要回收，采用Copying算法，快！
老年代 大部分不需要回收，采用Mark-Compact算法
```
```
关于OutOfMemoryError，下面说法正确的是（）？
正确答案: A B C   你的答案: A B (错误)
java.lang.OutOfMemoryError: PermGen space 增加-XX:MaxPermSize这个参数的值的话，这个问题通常会得到解决。
java.lang.OutOfMemoryError: Requested array size exceeds VM limit当你正准备创建一个超过虚拟机允许的大小的数组时，这条错误将会出现
java.lang.OutOfMemoryError: Java heap space 一般情况下解决这个问题最快的方法就是通过-Xmx参数来增加堆的大小
java.lang.OutOfMemoryError: nativeGetNewTLA这个异常只有在jRockit虚拟机时才会碰到
```
```
java.lang.OutOfMemoryError: PermGen space
查了一下为"永久代"内存大小不足，“永久代”的解释应该为JVM中的方法区，主要用于存储类信息，常量，静态变量，即时编译器编译后代码等。本错误仅限于Hotspot虚拟机，本区进行垃圾回收很少，不够直接加大简单粗暴。

java.lang.OutOfMemoryError: Requested array size exceeds VM limit
直接翻译报错信息：数组过长导致堆内存溢出，加大堆内存或者减少数组长度。

java.lang.OutOfMemoryError: Java heap space
堆内存不足，直接增大堆内存。
```
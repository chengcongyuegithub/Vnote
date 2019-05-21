# check exception
```
checked exception：指的是编译时异常，该类异常需要本函数必须处理的，用try和catch处理，或者用throws抛出异常，然后交给调用者去处理异常。
runtime exception：指的是运行时异常，该类异常不必须本函数必须处理，当然也可以处理。
Thread.sleep()抛出的InterruptException属于checked exception；IllegalArgumentException属于Runtime exception;
```

```
Java的异常分为两种，一种是运行时异常（RuntimeException），一种是非运行异常也叫检查式异常（CheckedException）。
1、运行时异常不需要程序员去处理，当异常出现时，JVM会帮助处理。常见的运行时异常有：
ClassCastException(类转换异常)
ClassNotFoundException
IndexOutOfBoundsException(数组越界异常)
NullPointerException(空指针异常)
ArrayStoreException(数组存储异常，即数组存储类型不一致)
还有IO操作的BufferOverflowException异常
2、非运行异常需要程序员手动去捕获或者抛出异常进行显示的处理，因为Java认为Checked异常都是可以被修复的异常。常见的异常有：
IOException
SqlException
```
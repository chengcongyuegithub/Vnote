# Threadlocal
ThreadLocal的类声明：
```
public class ThreadLocal<T>
```
可以看出ThreadLocal并没有继承自Thread，也没有实现Runnable接口。
2、ThreadLocal类为每一个线程都维护了自己独有的变量拷贝。每个线程都拥有了自己独立的一个变量。
所以ThreadLocal重要作用并不在于**多线程间的数据共享，而是数据的独立**
由于每个线程在访问该变量时，读取和修改的，都是自己独有的那一份变量拷贝，不会被其他线程访问，
变量被彻底封闭在每个访问的线程中
```
ThreadLocal保证各个线程间数据安全，每个线程的数据不会被另外线程访问和破坏
```
ThreadLocal中定义了一个哈希表用于为每个线程都提供一个变量的副本：
```
 static class ThreadLocalMap {

        static class Entry extends WeakReference<ThreadLocal> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal k, Object v) {
                super(k);
                value = v;
            }
        }

        /**
         * The table, resized as necessary.
         * table.length MUST always be a power of two.
         */
        private Entry[] table;
}
```
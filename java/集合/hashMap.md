# hashMap
## hashMap的属性
```
HashMap不能保证元素的顺序,
HashMap能够将键设为null，也可以将值设为null，
与之对应的是Hashtable,(注意大小写：不是HashTable)，
Hashtable不能将键和值设为null，否则运行时会报空指针异常错误；
HashMap线程不安全，Hashtable线程安全
```
## 题1
```
java.util.HashMap map=new java.util.HashMap(); 
map.put("name",null);      
map.put("name","Jack");
System.out.println(map.size()); //1
```
解析:
```
HashMap可以插入null的key或value，插入的时候，检查是否已经存在相同的key，如果不存在，则直接插入，如果存在，则用新的value替换旧的value，在本题中，第一条put语句，会将key/value对插入HashMap，而第二条put，因为已经存在一个key为name的项，所以会用新的value替换旧的vaue，因此，两条put之后，HashMap中只有一个key/value键值对。那就是（name，jack）。所以，size为1.
```
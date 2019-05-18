# Iterator
## 题1
```
list是一个ArrayList的对象，哪个选项的代码填到//todo delete处，可以在Iterator遍历的过程中正确并安全的删除一个list中保存的对象？（）
Iterator it = list.iterator();
int index = 0;
while (it.hasNext())
{
    Object obj = it.next();
    if (needDelete(obj))  //needDelete返回boolean，决定是否要删除
    {
        //todo delete
    }
    index ++;
}
```
正确结果:
it.remove();
Iterator  支持从源集合中安全地删除对象，只需在 Iterator 上调用 remove() 即可。这样做的好处是可以避免 ConcurrentModifiedException ，当打开 Iterator 迭代集合时，同时又在对集合进行修改。有些集合不允许在迭代时删除或添加元素，但是调用 Iterator 的remove() 方法是个安全的做法

```
ArrayList 继承了 AbstractList， 其中AbstractList 中有个modCount 代表了集合修改的次数。在ArrayList的iterator方法中会判断 expectedModCount与 modCount是否相等，如果相等继续执行，不相等报错，只有iterator的remove方法会在调用自身的remove之后让 expectedModCount与modCount再相等，所以是安全的。
```
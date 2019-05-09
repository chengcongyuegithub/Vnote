# 生成maxTree(二)
通过(一),我们了解了单调栈,并且实现了功能,我们现在有了两个HashMap,现在就要生成maxTree.
## 分析,排除不可能的事件
### 多个树(森林的可能)
我们生成的maxTree可不可能不止一个根,可不可能有多个根.
不可能,因为只有左边为null,右边为null的才有可能是根节点.而这中可能的节点,也只可能有一个
### 多叉树的可能
也不可能,我们假设一下
```
  a---t1---t2
```
多叉树也就是不止有两个孩子,可能有多个孩子
如果可能的话,t1和t2都是a的孩子,那么a>t1,a>t2其中t1和t2肯定也是有大小关系的
如果t1>t2,这样的话a>t1>t2,t2会以t1为父亲,这种可能pass
然后t1<t2,这样的话a和t2都是大于t1的,这个时候t1会以更近的一个为父亲,,,,pass
## 代码实现
通过上面的分析,我们可以知道只有可能是二叉树.不存在其他的可能,那么接下来就是形成二叉树了.
我们先写一个大体的框架
```
for(//遍历数组)
{
    得到当前的节点
    通过两个map获得它左边比较大的和右边比较大的.
    left
    right
    然后左边和右边都有可能称为它的父亲然后就是if else判断
    ...
}
```
...就是我们要完善的地方,我们先提出所有的可能性
如果left为空
如果right为空
如果都为空
如果都不为空
```
if(left==null&&right==null)
{
    head=curNode
}else if(left==null)//也就是一边为空的情况
{
     if(left.right!=null)
     {
         left.left=curNode; 
     }else
     {
         left.right=curNode;
     }
}else if(right==null)
{
     if(right.right!=null)
     {
         right.left=curNode; 
     }else
     {
         right.right=curNode;
     }
}else
{
     比较一下大小,选择较小的那个
     parent=left.value<right.value?left:right;
     if(parent.left!=null)
     {
         parent.left=curNode;
     }else
     {
         parent.right=curNode;
     }
}
```
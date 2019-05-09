# 最大值减去最小值小于等于num的子数组的数量
有一个整形数组arr[],我们要求arr[i.....j],也就是arr[]中的子数组满足max(arr[i-----j])-min(arr[i-------j])<=num的子数组的数量
## 分析
子数组中的最大值和最小值相减.
子数组中的最大值和最小值的维护是可以通过窗口来进行维护的,所以我们要采用单调栈,但是我们arr[i-----j]中左边肯定会一直发生变化的.所以这里要采用在窗口中使用的双端队列.
然后我们回忆一下,窗口中双端队列,随着窗口移动,一直都维持着最大值在栈底的情况,然后随着窗口移动,一直都会有元素从队列的头出去.
这道题我们也使用双端队列,通过队尾来维护子数组的最大值和最小值,然后通过队头来维护,子数组i------j.

然后我们知道
```
arr[i---j]如果满足条件,那么其中的所有子数组都满足这个条件
arr[i---j]如果不满足条件,那就没必要在扩大了
```
## 代码实现
首先就是双端队列声明和队列尾部的维护
```
LinkedList<Integer> qmax=new LinkedList<Integer>();
LinkedList<Integer> qmin=new LinkedList<Integer>();

while(!qmax.isEmpty&&arr[qmax.peek()]<=arr[i])
{
     qmax.poll();
}
qmax.push(i);

while(!qmim.isEmpty&&arr[qmin.peek()]>=arr[i])
{
     qmin.poll();
}
qmin.push(i);
```
然后就是栈顶的维护
```
if(qmax.peekFirst()==i)
{
   qmax.pollFirst();
}
if(qmin.peekFirst()==i)
{
   qmin.pollFirst();
}
```
全部代码实现
```
public class Test04 {

    public static int f(int[] arr,int num)
    {
        int res=0;
        LinkedList<Integer> qmax=new LinkedList<>();
        LinkedList<Integer> qmin=new LinkedList<>();
        int j=0;
        for(int i=0;i<arr.length;i++)
        {
            for(;j<arr.length;j++)
            {
                while(!qmax.isEmpty()&&arr[qmax.peekLast()]<=arr[j])
                {
                    qmax.pollLast();
                }
                qmax.addLast(j);
                while(!qmin.isEmpty()&&arr[qmin.peekLast()]>=arr[j])
                {
                    qmin.pollLast();
                }
                qmin.addLast(j);

                if(arr[qmax.peekFirst()]-arr[qmin.peekFirst()]>num)
                {
                    break;
                }
            }

            if(qmax.peekFirst()==i)
            {
                qmax.pollFirst();
            }
            if(qmin.peekFirst()==i)
            {
                qmin.pollFirst();
            }
            res+=j-i;
        }

        return res;
    }

    public static void main(String[] args) {
        int[] arr={3,4,2,1,3,3,4,1};
        System.out.println(f(arr,1));
    }
}

```
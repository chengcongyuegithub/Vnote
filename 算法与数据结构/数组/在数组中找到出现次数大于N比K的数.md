# 在数组中找到出现次数大于N/K的数
## 题目
### 原题目
给定一个整形数组,打印其中出现次数超过一半的数,如果没有这样的数,我们就打印提示信息
### 进阶题目
给定一个整数K,打印出这个数组中出现次数大于N/K的的数字
## 分析
核心思路:
```
每一次的循环都在数组中删除K个不同的数字,当删到不能在删,也就是剩下的数字的种数,不满K个,这些剩下的数字,就是满足条件的
```
我们对照这原问题,来分析,我们每一次都删除两个不同的数字,每次都是两个不同的数字,删到最后只剩下一种,那么剩下的就是超过次数一半的数
对于进阶问题呢
```
4 4 4 4 3 3 3 1 1 1 
```
我们每次都删除3个不同的数字,最后的数字不满3个,那么剩余的就是结果
## 代码实现1
我们首先实现在一个for循环中,删除两个不同种类的树,循环结束之后,得到的就是结果
```
for(int i=0;i<arr.length;i++)
{
    if(times==0) 
    {
        cands=arr[i];
        times++;
    }else if(cands==arr[i])
    {
        times++; 
    }else if(cands!=arr[i])//同时此时的times>0
    {
        times--;
    }
}
```
在一个for循环中,我们就完成了我们的任务
cands表示当前的候选,如果当前的times为0,表示当前没有一个值是候选的,这个时候我们就把当前的arr[i]设置为候选值,当前的times为1,如果我们在有候选值的情况下,我们碰到了一个相同的值,我们就增加一个点数,相当去一行可以抵消不止一个不同的数字,如果我们候选值和当前的值不相等,我们相当于凑齐了两个不相同的值,抵消,此时的times如果为1,表明两个值真的抵消了,这个时候什么都没了,在下次循环的时候,我们要找另一个候选值,如果减完之后times还有值,表明可以继续和后面的值进行抵消
```
需要注意的地方:候选值和times的搭配使用
```
## 代码实现2
```
1 1 1 1 2 2 2 2 3
```
上面这种情况,最后1剩下了,可以1的次数并没有超过1/2,所以我们还要确认一下
```
        times=0;
        for(int i=0;i!=arr.length;i++)
        {
            if(arr[i]==cand) times++;
        }
        if(times>arr.length/2)
        {
            System.out.println(cand);
        }else
        {
            System.out.println("没有这个值!!");
        }
```
## 进阶题目的分析
对于进阶题目,我们已经知道了解题的思路,我们每次都删除K个不同的值,然后最后剩下不满K个值,肯定就是满足情况的,对于每一次删除两个值,我们设置了一个候选键,和一个相对应的times.对于每一次删除K个值,我们就要设置K-1个候选值,以及K-1个times
解题思路:
当我们遍历到arr[i]的时候,如果和某一个候选值相同,我们就在对应的times上面++,如果和其中的所有值都不相等,我们就要分情况
- 如果不满K-1个候选值,我们就添加
- 如果满K-1候选值了,我们就要抵消一次了,对于所有的候选值的times都要减减
## 代码实现3
```
public static void printKMajor(int[] arr,int K)
    {
        HashMap<Integer,Integer> cands=new HashMap<>();
        for(int i=0;i<arr.length;i++)
        {
            if(cands.containsKey(arr[i]))
            {
                cands.put(arr[i],cands.get(arr[i])+1);
            }else
            {
                if(cands.size()==K-1)
                {
                    allCandsMinusOne(cands);
                }
                else
                {
                    cands.put(arr[i], 1);
                }
            }
        }
        HashMap<Integer,Integer> reals=getReals(arr,cands);
        boolean hasPrint=false;
        for(Map.Entry<Integer,Integer> set:cands.entrySet())
        {
            Integer key=set.getKey();
            if(reals.get(key)>arr.length/K)
            {
                hasPrint=true;
                System.out.println(key+" ");
            }
        }
        System.out.println(hasPrint?"":"no such number.");
    }

    public static void allCandsMinusOne(HashMap<Integer,Integer> map)
    {
        List<Integer> removeList=new LinkedList<>();
        for(Map.Entry<Integer,Integer> set:map.entrySet())
        {
            Integer key=set.getKey();
            Integer value=set.getValue();
            if(value==1)
            {
                removeList.add(key);
            }
            map.put(key,value-1);
        }
        for(Integer removeKey:removeList)
        {
            map.remove(removeKey);
        }
    }

    public static HashMap<Integer,Integer> getReals(int[] arr,HashMap<Integer,Integer> cands)
    {
        HashMap<Integer,Integer> reals=new HashMap<>();
        for(int i=0;i<arr.length;i++)
        {
            int curNum=arr[i];
            if(cands.containsKey(curNum))
            {
               if(reals.containsKey(curNum))
               {
                   reals.put(curNum,reals.get(curNum)+1);
               }else
               {
                   reals.put(curNum,1);
               }
            }
        }
        return reals;
    }
```
## 拓展
![](_v_images/20190603195556761_1072820505.png =1060x)
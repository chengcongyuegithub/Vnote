# KMP算法的应用之最大子树
## 题目
判断t1树中是否有t2树的全部拓扑结构的子树,我们通过KMP算法来求解这一道题
## 分析
我们首先将t1和t2进行序列化,如果t2序列化的字符串是t1序列化字符串的子串,就是true
## 代码实现
```
public class Test03 {

    public static class Node
    {
        public int value;
        public Node left;
        public Node right;

        public Node(int data)
        {
            this.value=data;
        }
    }
    public static boolean isSubtree(Node t1, Node t2) {
        String t1Str = serialByPre(t1);
        String t2Str = serialByPre(t2);
        return getIndexOf(t1Str, t2Str) != -1;
    }


    public static String serialByPre(Node head)
    {
        if(head==null)
        {
            return "#!";
        }
        String res=head.value+"!";
        res+=serialByPre(head.left);
        res+=serialByPre(head.right);
        return res;
    }

    public static int getIndexOf(String s, String m) {
        if (s == null || m == null || m.length() < 1 || s.length() < m.length()) {
            return -1;
        }
        char[] ss = s.toCharArray();
        char[] ms = m.toCharArray();
        int[] nextArr = getNextArray(ms);
        int index = 0;
        int mi = 0;
        while (index < ss.length && mi < ms.length) {
            if (ss[index] == ms[mi]) {
                index++;
                mi++;
            } else if (nextArr[mi] == -1) {
                index++;
            } else {
                mi = nextArr[mi];
            }
        }
        return mi == ms.length ? index - mi : -1;
    }

    public static int[] getNextArray(char[] ms)
    {
        if(ms.length==1)
        {
            return new int[]{-1};
        }
        int[] nextArr=new int[ms.length];
        nextArr[0]=-1;
        nextArr[1]=0;
        int pos=2;
        int cn=0;
        while(pos<nextArr.length)
        {
            if(ms[pos-1]==ms[cn])
            {
                nextArr[pos++]=++cn;
            }else if(cn>0)
            {
                cn=nextArr[cn];
            }else
            {
                nextArr[pos++]=0;
            }
        }
        return nextArr;
    }
}
```
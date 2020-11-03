### 【题目】和为S的连续正数序列

小明很喜欢数学,有一天他在做数学作业时,要求计算出9~16的和,他马上就写出了正确答案是100。但是他并不满足于此,他在想究竟有多少种连续的正数序列的和为100(至少包括两个数)。没多久,他就得到另一组连续正数和为100的序列:18,19,20,21,22。现在把问题交给你,你能不能也很快的找出所有和为S的连续正数序列?

输出所有和为S的连续正数序列。序列内按照从小至大的顺序，序列间按照开始数字从小到大的顺序

### 【解题思路1】滑动窗口

```java
public class Solution {
    public ArrayList<ArrayList<Integer> > FindContinuousSequence(int sum) {
        ArrayList<ArrayList<Integer>> ans = new ArrayList<>();
        if(sum < 2) return ans;
        int i = 1, j = 1;
        int s = 0;
        while(i <= sum / 2) {
            if(s == sum) {
                ArrayList<Integer> temp = new ArrayList<>();
                for(int k = i; k <j; k++) {
                    temp.add(k);
                }
                ans.add(temp);
                s -= i;
                i++;
            } else if(s > sum) {
                s -= i;
                i++;
            } else {
                s += j;
                j++;
            }
        }
        return ans;
    }
}
```

```java
public class Solution {
    public ArrayList<ArrayList<Integer> > FindContinuousSequence(int sum) {
        ArrayList<ArrayList<Integer>> ans = new ArrayList<>();
        if(sum < 2) return ans;
        int i = 1, j = 2;
        while(i <= sum / 2) {
            if(i < j) {
                int s = (i + j) * (j - i + 1) / 2;
                if(s == sum) {
                    ArrayList<Integer> temp = new ArrayList<>();
                    for(int k = i; k <= j; k++) {
                        temp.add(k);
                    }
                    ans.add(temp);
                    i++;
                } else if(s < sum) {
                    j++;
                } else {
                    i++;
                }
            } else {
                j++;
            }
        }
        return ans;
    }
}
```


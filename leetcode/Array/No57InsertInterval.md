给出一个*无重叠的 ，*按照区间起始端点排序的区间列表。

在列表中插入一个新的区间，你需要确保列表中的区间仍然有序且不重叠（如果有必要的话，可以合并区间）。

示例 1：

```xml
输入：intervals = [[1,3],[6,9]], newInterval = [2,5]
输出：[[1,5],[6,9]]
示例 2：
```

示例 2：

```xml
输入：intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]], newInterval = [4,8]
输出：[[1,2],[3,10],[12,16]]
解释：这是因为新的区间 [4,8] 与 [3,5],[6,7],[8,10] 重叠。
```



```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        int left = newInterval[0];
        int right = newInterval[1];
      	//是否存放了数组
        boolean placed = false;
        List<int[]> list = new ArrayList<>();
        for(int[] interval : intervals){
            if(interval[0] > right){
                //在插入区间的右侧且无交集
                //[[1,2][4,6]] <--- [3,5]
                if(!placed){
                    list.add(new int[]{left,right});
                    placed = true;
                }
                list.add(interval);
            }else if(interval[1] < left){
                //在插入区间的左侧没有交集
                list.add(interval);
            }else{
                //与插入区间有交集，计算他们的并集
                left = Math.min(left,interval[0]);
                right = Math.max(right,interval[1]);
            }
        }
        if(!placed){
            list.add(new int[]{left,right});
        }
        int[][] ans = new int[list.size()][2];
        for(int i=0;i<list.size();++i){
            ans[i] = list.get(i);
        }
        return ans;
    }
}
```






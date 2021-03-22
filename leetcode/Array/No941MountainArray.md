给定一个整数数组 `A`，如果它是有效的山脉数组就返回 `true`，否则返回 `false`。

让我们回顾一下，如果 A 满足下述条件，那么它是一个山脉数组：

```xaml
A.length >= 3
在 0 < i < A.length - 1 条件下，存在 i 使得：
A[0] < A[1] < ... A[i-1] < A[i]
A[i] > A[i+1] > ... > A[A.length - 1]
```

```java
/**
双指针
*/
class Solution {
    public boolean validMountainArray(int[] A) {
        if(A.length < 3){
            return false;
        }
        int left = 0;
        int right = A.length - 1;
       // 从左往右便利，找最高点
        while(left + 1 < A.length && A[left] < A[left+1]){
            left++;
        }
      //从右往左遍历找，最高点
        while(right - 1 > 0 && A[right] < A[right-1]){
          right--;
        }
      //left不能是最右边，right也不能倒最左边，高点应该一致
        return left == right && left != A.length-1 &&
          right != 0; 
    }
}
```


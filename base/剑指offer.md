# 剑指offer
## 7. 斐波那契数列
### 7.1 算法描述
大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项。
n<=39

[newCode](https://www.nowcoder.com/practice/c6c7742f5ba7442aada113136ddea0c3?tpId=13&tqId=11160&tPage=1&rp=1&ru=%2Fta%2Fcoding-interviews&qru=%2Fta%2Fcoding-interviews%2Fquestion-ranking)

### 算法思路
用两个指针，一个记录倒数第一个，一个记录倒数第二个。
用这两个指针计算新的值，并重新更新这两个指针。复杂度 O(N)

### 算法实现
```
public class Solution {
    public int Fibonacci(int n) {
        if (n <= 0) {
            return 0;
        }
        if(n == 1) {
            return 1;
        }
        int preOne = 1, preTwo = 1;
        for(int i = 3; i <= n; i++){
            int temp = preOne;
            preOne = preOne + preTwo;
            preTwo = temp;
        }
        return preOne;
    }
}
```

## 8. 跳台阶
### 8.1 算法描述
一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

[newCode](https://www.nowcoder.com/practice/8c82a5b80378478f9484d87d1c5f12a4?tpId=13&tqId=11161&tPage=1&rp=1&ru=%2Fta%2Fcoding-interviews&qru=%2Fta%2Fcoding-interviews%2Fquestion-ranking)

### 8.2 算法思路
类似斐波那契数列。

### 8.3 算法实现

```
public class Solution {
    public int JumpFloor(int target) {
       if(target <=0) return 0 ;
       if(target == 1) return 1;
        
        int preOne = 2, preTwo = 1;
        for(int i = 3; i <= target; i++) {
            int temp = preOne;
            preOne = preOne + preTwo;
            preTwo = temp;
        }   
        return preOne;
    }
}
```

## 9. 变态跳台阶
### 9.1 算法描述
一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

### 9.2 算法思路
是2的n次方

### 9.3 算法实现

```
public class Solution {
    public int JumpFloorII(int target) {
        if (target <= 0) return 0;
        int result = 1;
        target --;
        while (target > 0) {
            result = result << 1;
            target--;
        }
         
        return result;
    }
}
```

## 10. 矩形覆盖
### 10.1 算法描述
我们可以用2*1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2*1的小矩形无重叠地覆盖一个2*n的大矩形，总共有多少种方法？

[newCode](https://www.nowcoder.com/practice/72a5a919508a4251859fb2cfb987a0e6?tpId=13&tqId=11163&tPage=1&rp=1&ru=%2Fta%2Fcoding-interviews&qru=%2Fta%2Fcoding-interviews%2Fquestion-ranking)

### 10.2 算法思路
和斐波那契数列一样

### 10.3 算法实现
```
public class Solution {
     public int RectCover(int target) {
        if (target <= 0) return 0;
        if (target == 1) return 1;
        if (target == 2) return 2;
        int temp;
        int preOne = 2;
        int preTwo = 1;
        for (int i = 3; i <= target; i++) {
            temp = preOne;
            preOne = preOne + preTwo;
            preTwo = temp;
        }
        return preOne;
    }
}
```

## 11. 



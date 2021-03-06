### 题目描述（容易）

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/Japanese-Language-Learning/1-Two-Sum.png)

输入一个数组和一个 target 值，要求输出数组中的两个和为 target 的数，并且一个元素不能用两次。  

#### 方法一：暴力循环
```  java

class Solution {
    public int[] twoSum(int[] nums, int target) {
        int i = 0,j = 0;
        //int result[2] = 0;
        int[] result = new int[2];
        for(i = 0;i < nums.length-1;i++){
            for(j = i+1;j < nums.length;j++){
                if(nums[i] == target - nums[j]){
                    result[0] = i;
                    result[1] = j;
                    return result;
                    //return new int[]{i,j};
                }
            }
        }
        return result;
        //throw new IllegalArgumentException("No two sum solution");
    }
}  
```
通过两层 for 循环遍历所有两个元素的组合。  
时间复杂度：O(n²)  
空间复杂度：O(1)  

#### 方法二：  
对于第二个 for 循环找剩下一个满足要求的数时，方法一用的是遍历剩下的元素，这会产生 O(n) 的时间复杂度，那么怎么才可以避免呢？   

有没有想过用 Map 来做，是不是想起了什么？Map 不同于数组的地方就在于它寻找一个元素是通过一个函数，而不是遍历！把 key 带入这个函数得到的数值就是 value 所在的地址，通过这个地址去找到 value，是不是不要傻傻的循环呢。  

我们用 hash table 来做，这里需要先将数组的每个元素保存为 hash 的 key，下标保存为 hash 的 value。这样如果你需要找一个 target - nums[j]，这个 target - nums[j] 就是一个 key 了，那么你就先判断 target - nums[j] 在不在 hash map 里面，如果在就可以找到他的 value，这个 value 也就是 target - nums[j] 在数组中的<strong>下标</strong>。此时的时间复杂度就是O(1)!  

但是要注意，因为同一个元素不能用两次，所以要判断找到的元素应该不是当前元素才行。 
```  java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer,Integer> map = new HashMap<>();
        for(int i = 0;i < nums.length;i++){
            map.put(nums[i],i);
        }
        for(int j = 0;j < nums.length;j++){
            int temp = target-nums[j];
            if(map.containsKey(temp) && (map.get(temp) != j)){//
                return new int[]{j,map.get(temp)};
            }
        }
        throw new IllegalArgumentException("No two sum solution");
    }
}                                           
```
时间复杂度：用了 hash map，所以少了一个 for 循环，变成 O(n)  
空间复杂度：由于开辟了一个 hash map，用空间换取了时间，空间复杂度由 O(1) 变成 O(n).
#### 方法三  
只需要一次循环 + hash map,就可以遍历所有的两两一对，然后找出最终满足条件的结果。比如一个数组[2,3,5,6,7]，最开始 map 中是空的，然后 map.put(2),2 和空的 map 去配比，然后 map.add(3),3 就和 map 中已经存在的 2 去配比，然后 map.add(5),5 就和 map 中存在的 2，3 去配比......这样你会发现，只用了一个 for 循环，就遍历了所有两两一对。  

详细举个例子就是：当你 map.add(5) 的时候，这个 5 肯定先去 和 2，3 配比，也就是说 5 前面的就此遍历了一遍，5 后面的每个元素进来 map 的时候肯定会和 5 来一次配比，那么这个 5 就和前面、后面除了自己的元素进行了配比。  
```  java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer,Integer> map = new HashMap<>();
        
        for(int j = 0;j < nums.length;j++){
            int temp = target-nums[j];
            if(map.containsKey(temp)){//先判断 map 中有没有满足条件的，这里很简单，因为用这种方法自己都还没进入 map，所以不需要检查是不是两个元素一样。如果 map 中有则立即 return。
                return new int[]{j,map.get(temp)};
            }
            map.put(nums[j],j);
        }
        throw new IllegalArgumentException("No two sum solution");
    }
}                                           
      
```
时间复杂度：照样依次 for 循环，所以为 O(n).  
空间复杂度：用了 hash map 存放数组元素，O(n).  
### 总结  
学了一招用 hash map 来消弱一层 for 循环，因为 for + for < for * for,所以这样可以降低时间复杂度，但是要牺牲空间来创建 hash map 来存数组元素。  

完

title: 以给定概率权值从一个数组中选出一个数
date: 2018-02-07 17:42:42
tags: 
- 概率
categories:
- 算法

description: 从n个数中随机选择一个数，要求抽出来的数出现的概率与给定的权值对应。

---
# 问题描述
假设n个数是[1,2,3,4,5,6,7,8,9,10]，权值对应为[20,7,3,1,12,12,17,19,4,5]（这里权值之和刚好等于100），那么随机抽取1000000次，1出现的概率应该是20%即200000次左右、2出现的概率为7%，即70000次左右，以此类推。
<!--more-->
# 建模
将权值数组累加构造成一个区间数组，即[20,27,30,31,43,55,72,91,95,100]
分别表示10个区间[0,20),[20,27),...,(95,100]，区间的长度对应权值的大小。然后随机生成一个范围是[0,100]的整数，那么这个数落在某个区间的概率跟这个区间对应的额权值相等，然后选这个区间的权值对应的整数就是所求。
算法的代码：
```

public class RandomChooseOnWeight {

    public static class Chooser {
        private int[] nums;
        private int[] weights;
        private int[] ranges;
        private Random random = new Random();
        private int maxR = 0;

        public Chooser(int[] nums, int[] weights) {
            this.nums = nums;
            this.weights = weights;
            buildRange();
        }

        private void buildRange() {//构造权值区间
            ranges = new int[weights.length];
            for (int i = 0; i < ranges.length; i++) {
                if (i > 0) {
                    ranges[i] = ranges[i - 1] + weights[i];
                } else {
                    ranges[i] = weights[i];
                }
            }
            maxR = ranges[ranges.length - 1];
        }

        private int randomIndex() {//获取随机区间
            int range = random.nextInt(maxR + 1);
            return indexInRange(range);
        }

        private int indexInRange(int range) {//查询随机数所在的区间
            for (int i = 0; i < ranges.length; i++) {
                if (ranges[i] >= range) {
                    return i;
                }
            }
            return -1;
        }

        public int chose() {//随机选择一个
            return nums[randomIndex()];
        }
    }

    public static void main(String[] args) {
        int[] nums = new int[]{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
        int[] weights = new int[]{20, 7, 3, 1, 12, 12, 17, 19, 4, 5};
        Chooser chooser = new Chooser(nums, weights);
        int COUNT = 1000000;
        Map<Integer, Integer> distributeMap = new HashMap<>();
        for (int i = 0; i < COUNT; i++) {
            int num = chooser.chose();
            Integer count = distributeMap.get(num);
            if (count == null) {
                count = 0;
            }
            count++;
            distributeMap.put(num, count);
        }
        for (int i = 0; i < nums.length; i++) {
            Integer count = distributeMap.get(nums[i]);
            System.out.println("num " + nums[i] + " :" + (count != null ? count * 1.0 / COUNT : "0"));
        }
    }
}
```

main函数有运行1000000次的结果是：
```
num 1 :0.208496
num 2 :0.0695
num 3 :0.029214
num 4 :0.009872
num 5 :0.118575
num 6 :0.118941
num 7 :0.168433
num 8 :0.188238
num 9 :0.03953
num 10 :0.049201
```
在四舍五入保留两位小数的结果跟权值是匹配的。

# 优化算法
由于`ranges`数组是递增的，所以，确定随机数的分组的方法`indexInRange(int range)`可以采用二分搜索算法的变种进行，这样，在`nums`长度很大时，可以大幅优化计算时间。
```
    //TODO
```

# 扩展到从n个数中随机挑选m个
可以这样实现：随机挑选一个数字，如果这个数字已经选出了，则挑选另外一个，知道挑满m个为止。
（这个算法应该是正确的，数学上的证明忘记了）

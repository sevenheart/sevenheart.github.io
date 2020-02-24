---
type: 排序算法
title: 希尔排序
date: 2020-02-17
category: sortAlgorithm
tags:
- 排序算法
---

### 希尔排序

##### 复杂度：

- **时间复杂度：**O(n log n）
-  **空间复杂度：**o(1)
- 稳定性：不稳定

步骤1：选择一个增量序列t1，t2，…，tk，其中ti>tj，tk=1；
步骤2：按增量序列个数k，对序列进行k 趟排序；
步骤3：每趟排序，根据对应的增量ti，将待排序列分割成若干长度为m 的子序列，分别对各子表进行直接插入排序。仅增量因子为1 时，整个序列作为一个表来处理，表长度即为整个序列的长度。

![](../../../sources/img/Algorithm/sort/xier.jpg)

##### Java代码实现：

```java
 //希尔排序
    public static void shellSort1(int[] arry){
        int grep = arry.length/2;
        //只要还能细分就分，分一次排序一次
        while (grep>0) {
            for(int i = 0;i < arry.length; i++){
                int temp  = arry[i];
                int index = i-grep;
                //开始插入排序
                while(index >= 0 && temp < arry[index]){
                    //数组后移
                    arry[index+grep] = arry[index];
                    //再前一个坐标
                    index = index - grep;
                }
                //这个数据已经插入排序完成了找到了插入的位置就把值插入，换下一个数据了。
                arry[index+grep] = temp;
                Object o = new Object();
                System.gc();

            }
            grep = grep/2;

        }
        System.out.println("希尔排序的结果是"+Arrays.toString(arry));
    }
```




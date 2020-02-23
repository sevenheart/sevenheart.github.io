---
type: 排序算法
title: 归并排序
date: 2020-02-15
category: sortAlgorithm
tags:
- 排序算法
---

### 归并排序

##### 复杂度：

- **时间复杂度：O(nlogn)**
-  **空间复杂度：O(N)**，归并排序需要一个与原数组相同长度的数组做辅助来排序
- 稳定性：归并排序是稳定的排序算法，值相等的时候两个元素的相对位置不变。

##### 步骤一：

![../../../../sources/img/Algorithm/sort/guibingone.png]()



##### 步骤二：

![../../../../sources/img/Algorithm/sort/guibingtwo.png]()



##### Java代码实现：

```java
 //归并排序
    public static  void  guibing(int left,int right,int[] arr){
        //分到最小了
        if(left >= right){
            return ;
        }
        //先左边
        guibing(left,(right-left)/2+left,arr);
        //后右边
        guibing((right-left)/2+1+left,right,arr);
        //排序左右的数组
        int initL = left;
        int initR = right;
        int mid = (right-left)/2+1+left;
        int usemid = mid;
        int[] temp = new int[right-left+1];
        int num = 0;//控制填入数组
        while (left <= mid-1 && usemid <= right){
            if(arr[left] <=  arr[usemid]){
                temp[num] = arr[left];
                left++;
                num++;
            }else {
                temp[num]  = arr[usemid];
                usemid++;
                num++;
            }
        }
        //没比较的直接加入
        if(left <= mid-1){
            //左边没比完直接加入
            while (left<=mid-1){
                temp[num] = arr[left];
                num++;
                left++;
            }

        }else{
            //右边没比较完直接加入
            while (usemid<=right){
                temp[num] = arr[usemid];
                num++;
                usemid++;
            }
        }
        //腹值给真的arr
        for(int i = initL;i <= initR ; i++ ){
            arr[i] = temp[i-initL];
        }
    }
```




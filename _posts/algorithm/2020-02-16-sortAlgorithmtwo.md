---
type: 排序算法
title: 快速排序
date: 2020-02-16
category: sortAlgorithm
tags:
- 排序算法
---

### 归并排序

### 快速排序

##### 复杂度：

- **时间复杂度：**O(n log n）
-  **空间复杂度：**o(nlogn)
- 稳定性：不稳定

##### 步骤一：

数组中找一个基准值

##### 步骤二：

使得小于基准值的值放在基准值的左边，将大于基准值的值放在基准值的右边。依次从左右数组再找基准值去进行排序。直到无法再分时说明排序已经完成了。

![../../../../sources/img/Algorithm/sort/quicksort.png]()



##### Java代码实现：

```java
 //快速排序
    public  static  void  quickSort(int left,int right,int[] arr){
        int index = left;//索引坐标
        int indexValue = arr[left];//基准值
        int head = left ;//头指针
        int tail = right;//尾指针
        //排序结束
        if(head >= tail){
            return;
        }
        //循环结束就以基准值分好了
        while (head < tail) {
            //从右向左比较
            while (head < tail && arr[tail] > indexValue) tail--;
            //判断是不是到头了或者该判断左边了
            if (head < tail) {
                arr[head] = arr[tail];
                head++;
            } else {
                break;
            }
            while (head < tail && arr[head] < indexValue) head++;
            if (head < tail) {
                arr[tail] = arr[head];
                tail --;
            } else {
                break;
            }

        }
        arr[head] = indexValue;
        quickSort(left,head,arr);
        quickSort(head+1,right,arr);
    }
```




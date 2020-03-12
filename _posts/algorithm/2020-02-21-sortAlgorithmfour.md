---
type: 排序算法
title: 小顶堆排序
date: 2020-02-21
category: sortAlgorithm
tags:
- 排序算法
---

### 小顶堆排序

**小顶堆**为左右孩子都小于父节点的二叉堆。

**注意：**以数组存储的二叉树，某个节点的左孩子 = 当前节点的下标*2+1， 某个节点的右孩子 = 当前节点的下标 * 2+2。

**步骤：**

#### 一、构建小顶堆

1. 二叉堆用数组来表示，可以根据其特点找到相关的位置。
2. 2.从最后一个非叶子节点开始做下沉调整，每次和左右子树比较小的交换位置，直到交换到叶子节点或者当前节点已经完全小于左右孩子就不做交换了。
3. 之后开始向前推做下一个节点的下沉操作，即当前节点下标-1；

##### Java代码下沉方法及初始化小顶堆的实现：

```java

 /**                                                              
  * 初始化一个二叉堆                                                      
  * @return                                                       
  */                                                              
 public static void init(int[] arr){                              
     //初始化从最后一个非叶子节点开始下沉。                            
     for(int i = arr.length/2-1;i >= 0; i--){                     
         downWay(i,arr,arr.length);                               
     }        
 }                                                                

/**                                                                                                          
 * 下沉方法,构建一个小顶堆大的下沉                                                                                          
 */                                                                                                          
public static void downWay(int i,int[] arr,int length){                                                      
    int temp = arr[i];                                                                                       
    int preIndex = 2*i+1;                                                                                    
    while (preIndex < length ){                                                                              
        //大于小于左子树                                                                                            
        if(preIndex+1 < length && arr[preIndex+1] < arr[preIndex]){                                          
            preIndex++;                                                                                      
        }                                                                                                    
        //当前要下沉的元素小于两个子元素中的最小值。就不做交换跳出                                                                       
        if(temp < arr[preIndex]) break;                                                                      
        //否则就单项赋值。                                                                                           
        arr[i] = arr[preIndex];                                                                              
        //从交换后的地方继续算出下一个子孩子                                                                                  
        i = preIndex;                                                                                        
        preIndex = 2*preIndex+1;                                                                             
    }                                                                                                        
    //下沉完毕，找到真正的地方开始交换，这里是一个优化避免了每次都交换，只有最后找到位置才交换                                                                                       
     arr[i] = temp;                                                                          
}                                                                                                            
```

#### 小顶堆的排序

**原理：**每次将最后一个节点和第一个节点交换，然后调整小顶堆，下一次再拿第一个节点和最后一个节点做交换直到只剩第一个节点，这样就完成了排序

**小顶堆的排序方法**

```java
/**                                                              
 * 排序方法                                                          
 * 对于一个初始化好的堆，奖最后一个元素节点与堆顶元素节点交换，并维护堆。                           
 *                                                               
 * @param arr                                                    
 */                                                              
public static void sort(int[] arr){                              
    int length = arr.length;                                     
    //开始交换维护                                                     
    while (length > 0){                                          
        int temp = arr[0];                                       
        arr[0] = arr[length-1];                                  
        arr[length-1] = temp;                                    
        length = length-1;                                       
        downWay(0,arr,length);                                   
    }
}
```

#### 小顶堆的节点删除原理

​	和二叉堆的排序原理思路一样，每次删除第一个节点，然后把最后一个节点放在第一个节点的位置开始调整。

#### 小顶堆的节点插入原理

​	把节点插入到最后一个节点的后面，然后**上浮**与调整二叉堆的上浮相反。
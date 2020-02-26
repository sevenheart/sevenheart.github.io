---
type: 经典算法题目
title: 八皇后问题
date: 2020-02-18
category: Algorithm
---

### 希尔排序

**问题描述：**在8×8格的国际象棋上摆放八个皇后，使其不能互相攻击，即任意两个皇后都不能处于同一行、同一列或同一斜线上，问有多少种摆法。

**思路：**

1. N*N的棋盘放置N个皇后要找出摆法可以通过递归回溯的方式。

2. 为满足要求那么一行必定为一个皇后。

3. 从第一个格子开始验证是否满足，满足就到下一层去一个一个试。如果不满足就往后一个格子再试，如果到达边界还是不满足那么就说明再没有解法了，只能返回上一层，再从上一层的下一个格子开始尝试直到尝试出所有解法。

4. 可以将棋盘拟定为一个二维数组，0表示未放置棋子，放置后为1。

##### 解出一种摆法的代码：

坐标检查是否可以摆放的方法：

```java
    //根据xy判断这里是否能放皇后
    public  static boolean  check(int x,int y){
        //纵向判断
        for(int i = 0;i < qipan.length ;i++){
            //横向有皇后就不能放置
            if(qipan[i][y] == 1){ return  false;}
        }
        //纵向判断
        for(int i = 0;i < qipan[0].length ;i++){
            //横向有皇后就不能放置
            if(qipan[x][i] == 1){ return  false;}
        }
        //(左下)
        int i = x;
        int j = y;
        while (j >= 0 && i < qipan.length) {
            //横向有皇后就不能放置
            if (qipan[i][j] == 1) {
                return false;
            }
            i = i+1;
            j = j-1;
        }
        //纵向判断(右上)
        i = x;
        j = y;
        while (j <qipan[0].length  && i >= 0) {
            //横向有皇后就不能放置
            if (qipan[i][j] == 1) {
                return false;
            }
            i = i-1;
            j = j+1;
        }
        //纵向判断(右下)
        i = x;
        j = y;
        while (i <qipan.length  && j <qipan[0].length) {
            //横向有皇后就不能放置
            if (qipan[i][j] == 1) {
                return false;
            }
            i = i+1;
            j = j+1;
        }
        //纵向判断(左上)
        i = x;
        j = y;
        while (i >= 0  && j >= 0) {
            //横向有皇后就不能放置
            if (qipan[i][j] == 1) {
                return false;
            }
            i = i-1;
            j = j-1;
        }

        return  true;
    }
```

递归回溯的方法：

```java
public  static boolean eightQueen(int y){
        //如果能放就进入下一层的递归
        if(y>=qipan[0].length){
            return  true;
        }
        //当前行验证每个格子
        for (int i = 0;i<qipan.length;i++){
            //验证成功
            if(check(i,y)){
                qipan[i][y] = 1;
                //下一层
                if(eightQueen(y+1)){
                    //ture表示下面的都找了
                    return true;
                }
            }
        }
        return  false;

    }
```


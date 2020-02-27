---
type: 经典算法题目
title: 字符串匹配算法
date: 2020-02-19
category: Algorithm
---

问题描述：AB两个字符串，判断B是否为A的子串如果是第一次出现的位置是神马？

### 第一种：BM匹配算法（暴力匹配）

**思路：**

1. 比长度如果B小于A才开始比较。

2. 从A的第一位开始与B的每一位开始比较，如果不匹配了就开始从A的下一位匹配，直到A长度不足。

3. 时间复杂度O(MN)

     **实现代码：**

```java
public static  int matchString(String a,String b){
        if(a.length()<b.length()) return  -1;
        //开始匹配
        char[] char1 = a.toCharArray();
        char[] char2 = b.toCharArray();
        for(int i = 0; i < char1.length;i++){
            int temp = i;
            for (int j = 0;j < char2.length && temp<char1.length ;j++){
                //匹配成功继续匹配；
                if(char1[temp] != char2[j]) {
                    break;
                }
                //匹配成功继续向后匹配
                temp++;
                if(j == char2.length-1 ){
                    return  i;
                }
            }

        }
        return  -1;
    }
```

### 第二种：RK算法(BF的改进)

当某些极端情况下BF算法效率低，比如一长串字符串匹配两个字符但是要匹配的位置在最后。前面的比较就有点多了。

思路：采用**hashcode比较**的形式去匹配。

1. 自己定义一个生产hashcode的函数：比如每个字符表示一个数字hashcode就是他们的数字之和。
2. 从主串中每次拿连续的模板字符串的长度的字符串，然后将他做hash生成hashcode后和模板字符串生成的hashcode做比较。
3. 如果不同就后移一位继续做hashcode，这里可以根据hash函数的生成规则去减去第一个字符，加上尾部新加的字符来减少hashcode的运算时间。
4. 如果相同那么考虑hash冲突的情况再采用BF算法的形式去一个一个比较这样时间复杂度可以降低到O（n）

**实现代码：**

```JAVA
 //自定义hashcode生成规则
    public static int hashcode(String str){
        //自定义hashcode生成规则
        char[] chars = str.toCharArray();
        int j = 0;
        for(int i = 0;i < chars.length;i++){
            j = j+(int)chars[i];
        }
        return j;
    }
```

BF流程

```java
    //匹配
    public static int RFForce(String a,String b){
        if(a.length()<b.length()) return -1;
        //计算模板字符串的hashcode
        int bhashcode = hashcode(b);
        int ahashcode = hashcode(a.substring(0,b.length()));
        char[] aChars = a.toCharArray();
        char[] bChars = b.toCharArray();
        for(int i = 0;i < a.length();i++){
            if(ahashcode == bhashcode){
                //开始比较
                for(int j = 0 ; j < b.length();j++){
                    if(aChars[j] != bChars[j]){
                        break;
                    }else if(j == bChars.length-1){
                        return  i;
                    }
                }

            }
            ahashcode = ahashcode - (int)aChars[i] + (int)aChars[i + b.length()];

        }
        return -1;
    }
```

### 第三钟：BM匹配算法

#### 坏字符规则；

定义：将B串与A串匹配，从匹配处从后往前的第一个不匹配的字符就是坏字符

例如：A="GTTATAGCTGGTAGCGGCGAA"与B=“GTAGCGGCG”中A的第八个字符‘T’就是坏字符。

思路：

1. 从B串中找到与坏字符相等的字符，下一次匹配移动直接将AB串的这两个位置匹配。
2. 如果匹配了其他的地方不匹配那么就后移一位。
3. 如果B串中没有与A匹配的字符串那么就直接从A的坏字符的下一位开始匹配。

**算法实现：**

```

```

#### 好后缀规则

定义：第一轮比较找到AB完全相等的后缀就是好后缀，之后去找B串中的好后缀，之后每次移动就以好后缀对齐为基础去移动匹配。但是如果B串中没有好后缀也不能直接从A的好后缀之后在开始匹配，应该将好后缀继续细化在去移动到细化后的好后缀去开始匹配。

例如：A="TGGGCGAGCGGAA"B="CGAGCG",此时好后缀是GCG但是下次比较不能从A的GCG之后开始，应该将GCG细化为CG，此时下一轮比较就是从A的第四个字符串开始匹配因为此时B有和他一样的前缀了。

**对于好坏字符串的结合应该在每一轮比较完成后判断用那种方法挪动的位置多就用那种方法**

### 第四种：KMP算法

[KMP算法]: https://mp.weixin.qq.com/s/3gYbmAAFh08BQmT-9quItQ


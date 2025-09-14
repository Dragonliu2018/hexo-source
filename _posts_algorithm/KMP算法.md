---
title: KMP算法
tags:
  - KMP
categories:
  - 06-字符串
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-19 13:21:19
---

# 1 算法思想

## 1.1 思路

**解决问题**：当主串中第 i 个字符与模式中第 j 个字符失配（即比较不相等）时，主串中第 i 个字符（i 指针不回溯）应与模式中哪个字符再比较。

1. $next[j]$：模式中第 j 个字符失配时，在模式中重新与主串比较的字符位置（最长公共前后缀长度+1，下标从 1 开始）

   $\text { next }[j]=\left\{\begin{array}{ll}
   0 & \text { 当 } j=1 \text { 时 } \\
   \operatorname{Max}\left\{k \mid 1<k<j \text { 且 }^{\prime} p_{1} \cdots p_{k-1}^{\prime}={ }^{\prime} p_{j-k+1} \cdots p_{j-1}{ }^{\prime}\right\} &
   \text { 当此集合不空时 } \\
   1 & \text { 其他情况 }
   \end{array}\right.$

2. 计算next数组：仅取决于模式串本身而和相匹配的主串无关

   * 仿照kmp算法，如果$p[j] != p[next[j]]$，那么$next[j+1]$的可能次大值为$next[next[j]] + 1$，依次类推即可高效求出$next[j+1]$

3. 若在匹配过程中$s_i=p_j$，则 i 与 j 分别增 1；

4. 否则，i 不变，j 退至 next[j] 的位置再比较；若 j 退到值为 0（即模式串的第一个字符失配），则从主串的下一个字符$s_{i+1}$起与模式串重新开始匹配；

5. 重复执行step3、step4，直至结束。

## 1.2 优化

修正next数组，$next[j]=k$，若模式中$p[j]=p[k]$，则$next[j] = next[k]$

## 1.3 算法分析

* **时间复杂度**：$O(m+n)$
* **空间复杂度**：next数组，$O(m)$

# 2 代码实现

## 2.1 C++

```c++
#include <iostream> 
typedef long long ll;
using namespace std;
// string、next数组下标从1开始
const int maxn = 1e4;
int next2[maxn];
int nextval[maxn];

// 计算next数组
void get_next(string p) {
    int i = 1, j = 0;
    next2[1] = 0;
    while (i < p.size()) {
        if (j == 0 || p[i] == p[j]) {
            i++;
            j++;
            next2[i] = j;
        } else {
            j = next2[j];
        }
    }
}
//计算nextval数组
void get_nextval(string p) {
    int i = 1, j = 0;
    nextval[1] = 0;
    while (i < p.size()) {
        if (j == 0 || p[i] == p[j]) {
            i++;
            j++;
            if (p[i] != p[j])
                nextval[i] = j;
            else
                nextval[i] = nextval[j];
        }
        else {
            j = nextval[j];
        }
    }
}
// KMP算法
int kmp(string s, string p) {
    //get_next(p);//计算next数组
    get_nextval(p);
    int i = 0, j = 1;//主串从第i个字符之后匹配
    while (i < s.size() && j < p.size()) {
        if (j == 0 || s[i] == p[j]) {//模式串首字符失配或相同继续匹配
            i++;
            j++;
        } else {//失配
            j = nextval[j];
        }
    }
    if (j >= p.size())//找到
        return i - p.size() - 1;
    else
        return 0;
}
int main() {
    std::ios::sync_with_stdio(false);
    string s = " acabaabaabcacaabc", t = " abaabcac";
    cout << kmp(s, t);
    /*for (int i = 1; i < t.size(); i++) {
        cout << next2[i] << endl;
    }*/
    return 0;
}
```

## 2.2 Java

```java
class KMP {
	private int maxn = 1000;
	private int[] next = new int[maxn];
	private int[] nextval = new int[maxn];
	
	//构造函数
	public KMP() {
		
	}
	//计算next数组
    public void get_next(String p) {
        int i = 1, j = 0;
        next[1] = 0;
        while (i < p.length()) {
            if (j == 0 || p.charAt(i) == p.charAt(j)) {
                i++;
                j++;
                next[i] = j;
            } else {
                j = next[j];
            }
        }
    }
    //计算nextval数组
    public  void get_nextval(String p) {
        int i = 1, j = 0;
        nextval[1] = 0;
        while (i < p.length() - 1) {
            if (j == 0 || p.charAt(i) == p.charAt(j)) {
                i++;
                j++;
                if (p.charAt(i) != p.charAt(j))
                    nextval[i] = j;
                else
                    nextval[i] = nextval[j];
            }
            else {
                j = nextval[j];
            }
        }
    }
    // KMP算法
    public int kmp(String s, String p) {
        //get_next(p);//计算next数组
        get_nextval(p);
        int i = 0, j = 1;//主串从第i个字符之后匹配
        while (i < s.length() && j < p.length()) {
            if (j == 0 || s.charAt(i) == p.charAt(j)) {//模式串首字符失配或相同继续匹配
                i++;
                j++;
            } else {//失配
                j = nextval[j];
            }
        }
        if (j >= p.length())//找到
            return i - p.length() - 1;
        else
            return 0;
    }
}
public class test1 {
	
    public static void main(String[] args) {
        // TODO自动生成的方法存根
    	KMP test = new KMP();
        String s = " acabaabaabcacaabc", t = " abaabcac";
        System.out.println(test.kmp(s, t));
    }
}
```

# 3 参考

* [KMP算法之求next数组代码讲解](https://www.bilibili.com/video/BV16X4y137qw?from=search&seid=428213560995313343)
* 

---
title: "Splitting and Merging--区域分裂与合并算法"
date: 2015-12-06 00:00:00
category: 课程研究
tags: [Splitting,Merging,图像分裂与合并,spam]
---
# 区域分裂与合并算法

**基本思想：**先确定一个分裂合并的准则,即区域特征一致性的量度,当图像中某个区域的特征不一致时就将该区域分裂成4个相等的子区域,当相邻的子区域满足一致性特征时则将它们合成一个大区域,直至所有区域不再满足分裂合并的条件为止.当分裂到不能再分的情况时，分裂结束，然后它将查找相邻区域有没有相似的特征，如果有就将相似区域进行合并，最后达到分割的作用。在一定程度上区域生长和区域分裂合并算法有异曲同工之妙，互相促进相辅相成的，区域分裂到极致就是分割成单一像素点，然后按照一定的测量准则进行合并，在一定程度上可以认为是单一像素点的区域生长方法。区域生长比区域分裂合并的方法节省了分裂的过程，而区域分裂合并的方法可以在较大的一个相似区域基础上再进行相似合并，而区域生长只能从单一像素点出发进行生长（合并）。


## 分裂原理

### 分裂的过程

```
Set ProcessList = IMAGE
Repeat
	 取出 ProcessList 中第一个元素
	 IF 区域内是均匀的，则加入 RegionList
	 Else 把区域分成四个自区域，然后都加到 ProcessList
Until ( ProcessList 中没有元素)
```
### 如何判断是否可以分裂

对于一个灰度图，如果说某一区域是均匀的，那么该区域所有灰度值的标准差需要小于一定的阈值，标准差的公式如下：

$$  \sigma  = \left [ \frac{1}{N-1} \sum_{j=1}^{N} \left ( x_{j}-\bar{x} \right )^{2}\right ] $$



## 合并原理

### 合并的过程

```
Put all regions on ProcessList
Repeat
	 从 ProcessList 每次取出一个元素
	 遍历列表中的元素，找到相似的元素 (homogeneity criterion)
	 If 相邻 then
 		 合并区域 然后上面的步骤;
until (不能合并为止)
```
### 如何判断可以合并


## 分裂与合并

如果只使用拆分，最后的分区可能会包含具有相同性质的相邻区域。这种缺陷可以通过进行拆分的同时也允许进行区域聚合来得到矫正。就是说，只有在P(Rj∪Rk)=TRUE时，两个相邻的区域Rj和Rk才能聚合。

前面的讨论可以总结为如下过程。在反复操作的每一步，我们需要做：

```
1.对于任何区域Ri，如果P(Ri)=FALSE，就将每个区域都拆分为4个相连的象限区域。
2.将P(Rj∪Rk)=TRUE的任意两个相邻区域Rj和Rk进行聚合。
3.当再无法进行聚合或拆分时操作停止。
```

开始时将图像拆分为一组图象块。然后对每个块进一步进行上述拆分，但聚合操作开始时受只能将4个块并为一组的限制。这4个块是四叉树表示法中节点的后代且都满足谓词P。当不能再进行此类聚合时，这个过程终止于满足步骤2的最后的区域聚合。在这种情况下，聚合的区域可能会大小不同。这种方法的主要优点是对于拆分和聚合都使用同样的四叉树，直到聚合的最后一步。

## 实验分析

### Matlab源码

1. 源码一:splitmerge.m

```Matlab
function g = splitmerge( f,mindim,fun )
Q = 2^nextpow2(max(size(f)));
[M,N] = size(f);
f = padarray(f,[Q-M,Q-N],'post');
S = qtdecomp(f,@split_test,mindim,fun);
Lmax = full(max(S(:)));
g = zeros(size(f));
MARKER = zeros(size(f));
for K = 1:Lmax
    [vals,r,c] = qtgetblk(f,S,K);
    if ~isempty(vals)
        for I = 1:length(r)
            xlow = r(I);
            ylow = c(I);
            xhigh = xlow + K - 1;
            yhigh = ylow + K - 1;
            region = f(xlow:xhigh,ylow:yhigh);
            flag = feval(fun, region);
            if flag
                g(xlow:xhigh,ylow:yhigh) = 1;
                MARKER(xlow,ylow) = 1;
            end
        end
    end
end
g = bwlabel(imreconstruct(MARKER,g));
g = g(1:M,1:N);
end
```

2. 源码二:split_test.m


```Matlab
function v = split_test( B,mindim,fun )
k = size(B,3);
v(1:k) = false;
for I = 1:k
    quadregion = B(:, :, I);
    if size(quadregion,1) <= mindim
        v(I) = false;
        continue
    end
    flag = feval(fun,quadregion);
    if flag
        v(I) = true;
    end
end
end
```

3. 源码三:predicate.m

```Matlab
function flag = predicate( region )
sd = std2(region);
m = mean2(region);
flag = (sd > 10) & (m > 0) & (m < 255);
end
```
### 效果图

#### 实验一

1. 原图

![原始图](http://ww1.sinaimg.cn/bmiddle/9d2c4511gw1eyr1am4g7gj20e80e8abn.jpg)

2. 处理后的图

![图像分割后](http://ww1.sinaimg.cn/bmiddle/9d2c4511gw1eyr1als9rij20it0gm74p.jpg)


#### 实验二

1. 原图

![原始图](http://ww3.sinaimg.cn/bmiddle/9d2c4511gw1eyr1anc2rmj20bo09iq31.jpg)

2. 处理后的图

![图像分割后](http://ww3.sinaimg.cn/bmiddle/9d2c4511gw1eyr1ammyuyj20bo09ijrn.jpg)

#### 实验三

1. 原图

![原始图](http://ww2.sinaimg.cn/bmiddle/9d2c4511gw1eyr1aoldl1j20ei0hbwer.jpg)

2. 处理后的图

![图像分割后](http://ww4.sinaimg.cn/bmiddle/9d2c4511gw1eyr1ao0fhpj20ei0hbgmb.jpg)

## 参考文章

[1] http://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/MARBLE/medium/segment/split.htm

[2] http://blog.csdn.net/bagboy_taobao_com/article/details/5666109

[3] http://blog.csdn.net/bagboy_taobao_com/article/details/5666091

[4] 冈萨雷斯-数字图像处理（MATLAB版）
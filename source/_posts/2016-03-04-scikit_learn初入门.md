---
title: "scikit_learn初入门"
date: 2016-03-04 00:00:00
category: 科学计算
tags: [机器学习,聚类算法]
---

# scikit_learn初入门(更新于2016/10/9)

> 标准的官方网址：http://scikit-learn.org/stable/

> `scikit_learn`在kaggle比赛中看到和pandas结合起来用的非常多，也确实在机器学习方面这里面的算法非常的全面。


## scikit_learn的介绍

|   基本领域   |
| :------: |
|  分类--Classification  |
|  回归--Regression  |
|  聚类--Clustering  |
|  降维--Dimensionality reduction  |
|  模型选择--Model selection |
|  预处理--Preprocessing  |

## 如何快速安装python库

> 无论是在`Mac os`，'Linux'还是'Windows'，最为方便管理python扩展的工具是`pip`命令，如果不知道怎么使用和安装`pip`，请自行百度或谷歌。

## skLearn使用入门

> 下面sk-learning的基本用法将逐步的完善补充，主要还是根据自己的学习进度进行推进

### **分类--Classification**

#### 逻辑回归

> 参考文档：[http://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html](http://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html)

```python
#coding=utf-8

import matplotlib.pyplot as plt
import numpy as np
from matplotlib.colors import ListedColormap


def plot_decision_regions(X, y, classifier, test_idx=None, resolution=0.02):
    # setup marker generator and color map
    markers = ('s', 'x', 'o', '^', 'v')
    colors = ('red', 'blue', 'lightgreen', 'gray', 'cyan')
    cmap = ListedColormap(colors[:len(np.unique(y))])
    # plot the decision surface
    x1_min, x1_max = X[:, 0].min() - 1, X[:, 0].max() + 1
    x2_min, x2_max = X[:, 1].min() - 1, X[:, 1].max() + 1
    xx1, xx2 = np.meshgrid(np.arange(x1_min, x1_max, resolution), np.arange(x2_min, x2_max, resolution))
    Z = classifier.predict(np.array([xx1.ravel(), xx2.ravel()]).T)
    Z = Z.reshape(xx1.shape)
    plt.contourf(xx1, xx2, Z, alpha=0.4, cmap=cmap)
    plt.xlim(xx1.min(), xx1.max())
    plt.ylim(xx2.min(), xx2.max())
    # plot class samples
    for idx, cl in enumerate(np.unique(y)):
        plt.scatter(x=X[y == cl, 0], y=X[y == cl, 1],alpha=0.8, c=cmap(idx),marker=markers[idx], label=cl)
    # highlight test samples
    if test_idx:
        X_test, y_test = X[test_idx, :], y[test_idx]
        plt.scatter(X_test[:, 0], X_test[:, 1], c='', alpha=1.0, linewidth=1, marker='o', s=55, label='test set')

from sklearn import datasets
import numpy as np
from sklearn.cross_validation import train_test_split

iris = datasets.load_iris()# 由于Iris是很有名的数据集，scikit-learn已经原生自带了。
X = iris.data[:, [2, 3]]
y = iris.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=0)

# 为了追求机器学习和最优化算法的最佳性能，我们将特征缩放
from sklearn.preprocessing import StandardScaler
sc = StandardScaler()
sc.fit(X_train)# 估算每个特征的平均值和标准差
X_train_std = sc.transform(X_train)
# 注意：这里我们要用同样的参数来标准化测试集，使得测试集和训练集之间有可比性
X_test_std = sc.transform(X_test)

X_combined_std = np.vstack((X_train_std, X_test_std))
y_combined = np.hstack((y_train, y_test))

'''
# 训练感知机模型
from sklearn.linear_model import Perceptron
# n_iter：可以理解成梯度下降中迭代的次数
# eta0：可以理解成梯度下降中的学习率
# random_state：设置随机种子的，为了每次迭代都有相同的训练集顺序
ppn = Perceptron(n_iter=40, eta0=0.1, random_state=0)
ppn.fit(X_train_std, y_train)
 
# 分类测试集，这将返回一个测试结果的数组
y_pred = ppn.predict(X_test_std)
# 计算模型在测试集上的准确性，我的结果为0.9，还不错
accuracy_score(y_test, y_pred)
'''
from sklearn.linear_model import LogisticRegression
lr = LogisticRegression(C=1000.0, random_state=0)
lr.fit(X_train_std, y_train)
lr.predict_proba(X_test_std[0,:]) # 查看第一个测试样本属于各个类别的概率
plot_decision_regions(X_combined_std, y_combined, classifier=lr, test_idx=range(105,150))
plt.xlabel('petal length [standardized]')
plt.ylabel('petal width [standardized]')
plt.legend(loc='upper left')
plt.show()
```

**分类结果**

![逻辑回归](http://www.2cto.com/uploadfile/Collfiles/20160502/20160502112253360.png)

### **回归--Regression**

#### 一元线性回归

```python
import numpy as np
from sklearn.linear_model import LinearRegression
#一元线性回归
X = [[6], [8], [10], [14], [18]]
y = [[7], [9], [13], [17.5], [18]]
# 创建并拟合模型
model = LinearRegression()
model.fit(X, y)
print('预测一张12英寸匹萨价格：$%.2f' % model.predict(np.array([12]).reshape(-1, 1))[0])
```

#### 多变量线性回归

```python
from sklearn.linear_model import LinearRegression
X = [[6, 2], [8, 1], [10, 0], [14, 2], [18, 0]]
y = [[7], [9], [13], [17.5], [18]]
model = LinearRegression()
model.fit(X, y)
X_test = [[8, 2], [9, 0], [11, 2], [16, 2], [12, 0]]
y_test = [[11], [8.5], [15], [18], [11]]
predictions = model.predict(X_test)
for i, prediction in enumerate(predictions):
    print('Predicted: %s, Target: %s' % (prediction, y_test[i]))
print('R-squared: %.2f' % model.score(X_test, y_test))
```
#### 多项式回归
> 参考文献：
> 1.[多项式回归的文献](http://blog.csdn.net/jairuschan/article/details/7517773/)
> 2.[最小二乘法的WiKi解释](https://zh.wikipedia.org/wiki/%E6%9C%80%E5%B0%8F%E4%BA%8C%E4%B9%98%E6%B3%95)

```python
#coding=utf-8
import numpy as np
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import PolynomialFeatures
import matplotlib.pyplot as plt
def runplt():
    plt.figure()
    plt.title(u'diameter-cost curver')
    plt.xlabel(u'diameter')
    plt.ylabel(u'cost')
    plt.axis([0, 25, 0, 25])
    plt.grid(True)
    return plt
X_train = [[6], [8], [10], [14], [18]]
y_train = [[7], [9], [13], [17.5], [18]]
X_test = [[6], [8], [11], [16]]
y_test = [[8], [12], [15], [18]]
# 建立线性回归，并用训练的模型绘图
# regressor = LinearRegression()
# regressor.fit(X_train, y_train)
xx = np.linspace(0, 26, 100)#计数的100个点
# yy = regressor.predict(xx.reshape(xx.shape[0], 1))
# plt = runplt()
# plt.plot(X_train, y_train, 'k.')
# plt.plot(xx, yy)

plt = runplt()
plt.plot(X_train, y_train, 'k.') #花训练样本集

quadratic_featurizer = PolynomialFeatures(degree=2)  #设置拟合度
X_train_quadratic = quadratic_featurizer.fit_transform(X_train)
X_test_quadratic = quadratic_featurizer.transform(X_test)
regressor_quadratic = LinearRegression()
regressor_quadratic.fit(X_train_quadratic, y_train)
xx_quadratic = quadratic_featurizer.transform(xx.reshape(xx.shape[0], 1))
plt.plot(xx, regressor_quadratic.predict(xx_quadratic), 'r-')

seventh_featurizer = PolynomialFeatures(degree=7)
X_train_seventh = seventh_featurizer.fit_transform(X_train)
X_test_seventh = seventh_featurizer.transform(X_test)
regressor_seventh = LinearRegression()
regressor_seventh.fit(X_train_seventh, y_train)
xx_seventh = seventh_featurizer.transform(xx.reshape(xx.shape[0], 1))
plt.plot(xx, regressor_seventh.predict(xx_seventh))

print('2 r-squared', regressor_quadratic.score(X_test_quadratic, y_test))
print('7 r-squared', regressor_seventh.score(X_test_seventh, y_test))


###下面是官方demo中的方法
# from sklearn.pipeline import Pipeline
# from sklearn.linear_model import (LinearRegression, TheilSenRegressor, RANSACRegressor)
from sklearn.pipeline import make_pipeline
# estimators = [('OLS', LinearRegression()),\
#               ('Theil-Sen', TheilSenRegressor(random_state=42)),\
#               ('RANSAC', RANSACRegressor(random_state=42))]
model = make_pipeline(PolynomialFeatures(2),LinearRegression())
model.fit(X_train,y_train)
COEF = model.named_steps['linearregression'].coef_ #输出训练的多项式系数
print "coefficients",COEF
X_test_quadratic = model.predict(X_test)
# print X_test_quadratic
print('2 r-squared easy method', model.score(X_test_quadratic, y_test))
plt.show()
```
![多项式回归](http://img.blog.csdn.net/20160617233424823)


### **聚类--Clustering**

#### **sklearn 中 make_blobs模块使用**

> 实验中需要生成特定个簇的时候`make_blobs`就很管用了

- 例如要生成5类数据（100个样本，每个样本有2个特征），代码如下：

```python
from sklearn.datasets import make_blobs
from matplotlib import pyplot

data, label = make_blobs(n_samples=100, n_features=2, centers=5)
# 绘制样本显示
pyplot.scatter(data[:, 0], data[:, 1], c=label)
pyplot.show()
```

![](http://ww3.sinaimg.cn/mw690/9d2c4511gw1fbemfuid66j20m80gkjry.jpg)

- 如果希望为每个类别设置不同的方差，需要在上述代码中加入cluster_std参数：

```python
from sklearn.datasets import make_blobs
from matplotlib import pyplot

data, label = make_blobs(n_samples=10, n_features=2, centers=3, cluster_std=[0.8, 2.5, 4.5])
# 绘制样本显示
pyplot.scatter(data[:, 0], data[:, 1], c=label)
pyplot.show()
```

![](http://ww4.sinaimg.cn/mw690/9d2c4511gw1fbemfutayzj20m80gkjry.jpg)

### **降维--Dimensionality reduction**

### **模型选择--Model selection**

### **预处理--Preprocessing**

> 参考网址：[http://www.cnblogs.com/cherishZhang/p/4267113.html](http://www.cnblogs.com/cherishZhang/p/4267113.html)

## scikit_learn的数据集

> **from sklearn import datasets**

### iris数据集

> `Iris`数据集是常用的分类实验数据集，由`Fisher`, 1936收集整理。`Iris`也称鸢尾花卉数据集，是一类多重变量分析的数据集。数据集包含150个数据集，分为3类，每类50个数据，每个数据包含4个属性。可通过花萼长度，花萼宽度，花瓣长度，花瓣宽度4个属性预测鸢尾花卉属于（`Setosa`，`Versicolour`，`Virginica`）三个种类中的哪一类。

```python
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
 
df = pd.read_csv('http://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data', header=None) # 加载Iris数据集作为DataFrame对象
X = df.iloc[:, [0, 2]].values # 取出2个特征，并把它们用Numpy数组表示
 
plt.scatter(X[:50, 0], X[:50, 1],color='red', marker='o', label='setosa') # 前50个样本的散点图
plt.scatter(X[50:100, 0], X[50:100, 1],color='blue', marker='x', label='versicolor') # 中间50个样本的散点图
plt.scatter(X[100:, 0], X[100:, 1],color='green', marker='+', label='Virginica') # 后50个样本的散点图
plt.xlabel('petal length')
plt.ylabel('sepal length')
plt.legend(loc=2) # 把说明放在左上角，具体请参考官方文档
plt.show()
```



## scikit_learn的交叉验证模块`cross_validation`

> 常用的函数有`train_test_split()`将数据集分为训练集和测试集`k-fold`

> 参考文献：[http://blog.csdn.net/u010454729/article/details/50754076](http://blog.csdn.net/u010454729/article/details/50754076)


```python
from sklearn import datasets
import numpy as np
from sklearn.cross_validation import train_test_split

iris = datasets.load_iris()
X = iris.data[:, [2, 3]]
y = iris.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=0)
```

##  机器学习相关的库

> 
> 1. scikit_learn   机器学习方面的库
> 2. pandas         处理数据文本用
> 3. tensorflow     谷歌的深度学习，神经网络库
> 4. pybrain，keras        机器学习中神经网络的库
> 5. NLTK           自然语言处理
> 6. xgboost        预测模型


# 参考：
[1]：http://scikit-learn.org/
[2]：https://www.tensorflow.org/
[3]: http://pybrain.org/
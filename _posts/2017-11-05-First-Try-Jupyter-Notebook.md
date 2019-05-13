---
title: First-Try-Jupyter Notebook
date: 2017-11-05 19:00:00
tags: 数据分析
categories: python
toc: true
---

我好像突然发现了一个新大陆，各种炫酷、简洁、交互式的IDE，在体验的同时只能感叹科技如此发达。

<!--more-->


#### Notebook文件保存
1. 另存为pdf
报错如下，该问题暂时还没有解决。
```
nbconvert failed: xelatex not found on PATH, if you have not installed xelatex you may need to do so.
```

#### 运行一段示例代码

##### 良／恶性乳腺癌肿瘤预测完整代码Review

```python
import pandas as pd

#csv文件列：序号|Clump Thickness|Cell Size|Type
df_train = pd.read_csv('../Datasets/Breast-Cancer/breast-cancer-train.csv')
df_test = pd.read_csv('../Datasets/Breast-Cancer/breast-cancer-test.csv')

df_test_negative = df_test.loc[df_test['Type'] == 0][['Clump Thickness', 'Cell Size']]
df_test_positive = df_test.loc[df_test['Type'] == 1][['Clump Thickness', 'Cell Size']]
```


```python
import matplotlib.pyplot as plt

plt.scatter(df_test_negative['Clump Thickness'],df_test_negative['Cell Size'], marker = 'o', s=200, c='red')
plt.scatter(df_test_positive['Clump Thickness'],df_test_positive['Cell Size'], marker = 'x', s=150, c='black')

plt.xlabel('Clump Thickness')
plt.ylabel('Cell Size')

plt.show()
```


![png](/images/posts/2017.11.5/output_2_0.png)


```python
import numpy as np

intercept = np.random.random([1])
coef = np.random.random([2])
print("interdept: ",intercept)
print("coef:",coef)

lx = np.arange(0, 12)
ly = (-intercept - lx * coef[0]) / coef[1]
print("lx:",lx)
print("ly:",ly);
plt.plot(lx, ly, c='yellow')


plt.scatter(df_test_negative['Clump Thickness'],df_test_negative['Cell Size'], marker = 'o', s=200, c='red')
plt.scatter(df_test_positive['Clump Thickness'],df_test_positive['Cell Size'], marker = 'x', s=150, c='black')
plt.xlabel('Clump Thickness')
plt.ylabel('Cell Size')
plt.show()
```

    interdept:  [ 0.60880085]
    coef: [ 0.74079764  0.92597959]
    lx: [ 0  1  2  3  4  5  6  7  8  9 10 11]
    ly: [-0.65746681 -1.45748189 -2.25749697 -3.05751206 -3.85752714 -4.65754222
     -5.4575573  -6.25757238 -7.05758747 -7.85760255 -8.65761763 -9.45763271]



![png](/images/posts/2017.11.5/output_3_1.png)



```python
# 导入sklearn中的logistic回归分类器
from sklearn.linear_model import LogisticRegression
lr = LogisticRegression()

lr.fit(df_train[['Clump Thickness', 'Cell Size']][:10], df_train['Type'][:10])
print ('Testing accuracy (10 training samples):', lr.score(df_test[['Clump Thickness', 'Cell Size']], df_test['Type']))

```

    Testing accuracy (10 training samples): 0.868571428571



```python
intercept = lr.intercept_
coef = lr.coef_[0, :]

ly = (-intercept - lx * coef[0]) / coef[1]

plt.plot(lx, ly, c='green')
plt.scatter(df_test_negative['Clump Thickness'],df_test_negative['Cell Size'], marker = 'o', s=200, c='red')
plt.scatter(df_test_positive['Clump Thickness'],df_test_positive['Cell Size'], marker = 'x', s=150, c='black')
plt.xlabel('Clump Thickness')
plt.ylabel('Cell Size')
plt.show()
```


![png](/images/posts/2017.11.5/output_5_0.png)



```python
lr = LogisticRegression()

lr.fit(df_train[['Clump Thickness', 'Cell Size']], df_train['Type'])
print ('Testing accuracy (all training samples):', lr.score(df_test[['Clump Thickness', 'Cell Size']], df_test['Type']))

```

    Testing accuracy (all training samples): 0.937142857143



```python
intercept = lr.intercept_
coef = lr.coef_[0, :]
ly = (-intercept - lx * coef[0]) / coef[1]

plt.plot(lx, ly, c='blue')
plt.scatter(df_test_negative['Clump Thickness'],df_test_negative['Cell Size'], marker = 'o', s=200, c='red')
plt.scatter(df_test_positive['Clump Thickness'],df_test_positive['Cell Size'], marker = 'x', s=150, c='black')
plt.xlabel('Clump Thickness')
plt.ylabel('Cell Size')
plt.show()
```


![png](/images/posts/2017.11.5/output_7_0.png)

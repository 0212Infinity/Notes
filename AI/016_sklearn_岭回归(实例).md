```shell
Longley数据集来自J．W．Longley（1967）发表在JASA上的一篇论文，是强共线性的宏观经济数据,包含
GNP deflator(GNP平减指数)、GNP(国民生产总值)、Unemployed(失业率)、ArmedForces(武装力量)、Population(人口)、year(年份)，Emlpoyed(就业率)。
LongLey数据集因存在严重的多重共线性问题，在早期经常用来检验各种算法或计算机的计算精度
```



```python
import numpy as np
from numpy import genfromtxt
from sklearn import linear_model
import matplotlib.pyplot as plt

# 读入数据 genfromtxt,这种方式只能读取数字,非数字为nan
data = genfromtxt(r"longley.csv",delimiter=',')
print(data)
```

```python
# 切分数据
x_data = data[1:,2:]
y_data = data[1:,1]
print(x_data)
print(y_data)
```

```python
# 创建模型
# 生成50个值, linspace(0.001, 1) => 从0.001到1生成50(默认值)个值
alphas_to_test = np.linspace(0.001, 1)
# 创建模型，保存误差值	RidgeCV=>交叉验证
model = linear_model.RidgeCV(alphas=alphas_to_test, store_cv_values=True)
model.fit(x_data, y_data)

# 岭系数 alphas_to_test中最好的岭系数
print(model.alpha_)
# loss值
print(model.cv_values_.shape)
```

```python
# 画图
# 岭系数跟loss值的关系
plt.plot(alphas_to_test, model.cv_values_.mean(axis=0))
# 选取的岭系数值的位置
plt.plot(model.alpha_, min(model.cv_values_.mean(axis=0)),'ro')
plt.show()
```

```python
model.predict(x_data[2,np.newaxis])
```


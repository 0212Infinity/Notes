```python
from sklearn.linear_model import LinearRegression
import numpy as np
import matplotlib.pyplot as plt

# 载入数据
# data.csv 数据 => (100, 2)
data = np.genfromtxt("data.csv", delimiter=",")
x_data = data[:,0]
y_data = data[:,1]
plt.scatter(x_data,y_data)
plt.show()
# (100,)
print(x_data.shape)
```



```python
# np.newaxis 加上一个维度
x_data = data[:,0,np.newaxis]
y_data = data[:,1,np.newaxis]
# (100,1)
print(x_data.shape)
# 创建并拟合模型
model = LinearRegression()
model.fit(x_data, y_data)
```



```python
# 画图
plt.plot(x_data, y_data, 'b.')
plt.plot(x_data, model.predict(x_data), 'r')
plt.show()
```


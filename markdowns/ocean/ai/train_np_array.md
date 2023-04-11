# train NP Array data simple
### demo
```python
import numpy as np
from keras.models import Sequential
from keras.layers import Dense
dtype = [('col1', 'float64'), ('col2', 'float64'), ('col3', 'float64'), ('col4', 'float64'), ('col5', 'int64')]
data = np.loadtxt('detail.txt', delimiter=',')
# print(data)
data_flat = data.flatten()
data_y = [1]

# 创建并编译模型
model = Sequential()
model.add(Dense(units=158*5, activation='relu', input_dim=5))  # 添加输入层
model.add(Dense(units=1, activation='sigmoid'))  # 添加输出层
model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])  # 编译模型
# 训练模型
model.fit(data_flat, np.array(data_y), epochs=100, batch_size=1)  # 训练模型，设置迭代次数和批量大小
# max length = 158 * 5

```

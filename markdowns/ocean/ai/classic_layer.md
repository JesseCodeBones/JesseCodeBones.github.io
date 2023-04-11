from tensorflow.keras.models import Model
from tensorflow.keras.layers import Input, LSTM, GRU, Conv1D, Dense, concatenate

# 定义输入数据的形状
input_shape = (timesteps, input_dim)

# 定义输入层
input_layer = Input(shape=input_shape)

# 使用 LSTM 模型
lstm_layer = LSTM(units=64, activation='relu', return_sequences=True)(input_layer)

# 使用 GRU 模型
gru_layer = GRU(units=64, activation='relu', return_sequences=True)(input_layer)

# 使用 1D CNN 模型
cnn_layer = Conv1D(filters=32, kernel_size=3, activation='relu')(input_layer)

# 合并不同类型的层
merged_layer = concatenate([lstm_layer, gru_layer, cnn_layer], axis=1)

# 添加全连接层
dense_layer = Dense(units=64, activation='relu')(merged_layer)

# 输出层
output_layer = Dense(units=output_dim)(dense_layer)

# 构建混合模型
model = Model(inputs=input_layer, outputs=output_layer)

# 编译模型
model.compile(optimizer='adam', loss='mse')

# 训练模型
model.fit(X_train, y_train, epochs=epochs, batch_size=batch_size, validation_data=(X_val, y_val))

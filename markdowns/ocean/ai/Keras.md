# Keras Training

## fine tuning
对于数据集中的数据太少而无法从头开始训练完整模型的任务，通常会执行迁移学习。

在深度学习情境中，迁移学习最常见的形式是以下工作流：

从之前训练的模型中获取层。
冻结这些层，以避免在后续训练轮次中破坏它们包含的任何信息。
在已冻结层的顶部添加一些新的可训练层。这些层会学习将旧特征转换为对新数据集的预测。
在您的数据集上训练新层。
最后一个可选步骤是微调，包括解冻上面获得的整个模型（或模型的一部分），然后在新数据上以极低的学习率对该模型进行重新训练。以增量方式使预训练特征适应新数据，有可能实现有意义的改进。  


层和模型还具有布尔特性 trainable。此特性的值可以更改。将 layer.trainable 设置为 False 会将层的所有权重从可训练移至不可训练。这一过程称为“冻结”层：已冻结层的状态在训练期间不会更新（无论是使用 fit() 进行训练，还是使用依赖于 trainable_weights 来应用梯度更新的任何自定义循环进行训练时）。

```python
layer = keras.layers.Dense(3)
layer.build((None, 4))  # Create the weights
layer.trainable = False  # Freeze the layer

print("weights:", len(layer.weights))
print("trainable_weights:", len(layer.trainable_weights))
print("non_trainable_weights:", len(layer.non_trainable_weights))
```

具体流程：  
实例化一个基础模型并加载预训练权重。   
通过该模型运行新的数据集，并记录基础模型中一个（或多个）层的输出。这一过程称为特征提取。  
使用该输出作为新的较小模型的输入数据。  
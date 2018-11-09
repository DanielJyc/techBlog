---
title: TensorFlow快速入门示例
date: 2018-07-08 20:27:59
tags: TensorFlow, 机器学习, ML, Eager Execution
---

# 设置程序

## 安装最新版本的 TensorFlow


``` python
!pip install -q --upgrade tensorflow
```

## 配置导入和 Eager Execution


``` python
from __future__ import absolute_import, division, print_function

import os
import matplotlib.pyplot as plt

import tensorflow as tf
import tensorflow.contrib.eager as tfe

tf.enable_eager_execution()

print("TensorFlow version: {}".format(tf.VERSION))
print("Eager execution: {}".format(tf.executing_eagerly()))


```

    TensorFlow version: 1.9.0-rc2
    Eager execution: True


# 导入和解析训练数据集

## 下载数据集


```python
train_dataset_url = "http://download.tensorflow.org/data/iris_training.csv"

train_dataset_fp = tf.keras.utils.get_file(fname=os.path.basename(train_dataset_url),
                                           origin=train_dataset_url)

print("Local copy of the dataset file: {}".format(train_dataset_fp))

```

    Downloading data from http://download.tensorflow.org/data/iris_training.csv
    8192/2194 [================================================================================================================] - 0s 0us/step
    Local copy of the dataset file: /content/.keras/datasets/iris_training.csv


## 检查数据: 使用 head -n5 命令查看前 5 个条目：



```python
!head -n5 {train_dataset_fp}
```

    120,4,setosa,versicolor,virginica
    6.4,2.8,5.6,2.2,2
    5.0,2.3,3.3,1.0,1
    4.9,2.5,4.5,1.7,2
    4.9,3.1,1.5,0.1,0


# 解析数据集


```python
def parse_csv(line):
  example_defaults = [[0.], [0.], [0.], [0.], [0]]  # sets field types
  parsed_line = tf.decode_csv(line, example_defaults)
  # First 4 fields are features, combine into single tensor
  features = tf.reshape(parsed_line[:-1], shape=(4,))
  # Last field is the label
  label = tf.reshape(parsed_line[-1], shape=())
  return features, label
```

# 创建训练 tf.data.Dataset|


```python
train_dataset = tf.data.TextLineDataset(train_dataset_fp)
train_dataset = train_dataset.skip(1)             # skip the first header row
train_dataset = train_dataset.map(parse_csv)      # parse each row
train_dataset = train_dataset.shuffle(buffer_size=1000)  # randomize
train_dataset = train_dataset.batch(32)

# View a single example entry from a batch
features, label = iter(train_dataset).next()
print("example features:", features[0])
print("example label:", label[0])
```

    example features: tf.Tensor([5.8 2.7 4.1 1. ], shape=(4,), dtype=float32)
    example label: tf.Tensor(1, shape=(), dtype=int32)


# 选择模型类型: 使用 Keras 创建模型



```python
model = tf.keras.Sequential([
  tf.keras.layers.Dense(10, activation="relu", input_shape=(4,)),  # input shape required
  tf.keras.layers.Dense(10, activation="relu"),
  tf.keras.layers.Dense(3)
])
print( model)
```

    <tensorflow.python.keras.engine.sequential.Sequential object at 0x7fa0842648d0>


# 训练模型

## 定义损失和梯度函数


```python
def loss(model, x, y):
  y_ = model(x)
  return tf.losses.sparse_softmax_cross_entropy(labels=y, logits=y_)

def grad(model, inputs, targets):
  with tf.GradientTape() as tape:
    loss_value = loss(model, inputs, targets)
  return tape.gradient(loss_value, model.variables)

```

## 创建优化器

TensorFlow 拥有许多可用于训练的优化算法。此模型使用的是 tf.train.GradientDescentOptimizer，它可以实现随机梯度下降法 (SGD)。


```python
optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.01)
```

## 训练循环
1. 迭代每个周期。通过一次数据集即为一个周期。
2. 在一个周期中，遍历训练 Dataset 中的每个样本，并获取样本的特征 (x) 和标签 (y)。
3. 根据样本的特征进行预测，并比较预测结果和标签。衡量预测结果的不准确性，并使用所得的值计算模型的损失和梯度。
4. 使用 optimizer 更新模型的变量。
5. 跟踪一些统计信息以进行可视化。
6. 对每个周期重复执行以上步骤。

num_epochs 变量是遍历数据集集合的次数。与直觉恰恰相反的是，训练模型的时间越长，并不能保证模型就越好。

num_epochs 是一个可以调整的超参数。选择正确的次数通常需要一定的经验和实验基础。




```python
## Note: Rerunning this cell uses the same model variables

# keep results for plotting
train_loss_results = []
train_accuracy_results = []

num_epochs = 201

for epoch in range(num_epochs):
  epoch_loss_avg = tfe.metrics.Mean()
  epoch_accuracy = tfe.metrics.Accuracy()

  # Training loop - using batches of 32
  for x, y in train_dataset:
    # Optimize the model
    grads = grad(model, x, y)
    optimizer.apply_gradients(zip(grads, model.variables),
                              global_step=tf.train.get_or_create_global_step())

    # Track progress
    epoch_loss_avg(loss(model, x, y))  # add current batch loss
    # compare predicted label to actual label
    epoch_accuracy(tf.argmax(model(x), axis=1, output_type=tf.int32), y)

  # end epoch
  train_loss_results.append(epoch_loss_avg.result())
  train_accuracy_results.append(epoch_accuracy.result())

  if epoch % 50 == 0:
    print("Epoch {:03d}: Loss: {:.3f}, Accuracy: {:.3%}".format(epoch,
                                                                epoch_loss_avg.result(),
                                                                epoch_accuracy.result()))
```

    Epoch 000: Loss: 1.433, Accuracy: 34.167%
    Epoch 050: Loss: 0.636, Accuracy: 70.000%
    Epoch 100: Loss: 0.417, Accuracy: 71.667%
    Epoch 150: Loss: 0.331, Accuracy: 87.500%
    Epoch 200: Loss: 0.239, Accuracy: 96.667%


## 可视化损失函数随时间推移而变化的情况


```python
fig, axes = plt.subplots(2, sharex=True, figsize=(12, 8))
fig.suptitle('Training Metrics')

axes[0].set_ylabel("Loss", fontsize=14)
axes[0].plot(train_loss_results)

axes[1].set_ylabel("Accuracy", fontsize=14)
axes[1].set_xlabel("Epoch", fontsize=14)
axes[1].plot(train_accuracy_results)

plt.show()

```


![](/images/TensorFlow入门示例_22_0.png)


# 评估模型的效果

## 设置测试数据集


```python
test_url = "http://download.tensorflow.org/data/iris_test.csv"

test_fp = tf.keras.utils.get_file(fname=os.path.basename(test_url),
                                  origin=test_url)

test_dataset = tf.data.TextLineDataset(test_fp)
test_dataset = test_dataset.skip(1)             # skip header row
test_dataset = test_dataset.map(parse_csv)      # parse each row with the funcition created earlier
test_dataset = test_dataset.shuffle(1000)       # randomize
test_dataset = test_dataset.batch(32)           # use the same batch size as the training set

```

    Downloading data from http://download.tensorflow.org/data/iris_test.csv
    8192/573 [============================================================================================================================================================================================================================================================================================================================================================================================================================================] - 0s 0us/step


## 根据测试数据集评估模型


```python
test_accuracy = tfe.metrics.Accuracy()

for (x, y) in test_dataset:
  prediction = tf.argmax(model(x), axis=1, output_type=tf.int32)
  test_accuracy(prediction, y)

print("Test set accuracy: {:.3%}".format(test_accuracy.result()))

```

    Test set accuracy: 96.667%


# 使用经过训练的模型进行预测


```python
class_ids = ["Iris setosa", "Iris versicolor", "Iris virginica"]

predict_dataset = tf.convert_to_tensor([
    [5.1, 3.3, 1.7, 0.5,],
    [5.9, 3.0, 4.2, 1.5,],
    [6.9, 3.1, 5.4, 2.1]
])

predictions = model(predict_dataset)

for i, logits in enumerate(predictions):
  class_idx = tf.argmax(logits).numpy()
  name = class_ids[class_idx]
  print("Example {} prediction: {}".format(i, name))

```

    Example 0 prediction: Iris setosa
    Example 1 prediction: Iris versicolor
    Example 2 prediction: Iris virginica

# colab地址

https://colab.research.google.com/drive/1iGHulgf_ioKl_GP_7_v8yTwwyfzP6KTR
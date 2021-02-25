---
title: "理解LSTM层与GRU层"
date: 2021-01-22T17:27:21+08:00
draft: false
---
# 理解LSTM层与GRU层

> SimpleRNN的问题在于，在时刻t，理论上来说，它应该能够记住许多时间部之前见过的各种信息，但实际上它是不可能学到这种长期依赖的。其原因在于“梯度消失”问题，这一效应类似于在层数较多的非循环网络中观察到的效应，随着层数的增加，网络最终变得无法训练。

## 1. LSTM层

LSTM层是SimpleRNN的一种变体，它增加了一种携带信息跨越多个时间步的方法。假设有一条传送带，其运行方向平行于你所处理的序列。序列中的信息可以在任意位置跳上传送带，然后被传送到更晚的时间步，并在需要时原封不动地跳回来。这实际上就是LSTM的原理：它保存信息以便后面使用，从而防止较早期的信号在处理过程中逐渐消失。

### 1.1 Keras 中一个LSTM的例子


```python
# 准备数据
from keras.datasets import imdb
from keras.preprocessing import sequence

max_features = 10000
maxlen = 500 
batch_size = 32

print('Loading data...')
(input_train, y_train), (input_test, y_test) = imdb.load_data(num_words=max_features)
print(len(input_train), 'train_sequences')
print(len(input_test), 'test sequences')

print('Pad sequences (samples x time)')
input_train = sequence.pad_sequences(input_train, maxlen=maxlen)
input_test = sequence.pad_sequences(input_test, maxlen=maxlen)
print('input_train shape: ', input_train.shape)
print('input_test shape:', input_test.shape)
```

    Loading data...
    25000 train_sequences
    25000 test sequences
    Pad sequences (samples x time)
    input_train shape:  (25000, 500)
    input_test shape: (25000, 500)



```python
# 使用Keras中的LSTM层
from keras.layers import LSTM, Dense
from keras.models import Sequential
from keras.layers import Embedding, SimpleRNN

max_features = 10000

model = Sequential()
model.add(Embedding(max_features, 32))
model.add(LSTM(32))
model.add(Dense(1, activation='sigmoid'))
model.compile(optimizer='rmsprop', loss='binary_crossentropy', metrics=['acc'])
history = model.fit(input_train, y_train, epochs=10, batch_size=128, validation_split=0.2)
```

    /usr/lib/python3.7/site-packages/tensorflow/python/ops/gradients_util.py:93: UserWarning: Converting sparse IndexedSlices to a dense Tensor of unknown shape. This may consume a large amount of memory.
      "Converting sparse IndexedSlices to a dense Tensor of unknown shape. "


    Train on 20000 samples, validate on 5000 samples
    Epoch 1/10
    20000/20000 [==============================] - 487s 24ms/step - loss: 0.5123 - acc: 0.7542 - val_loss: 0.4508 - val_acc: 0.7902
    Epoch 2/10
    20000/20000 [==============================] - 489s 24ms/step - loss: 0.2947 - acc: 0.8866 - val_loss: 0.2914 - val_acc: 0.8816
    Epoch 3/10
    20000/20000 [==============================] - 482s 24ms/step - loss: 0.2368 - acc: 0.9094 - val_loss: 0.3113 - val_acc: 0.8738
    Epoch 4/10
    20000/20000 [==============================] - 530s 26ms/step - loss: 0.2033 - acc: 0.9258 - val_loss: 0.3399 - val_acc: 0.8534
    Epoch 5/10
    20000/20000 [==============================] - 480s 24ms/step - loss: 0.1735 - acc: 0.9384 - val_loss: 0.5543 - val_acc: 0.8110
    Epoch 6/10
    20000/20000 [==============================] - 465s 23ms/step - loss: 0.1608 - acc: 0.9438 - val_loss: 0.2991 - val_acc: 0.8796
    Epoch 7/10
    20000/20000 [==============================] - 454s 23ms/step - loss: 0.1406 - acc: 0.9494 - val_loss: 0.3217 - val_acc: 0.8870
    Epoch 8/10
    20000/20000 [==============================] - 448s 22ms/step - loss: 0.1361 - acc: 0.9531 - val_loss: 0.6040 - val_acc: 0.8540
    Epoch 9/10
    20000/20000 [==============================] - 449s 22ms/step - loss: 0.1270 - acc: 0.9553 - val_loss: 0.4197 - val_acc: 0.8782
    Epoch 10/10
    20000/20000 [==============================] - 448s 22ms/step - loss: 0.1165 - acc: 0.9596 - val_loss: 0.3421 - val_acc: 0.8856


---
author: "李昌"
title: "理解循环神经网络"
date: "2021-02-25"
tags: ["DL&ML"]
categories: ["DL&ML"]
ShowToc: true
TocOpen: true
---

# 理解循环神经网络

## 1. 简单的循环神经网络

RNN以渐进的方式处理信息，同时保存一个关于所处理内容的内部模型，这个模型是根据过去的信息构建的，并随着新信息的进入而不断更新。  
RNN处理序列的方式是：遍历所有序列元素，并保存一个状态（State），其中包含与已查看内容相关的信息。
  
RNN的伪代码：
```python
state_t = 0
for input_t in input_sequence:
    output_t = f(input_t, state_t)
    state_t = output_t
```
可以给出具体的函数f,从输入和状态到输出的变换，其参数包括两个矩阵（W和U）和一个偏置向量。它类似于前馈网络中密集连接层所做的变换。
```python
state_t = 0
for input_t in input_sequence:
    output_t = activation(dot(W, input_t) + dot(U, state_t) + b)
    state_t = output_t
```


```python
# 简单RNN的numpy实现
import numpy as np

timesteps = 100  # 输入序列的时间步数
input_features = 32   # 输入特征空间的维度
output_features = 64   # 输出特征空间的维度

inputs = np.random.random((timesteps, input_features))  # 输入数据：随机噪声，仅作为示例

state_t = np.zeros((output_features,))   # 初试状态：全零向量

# 创建随机的权重矩阵
W = np.random.random((output_features, input_features))   
U = np.random.random((output_features, output_features))
b = np.random.random((output_features, ))

successive_outputs = []
for input_t in inputs:    # input_t是形状为（input_features, )的向量
    output_t = np.tanh(np.dot(W, input_t) + np.dot(U, state_t) + b)   # 由输入和当前状态计算得到当前输出
    
    successive_outputs.append(output_t) # 将这个输出保存到一个列表中
    
    state_t = output_t  # 更新网络的状态，用于下一个时间步
    
# 最终输出是一个形状为（timesteps, output_features）的二维张量
final_output_sequence = np.stack(successive_outputs, axis=0)
final_output_sequence
```




    array([[0.99999999, 0.99999995, 0.99999999, ..., 0.99999999, 0.9999997 ,
            0.99999992],
           [1.        , 1.        , 1.        , ..., 1.        , 1.        ,
            1.        ],
           [1.        , 1.        , 1.        , ..., 1.        , 1.        ,
            1.        ],
           ...,
           [1.        , 1.        , 1.        , ..., 1.        , 1.        ,
            1.        ],
           [1.        , 1.        , 1.        , ..., 1.        , 1.        ,
            1.        ],
           [1.        , 1.        , 1.        , ..., 1.        , 1.        ,
            1.        ]])



## 2. Keras 中的循环层

上面numpy的简单实现，对应一个实际的Keras层，即SimpleRNN层。
```python
from keras.layers import SimpleRNN
```

与Keras中所有的循环层一样，SimpleRNN可以在两种不同的模式下运行：一种是返回每个时间步连续输出的完整序列，即形状为（batch_size, timesteps, output_features）的三维张量；另一种是只返回每个输入系列的最终输出，即形状为(batch_size, output_featrues)的二维张量。这个两种模式由return_sequences这个构造函数参数来控制。


```python
from keras.models import Sequential
from keras.layers import Embedding, SimpleRNN

model = Sequential()
model.add(Embedding(10000, 32))
model.add(SimpleRNN(32))  #` 只返回最后一个时间步的输出
model.summary()
```

    Model: "sequential_2"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    embedding_2 (Embedding)      (None, None, 32)          320000    
    _________________________________________________________________
    simple_rnn_2 (SimpleRNN)     (None, 32)                2080      
    =================================================================
    Total params: 322,080
    Trainable params: 322,080
    Non-trainable params: 0
    _________________________________________________________________
    


```python
from keras.models import Sequential
from keras.layers import Embedding, SimpleRNN

model = Sequential()
model.add(Embedding(10000, 32))
model.add(SimpleRNN(32, return_sequences=True)) # 返回完整的状态序列
model.summary()
```

    Model: "sequential_3"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    embedding_3 (Embedding)      (None, None, 32)          320000    
    _________________________________________________________________
    simple_rnn_3 (SimpleRNN)     (None, 32)                2080      
    =================================================================
    Total params: 322,080
    Trainable params: 322,080
    Non-trainable params: 0
    _________________________________________________________________
    

为了提高网络的表示能力，将多个循环层逐个堆叠有时也是很有用的。在这种情况下，你需要让所有中间层都返回完整的输出序列。


```python
model = Sequential()
model.add(Embedding(10000, 32))
model.add(SimpleRNN(32, return_sequences=True))
model.add(SimpleRNN(32, return_sequences=True))
model.add(SimpleRNN(32, return_sequences=True))
model.add(SimpleRNN(32))
model.summary()
```

    Model: "sequential_4"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    embedding_4 (Embedding)      (None, None, 32)          320000    
    _________________________________________________________________
    simple_rnn_4 (SimpleRNN)     (None, None, 32)          2080      
    _________________________________________________________________
    simple_rnn_5 (SimpleRNN)     (None, None, 32)          2080      
    _________________________________________________________________
    simple_rnn_6 (SimpleRNN)     (None, None, 32)          2080      
    _________________________________________________________________
    simple_rnn_7 (SimpleRNN)     (None, 32)                2080      
    =================================================================
    Total params: 328,320
    Trainable params: 328,320
    Non-trainable params: 0
    _________________________________________________________________
    

## 3. 将RNN应用于IMDB数据集

### 3.1 准备数据


```python
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
# 用Embedding层和SimpleRNN层来训练模型
from keras.layers import Dense

model = Sequential()
model.add(Embedding(max_features, 32))
model.add(SimpleRNN(32))
model.add(Dense(1, activation='sigmoid'))

model.compile(optimizer='rmsprop', loss='binary_crossentropy', metrics=['acc'])
history = model.fit(input_train, y_train, epochs=10, batch_size=10, validation_split=0.2)
```

    /usr/lib/python3.7/site-packages/tensorflow/python/ops/gradients_util.py:93: UserWarning: Converting sparse IndexedSlices to a dense Tensor of unknown shape. This may consume a large amount of memory.
      "Converting sparse IndexedSlices to a dense Tensor of unknown shape. "
    

    Train on 20000 samples, validate on 5000 samples
    Epoch 1/10
    20000/20000 [==============================] - 4783s 239ms/step - loss: 0.5510 - acc: 0.7012 - val_loss: 0.4217 - val_acc: 0.8162
    Epoch 2/10
     1080/20000 [>.............................] - ETA: 1:14:34 - loss: 0.3898 - acc: 0.8250


    ---------------------------------------------------------------------------

    KeyboardInterrupt                         Traceback (most recent call last)

    <ipython-input-10-a41d37bef560> in <module>
          8 
          9 model.compile(optimizer='rmsprop', loss='binary_crossentropy', metrics=['acc'])
    ---> 10 history = model.fit(input_train, y_train, epochs=10, batch_size=10, validation_split=0.2)
    

    /usr/lib/python3.7/site-packages/keras/engine/training.py in fit(self, x, y, batch_size, epochs, verbose, callbacks, validation_split, validation_data, shuffle, class_weight, sample_weight, initial_epoch, steps_per_epoch, validation_steps, validation_freq, max_queue_size, workers, use_multiprocessing, **kwargs)
       1237                                         steps_per_epoch=steps_per_epoch,
       1238                                         validation_steps=validation_steps,
    -> 1239                                         validation_freq=validation_freq)
       1240 
       1241     def evaluate(self,
    

    /usr/lib/python3.7/site-packages/keras/engine/training_arrays.py in fit_loop(model, fit_function, fit_inputs, out_labels, batch_size, epochs, verbose, callbacks, val_function, val_inputs, shuffle, initial_epoch, steps_per_epoch, validation_steps, validation_freq)
        194                     ins_batch[i] = ins_batch[i].toarray()
        195 
    --> 196                 outs = fit_function(ins_batch)
        197                 outs = to_list(outs)
        198                 for l, o in zip(out_labels, outs):
    

    /usr/lib/python3.7/site-packages/tensorflow/python/keras/backend.py in __call__(self, inputs)
       3215         value = math_ops.cast(value, tensor.dtype)
       3216       converted_inputs.append(value)
    -> 3217     outputs = self._graph_fn(*converted_inputs)
       3218     return nest.pack_sequence_as(self._outputs_structure,
       3219                                  [x.numpy() for x in outputs])
    

    /usr/lib/python3.7/site-packages/tensorflow/python/eager/function.py in __call__(self, *args, **kwargs)
        556       raise TypeError("Keyword arguments {} unknown. Expected {}.".format(
        557           list(kwargs.keys()), list(self._arg_keywords)))
    --> 558     return self._call_flat(args)
        559 
        560   def _filtered_call(self, args, kwargs):
    

    /usr/lib/python3.7/site-packages/tensorflow/python/eager/function.py in _call_flat(self, args)
        625     # Only need to override the gradient in graph mode and when we have outputs.
        626     if context.executing_eagerly() or not self.outputs:
    --> 627       outputs = self._inference_function.call(ctx, args)
        628     else:
        629       self._register_gradient()
    

    /usr/lib/python3.7/site-packages/tensorflow/python/eager/function.py in call(self, ctx, args)
        413             attrs=("executor_type", executor_type,
        414                    "config_proto", config),
    --> 415             ctx=ctx)
        416       # Replace empty list with None
        417       outputs = outputs or None
    

    /usr/lib/python3.7/site-packages/tensorflow/python/eager/execute.py in quick_execute(op_name, num_outputs, inputs, attrs, ctx, name)
         58     tensors = pywrap_tensorflow.TFE_Py_Execute(ctx._handle, device_name,
         59                                                op_name, inputs, attrs,
    ---> 60                                                num_outputs)
         61   except core._NotOkStatusException as e:
         62     if name is not None:
    

    KeyboardInterrupt: 



```python
import matplotlib.pyplot as plt

acc = history.history['acc']
val_acc = history.history['val_acc']
loss = history.history['loss']
val_loss = history.history['val_loss']

epochs = range(1, len(acc)+1)

plt.plot(epochs, acc, 'bo', label='Training acc')
plt.plot(epochs, val, 'b', label='Validation acc')
plt.title("Trainging and validation accuracy")
plt.legend()

plt.figure()

plt.plot(epochs, loss, 'bo', label='Training loss')
plt.plot(epochs, val_loss, 'b', label='Validation loss')
plt.title('Training and validation loss')
plt.legend()

plt.show()
```

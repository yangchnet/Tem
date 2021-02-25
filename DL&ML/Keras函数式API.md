---
author: "李昌"
title: "Keras函数式API"
date: "2021-02-25"
tags: ["DL&ML"]
categories: ["DL&ML"]
ShowToc: true
TocOpen: true
---

# Keras函数式API

使用函数式API，你可以直接操作张量，也可以把层当做函数来使用，接收张量并返回张量。


```python
# 简单的实例
import os
# **** change the warning level ****
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '3'

from keras.models import Sequential, Model
from keras import layers
from keras import Input

# 使用Sequential模型
seq_model = Sequential()
seq_model.add(layers.Dense(32, activation='relu', input_shape=(64,)))
seq_model.add(layers.Dense(32, activation='relu'))
seq_model.add(layers.Dense(10, activation='softmax'))

# 对应的函数式API实现
input_tensor = Input(shape=(64,))
x = layers.Dense(32, activation='relu')(input_tensor)
x = layers.Dense(32, activation='softmax')(x)
output_tensor = layers.Dense(10, activation='softmax')(x)

model = Model(input_tensor, output_tensor)
model.summary()
```

    Model: "model_2"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    input_2 (InputLayer)         (None, 64)                0         
    _________________________________________________________________
    dense_10 (Dense)             (None, 32)                2080      
    _________________________________________________________________
    dense_11 (Dense)             (None, 32)                1056      
    _________________________________________________________________
    dense_12 (Dense)             (None, 10)                330       
    =================================================================
    Total params: 3,466
    Trainable params: 3,466
    Non-trainable params: 0
    _________________________________________________________________
    

Keras会在后台检索从input_tensor到output_tensor所包含的每一层，并将这些层组合成一个类图的数据结构，即一个Model。这种方法有效的原因在于，output_tensor是通过对input_tensor进行多次变换得到的。如果你试图利用不相关的输入和输出来构建一个模型，那么会得到RuntimeError


```python
unrelated_input = Input(shape=(32,))
bad_model = model = Model(unrelated_input, output_tensor)
```


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    <ipython-input-3-54197a8d0ec3> in <module>
          1 unrelated_input = Input(shape=(32,))
    ----> 2 bad_model = model = Model(unrelated_input, output_tensor)
    

    /usr/lib/python3.7/site-packages/keras/legacy/interfaces.py in wrapper(*args, **kwargs)
         89                 warnings.warn('Update your `' + object_name + '` call to the ' +
         90                               'Keras 2 API: ' + signature, stacklevel=2)
    ---> 91             return func(*args, **kwargs)
         92         wrapper._original_function = func
         93         return wrapper
    

    /usr/lib/python3.7/site-packages/keras/engine/network.py in __init__(self, *args, **kwargs)
         92                 'inputs' in kwargs and 'outputs' in kwargs):
         93             # Graph network
    ---> 94             self._init_graph_network(*args, **kwargs)
         95         else:
         96             # Subclassed network
    

    /usr/lib/python3.7/site-packages/keras/engine/network.py in _init_graph_network(self, inputs, outputs, name, **kwargs)
        239         # Keep track of the network's nodes and layers.
        240         nodes, nodes_by_depth, layers, layers_by_depth = _map_graph_network(
    --> 241             self.inputs, self.outputs)
        242         self._network_nodes = nodes
        243         self._nodes_by_depth = nodes_by_depth
    

    /usr/lib/python3.7/site-packages/keras/engine/network.py in _map_graph_network(inputs, outputs)
       1509                                          'The following previous layers '
       1510                                          'were accessed without issue: ' +
    -> 1511                                          str(layers_with_complete_input))
       1512                 for x in node.output_tensors:
       1513                     computable_tensors.append(x)
    

    ValueError: Graph disconnected: cannot obtain value for tensor Tensor("input_2:0", shape=(None, 64), dtype=float32) at layer "input_2". The following previous layers were accessed without issue: []


对这种Model实例进行编译、训练或评估时，其API与Sequential模型相同


```python
model.compile(optimizer='rmsprop', loss='categorical_crossentropy')

import numpy as np
x_train = np.random.random((1000, 64))
y_train = np.random.random((1000, 10))

model.fit(x_train, y_train, epochs=10, batch_size=128)

score = model.evaluate(x_train, y_train)
```

    Epoch 1/10
    1000/1000 [==============================] - 0s 83us/step - loss: 11.6211
    Epoch 2/10
    1000/1000 [==============================] - 0s 33us/step - loss: 11.6215
    Epoch 3/10
    1000/1000 [==============================] - 0s 32us/step - loss: 11.6216
    Epoch 4/10
    1000/1000 [==============================] - 0s 27us/step - loss: 11.6216
    Epoch 5/10
    1000/1000 [==============================] - 0s 36us/step - loss: 11.6216
    Epoch 6/10
    1000/1000 [==============================] - 0s 37us/step - loss: 11.6217
    Epoch 7/10
    1000/1000 [==============================] - 0s 37us/step - loss: 11.6217
    Epoch 8/10
    1000/1000 [==============================] - 0s 25us/step - loss: 11.6217
    Epoch 9/10
    1000/1000 [==============================] - 0s 40us/step - loss: 11.6217
    Epoch 10/10
    1000/1000 [==============================] - 0s 34us/step - loss: 11.6216
    1000/1000 [==============================] - 0s 69us/step
    

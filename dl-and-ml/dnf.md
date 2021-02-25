---
title: 基于卷积神经网络和决策树的体域网数据融合方法
date: '2021-01-22T09:27:21.000Z'
draft: false
---

# 基于卷积神经网络和决策树的体域网数据融合方法

现阶段想法:在softmax层后接随机森林，通过种树增加分类准确率

```python
import tensorflow as tf
import numpy as np
import tensorflow.examples.tutorials.mnist.input_data as input_data
import scipy as sp
%matplotlib inline
sess = tf.Session()
```

```python
DEPTH = 3  # Depth of a tree
N_LEAF = 2 ** (DEPTH + 1)  # Number of leaf node
N_LABEL = 10  # Number of classes
N_TREE = 5  # Number of trees (ensemble)
N_BATCH = 128  # Number of data points per mini-batch 分批训练，每一批128个
```

## 初始化矩阵

```python
def init_weights(shape):
    return tf.Variable(tf.random_normal(shape, stddev=0.01))
```

```python
def init_prob_weights(shape, minval=-5, maxval=5):
    return tf.Variable(tf.random_uniform(shape, minval, maxval))
```

## 定义模型

* a表示alive,激活之意，eg:l1a,表示layer\_1\_alive，第一个激活层
* w表示weight,权重

```python
def model(X, w, w2, w3, w4_e, w_d_e, w_l_e, p_keep_conv, p_keep_hidden):
    # 激活层1 & 池化层1 & dropout
    l1a = tf.nn.relu(tf.nn.conv2d(X, w, [1, 1, 1, 1], 'SAME'))
    l1 = tf.nn.max_pool(l1a, ksize=[1, 2, 2, 1], strides=[1, 2, 2, 1], 
                        padding='SAME')
    l1 = tf.nn.dropout(l1, p_keep_conv)

    # 激活层2 & 池化层 & dropout
    l2a = tf.nn.relu(tf.nn.conv2d(l1, w2, [1, 1, 1, 1], 'SAME'))
    l2 = tf.nn.max_pool(l2a, ksize=[1, 2, 2, 1],
                       strides=[1, 2, 2, 1], padding='SAME')
    l2 = tf.nn.dropout(l2, p_keep_conv)

    # 激活层3 & 池化层 & full connected layer & dropout
    l3a = tf.nn.relu(tf.nn.conv2d(l2, w3, [1, 1, 1, 1], 'SAME'))
    l3 = tf.nn.max_pool(l3a, ksize=[1, 2, 2, 1],
                       strides=[1, 2, 2, 1], padding='SAME')
    l3 = tf.reshape(l3, [-1, w4_e[0].get_shape().as_list()[0]])
    l3 = tf.nn.dropout(l3, p_keep_conv)

    # decision node & prediction node (leaf node)
    decision_p_e = []
    leaf_p_e = []
    for w4, w_d, w_l in zip(w4_e, w_d_e, w_l_e):
        l4 = tf.nn.relu(tf.matmul(l3, w4))
        l4 = tf.nn.dropout(l4, p_keep_conv)

        decision_p = tf.nn.sigmoid(tf.matmul(l4, w_d))
        # 从这一句看，好像叶子节点不与决策节点相关
        leaf_p = tf.nn.softmax(w_l)

        decision_p_e.append(decision_p)
        leaf_p_e.append(leaf_p)

    return decision_p_e, leaf_p_e
```

## 创建占位符作为输入

```python
X = tf.placeholder("float", [N_BATCH, 28, 28, 1]) 
Y = tf.placeholder("float", [N_BATCH, N_LABEL])
```

## 初始化参数

```python
w = init_weights([3, 3, 1, 32])
w2 = init_weights([3, 3, 32, 64])
w3 = init_weights([3, 3, 64, 128])

w4_ensemble = []
w_d_ensemble = []
w_l_ensemble = []
for i in range(N_TREE):
    w4_ensemble.append(init_weights([128*4*4, 625]))
    w_d_ensemble.append(init_prob_weights([625, N_LEAF], -1, 1))
    w_l_ensemble.append(init_prob_weights([N_LEAF, N_LABEL], -2, 2))

p_keep_conv = tf.placeholder("float")
p_keep_hidden = tf.placeholder("float")
```

## 定义一个完全可微deep-ndf

```python
decision_p_e, leaf_p_e = model(X, w, w2, w3, w4_ensemble, w_d_ensemble,
                              w_l_ensemble, p_keep_conv, p_keep_hidden)
flat_decision_p_e = []

for decision_p in decision_p_e:
    # decision_p是d, decision_p_comp是1-d
    decision_p_comp = tf.subtract(tf.ones_like(decision_p), decision_p)

    decision_p_pack = tf.stack([decision_p, decision_p_comp])

    flat_decision_p = tf.reshape(decision_p_pack, [-1])
    flat_decision_p_e.append(flat_decision_p)
```

```python
batch_0_indices = \
        tf.tile(tf.expand_dims(tf.range(0, N_BATCH * N_LEAF, N_LEAF), 
                               1), [1, N_LEAF])
```

```python
sess = tf.Session()
sess.run(batch_0_indices)
```

```text
array([[   0,    0,    0, ...,    0,    0,    0],
       [  16,   16,   16, ...,   16,   16,   16],
       [  32,   32,   32, ...,   32,   32,   32],
       ...,
       [2000, 2000, 2000, ..., 2000, 2000, 2000],
       [2016, 2016, 2016, ..., 2016, 2016, 2016],
       [2032, 2032, 2032, ..., 2032, 2032, 2032]], dtype=int32)
```

batch\_0\_indices.shape = 128 \* 16

```python
in_repeat = N_LEAF / 2
out_repeat = N_BATCH
```

```python
batch_complement_indices = \
    np.array([[0] * int(in_repeat), [N_BATCH * N_LEAF] \
             * int(in_repeat)] * out_repeat).reshape(N_BATCH, N_LEAF)

print(batch_complement_indices)
```

```text
[[   0    0    0 ... 2048 2048 2048]
 [   0    0    0 ... 2048 2048 2048]
 [   0    0    0 ... 2048 2048 2048]
 ...
 [   0    0    0 ... 2048 2048 2048]
 [   0    0    0 ... 2048 2048 2048]
 [   0    0    0 ... 2048 2048 2048]]
```

```python
sess.run(tf.add(batch_0_indices, batch_complement_indices))
```

```text
array([[   0,    0,    0, ..., 2048, 2048, 2048],
       [  16,   16,   16, ..., 2064, 2064, 2064],
       [  32,   32,   32, ..., 2080, 2080, 2080],
       ...,
       [2000, 2000, 2000, ..., 4048, 4048, 4048],
       [2016, 2016, 2016, ..., 4064, 4064, 4064],
       [2032, 2032, 2032, ..., 4080, 4080, 4080]], dtype=int32)
```


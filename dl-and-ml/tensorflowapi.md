---
title: ''
date: '2021-01-22T09:27:21.000Z'
draft: false
---

# tensorflowAPI

```python
import tensorflow as tf
import numpy as np
from IPython.display import Image
```

## tf.ones\_like\(\)

创建一个所有元素设置为1的tensor

## tf.subtract\(\)

两个矩阵相减

> decision\_p\_comp = tf.subtract\(tf.ones\_like\(decision\_p\), decision\_p\)
>
> 这一句计算出1-d

## tf.stack

矩阵拼接，例如

```python
a = tf.constant([1,2,3])
b = tf.constant([4,5,6])
c = tf.stack([a, b], axis = 0)
d = tf.stack([a, b], axis = 1)
sess = tf.Session()
print(sess.run(c))
print(sess.run(d))
```

```text
[[1 2 3]
 [4 5 6]]
[[1 4]
 [2 5]
 [3 6]]
```

## tf.expand\_dims

在axis位置增加一个维度

## tf.tile

在同一维度上进行复制

```python
with tf.Graph().as_default():
    a = tf.constant([1,2],name='a') 
    b = tf.tile(a,[3])
    sess = tf.Session()
    print(sess.run(b))
```

```text
[1 2 1 2 1 2]
```

axis 0 代表行 1 代表列

## tf.reduce\_mean

计算张量的各个维度上的元素的平均值。

```python
x = tf.constant([[1., 1.], [2., 2.]])
tf.reduce_mean(x)  # 1.5
tf.reduce_mean(x, 0)  # [1.5, 1.5]
tf.reduce_mean(x, 1)  # [1.,  2.]
```

## tf.nn.dropout

dropout的作用是为了防止或减轻过拟合，它一般用在全连接层 通过在不同的训练过程中随机扔掉一部分神经元，也就是让某个神经元的激活值以一定的概率p, 让其停止工作，这次训练过程中不更新权值，也不参加神经网络的计算，但是他的权重得保存下来，待下次样本输入时可能会重新工作。 其函数原型： tf.nn.dropout\(x, keep\_prob, noise\_shape=None, seed=None,name=None\) 第一个参数为输入，第二个参数设置神经元被选中的概率，

## tf.gather

根据索引，从输入张量中依次取元素，构成一个新的张量


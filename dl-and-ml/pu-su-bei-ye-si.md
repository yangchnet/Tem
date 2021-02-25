---
title: 朴素贝叶斯
date: '2021-01-22T09:27:21.000Z'
draft: false
---

# 朴素贝叶斯

## 1、理论部分

### 1.1、贝叶斯公式

$$P(c|x)=\frac{P(c)P(x|c)}{P(x)}\qquad\dots(1)$$  
其中，$P\(c\)$是类“先验概率”；$P\(x\|c\)$是样本$x$相对于类标记$c$的类条件概率，或称为“似然”；$P\(x\)$是用于归一化的“证据因子”。对给定样本$x$，证据因子$P\(x\)$与类标记无关，因此估计$P\(c\|x\)$的问题就转化为如何基于训练数据$D$来估计先验$P\(c\)$和似然$P\(x\|c\)$  
类先验概率$P\(c\)$表达了样本空间中各类样本所占的比例，根据大数定律，当训练集包含充足的独立同分布样本时，$P\(c\)$可通过各类样本出现的频率来进行估计。  
对类条件概率$\(P\(x\|c\)\)$来说，由于它涉及关于$x$所有属性的联合概率，直接根据样本出现的频率来估计将会遇到严重的困难。为避开这个障碍，朴素贝叶斯分类器采用了“属性条件独立性假设”；对已知类别，假设所有属性相互独立。换言之，假设每个属性独立的对分类结果产生影响。  
基于属性条件独立性假设，贝叶斯公式可重写为： $$P(c|x)=\frac{P(c)P(x|c)}{P(x)}\qquad=\frac{P(c)}{P(x)}\prod_{i=1}^d{P(x_i|c)}\dots(2)$$ 其中$d$为属性数目，$x_i$为$x$在第i个属性上的取值  
由于对于所有类别来说$P\(x\)$相同，因此贝叶斯判定准则：$$h_{nb}\(x\)=arg max_{c\in y}P\(c\)\prod_{i=1}^d{P\(x_i\|c\)}\dots\(3\)$$  
显然，朴素贝叶斯分类器的训练过程就是基于训练集$D$来估计类先验概率$P\(c\)$，并为每个属性估计条件概率$P\(x\_i\|c\)$  
令$D\_c$表示训练集$D$中第$c$类样本组成的集合，若有充足的独立同分布样本，则可容易的估计出先验概率：_$$P(c)=\frac{|D_c|}{|D|}\dots(4)$$  
_对离散属性而言，令$D_{c,x_i}$表示$D\_c$中在第$i$个属性上取值为$x\_i$的样本组成的集合，则条件概率$P\(x\_i\|c\)$可估计为$$P\(x\_i\|c\)=\frac{\|D_{c,x_i}\|}{\|D\_c\|}\qquad\dots\(5\)$$ 为了避免其他属性携带的信息被训练集中未出现的属性值抹去，在估计概率值时通常要进行“平滑”，常用“拉普拉斯修正”。具体来说，令$N$表示训练集$D$中可能的类别数，$N\_i$表示第$i$个属性可能的取值数，则\(4\)\(5\)两式分别修正为：_$$\hat{P}(c)=\frac{D_c+1}{|D|+N}\qquad\dots(6)$$ _$$\hat{P}\(x\_i\|c\)=\frac{D_{c,x\_i}+1}{\|D\|+N}\qquad\dots\(7\)$$

## 2、实战演练

### 2.1、加载数据集

```python
import numpy as np
def loadDataSet():
    """
    导入数据， 1代表脏话
    @ return postingList: 数据集
    @ return classVec: 分类向量
    """
    postingList = [['my', 'dog', 'has', 'flea', 'problems', 'help', 'please'],
                   ['maybe', 'not', 'take', 'him', 'to', 'dog', 'park', 'stupid'],
                   ['my', 'dalmation', 'is', 'so', 'cute', 'I', 'love', 'him'],
                   ['stop', 'posting', 'stupid', 'worthless', 'garbage'],
                   ['mr', 'licks', 'ate', 'my', 'steak', 'how', 'to', 'stop', 'him'],
                   ['quit', 'buying', 'worthless', 'dog', 'food', 'stupid']]
    classVec = [0, 1, 0, 1, 0, 1]
    return postingList, classVec
```

导入训练集及其分类，1代表是脏话，0代表不是

```python
loadDataSet()
```

```text
([['my', 'dog', 'has', 'flea', 'problems', 'help', 'please'],
  ['maybe', 'not', 'take', 'him', 'to', 'dog', 'park', 'stupid'],
  ['my', 'dalmation', 'is', 'so', 'cute', 'I', 'love', 'him'],
  ['stop', 'posting', 'stupid', 'worthless', 'garbage'],
  ['mr', 'licks', 'ate', 'my', 'steak', 'how', 'to', 'stop', 'him'],
  ['quit', 'buying', 'worthless', 'dog', 'food', 'stupid']],
 [0, 1, 0, 1, 0, 1])
```

### 2.2、创建单词表

将训练数据中的每一个词都存储到词库中。

```python
def createVocabList(dataSet):
    """
    创建词库
    @ param dataSet: 数据集
    @ return vocabSet: 词库
    """
    vocabSet = set([])
    for document in dataSet:
        # 求并集
        vocabSet = vocabSet | set(document)
    return list(vocabSet)
```

```python
listOPosts, listClasses = loadDataSet()
myVocabList = createVocabList(listOPosts)
myVocabList
```

```text
['quit',
 'how',
 'problems',
 'mr',
 'dalmation',
 'garbage',
 'love',
 'stop',
 'maybe',
 'him',
 'take',
 'to',
 'stupid',
 'food',
 'not',
 'cute',
 'buying',
 'flea',
 'park',
 'help',
 'ate',
 'dog',
 'licks',
 'please',
 'so',
 'has',
 'my',
 'is',
 'posting',
 'I',
 'steak',
 'worthless']
```

### 2.3、生成词向量

```python
def setOfWords2Vec(vocabList, inputSet):
    """
    文本词向量.词库中每个词当作一个特征，文本中有该词，该词特征就是1，没有就是0
    @ param vocabList: 词表
    @ param inputSet: 输入的数据集
    @ return returnVec: 返回的向量
    """
    returnVec = [0] * len(vocabList)
    for word in inputSet:
        if word in vocabList:
            returnVec[vocabList.index(word)] = 1
        else:
            print("单词: %s 不在词库中!" % word)
    return returnVec
```

```python
testEntry = ['love', 'my', 'dalmation']
thisDoc = np.array(setOfWords2Vec(myVocabList, testEntry))
thisDoc
```

```text
array([0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
       0, 0, 0, 0, 1, 0, 0, 0, 0, 0])
```

### 2.4、训练分类器

```python
def trainNB0(trainMatrix, trainCategory):
    """
    训练
    @ param trainMatrix: 训练集
    @ param trainCategory: 分类
    """
    numTrainDocs = len(trainMatrix)   # 训练数据的长度
    numWords = len(trainMatrix[0])    # 训练数据的词汇量
    pAbusive = sum(trainCategory) / float(numTrainDocs)
    # 防止某个类别计算出的概率为0，导致最后相乘都为0，所以初始词都赋值1，分母赋值为2.  拉普拉斯修正
    p0Num = np.ones(numWords)   # 分子
    p1Num = np.ones(numWords)
    p0Denom = 2   # 分母
    p1Denom = 2
    for i in range(numTrainDocs):
        if trainCategory[i] == 1:
            p1Num += trainMatrix[i]
            p1Denom += sum(trainMatrix[i])
        else:
            p0Num += trainMatrix[i]
            p0Denom += sum(trainMatrix[i])
    # 这里使用log函数，方便计算，因为最后是比较大小，所有对结果没有影响。
    p1Vect = np.log(p1Num / p1Denom)   # P^(x_1|c)
    p0Vect = np.log(p0Num / p0Denom)   # P^(x_2|c)
    return p0Vect, p1Vect, pAbusive
```

### 2.5、 进行分类

```python
def classifyNB(vec2Classify, p0Vec, p1Vec, pClass1):
    """
    判断大小
    """
    p1 = sum(vec2Classify * p1Vec)  # P(c)*P(x_1|c)
    p0 = sum(vec2Classify * p0Vec)  # P(c)*P(x_2|c)
    if p1 > p0:
        return 1
    else:
        return 0
```

### 2.6、测试

```python
def testingNB():
    listOPosts, listClasses = loadDataSet()
    myVocabList = createVocabList(listOPosts)
    trainMat = []
    for postinDoc in listOPosts:
        trainMat.append(setOfWords2Vec(myVocabList, postinDoc))
    print(trainMat)   # 查看训练集矩阵
    p0V, p1V, pAb = trainNB0(np.array(trainMat), np.array(listClasses))
    testEntry = ['love', 'my', 'dalmation']
    thisDoc = np.array(setOfWords2Vec(myVocabList, testEntry))
    print(testEntry, 'classified as: ', classifyNB(thisDoc, p0V, p1V, pAb))
    testEntry = ['stupid', 'garbage']
    thisDoc = np.array(setOfWords2Vec(myVocabList, testEntry))
    print(testEntry, 'classified as: ', classifyNB(thisDoc, p0V, p1V, pAb))
```

```python
testingNB()
```

```text
[[0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 1, 0, 0, 0, 0, 0], [0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 1, 0, 0, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0], [0, 0, 0, 0, 1, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 1, 0, 1, 0, 0], [0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 1], [0, 1, 0, 1, 0, 0, 0, 1, 0, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0], [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1]]
['love', 'my', 'dalmation'] classified as:  0
['stupid', 'garbage'] classified as:  1
```


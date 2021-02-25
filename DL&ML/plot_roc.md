---
author: "李昌"
title: "Importsomedatatoplaywith"
date: "2021-02-25"
tags: ["DL&ML"]
categories: ["DL&ML"]
ShowToc: true
TocOpen: true
---


```python
%matplotlib inline
```


=======================================
Receiver Operating Characteristic (ROC)
=======================================

Example of Receiver Operating Characteristic (ROC) metric to evaluate
classifier output quality.

ROC curves typically feature true positive rate on the Y axis, and false
positive rate on the X axis. This means that the top left corner of the plot is
the "ideal" point - a false positive rate of zero, and a true positive rate of
one. This is not very realistic, but it does mean that a larger area under the
curve (AUC) is usually better.

The "steepness" of ROC curves is also important, since it is ideal to maximize
the true positive rate while minimizing the false positive rate.

Multiclass settings
-------------------

ROC curves are typically used in binary classification to study the output of
a classifier. In order to extend ROC curve and ROC area to multi-class
or multi-label classification, it is necessary to binarize the output. One ROC
curve can be drawn per label, but one can also draw a ROC curve by considering
each element of the label indicator matrix as a binary prediction
(micro-averaging).

Another evaluation measure for multi-class classification is
macro-averaging, which gives equal weight to the classification of each
label.

<div class="alert alert-info"><h4>Note</h4><p>See also :func:`sklearn.metrics.roc_auc_score`,
             `sphx_glr_auto_examples_model_selection_plot_roc_crossval.py`.</p></div>





```python
print(__doc__)

import numpy as np
import matplotlib.pyplot as plt
from itertools import cycle

from sklearn import svm, datasets
from sklearn.metrics import roc_curve, auc
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import label_binarize
from sklearn.multiclass import OneVsRestClassifier
from scipy import interp

# Import some data to play with
iris = datasets.load_iris()
X = iris.data
y = iris.target

# Binarize the output
y = label_binarize(y, classes=[0, 1, 2])
n_classes = y.shape[1]

# Add noisy features to make the problem harder
random_state = np.random.RandomState(0)
n_samples, n_features = X.shape
X = np.c_[X, random_state.randn(n_samples, 200 * n_features)]

# shuffle and split training and test sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=.5,
                                                    random_state=0)

# Learn to predict each class against the other
classifier = OneVsRestClassifier(svm.SVC(kernel='linear', probability=True,
                                 random_state=random_state))
y_score = classifier.fit(X_train, y_train).decision_function(X_test)
# print('y_score')
# print(y_score)
# Compute ROC curve and ROC area for each class
fpr = dict()
tpr = dict()
roc_auc = dict()
for i in range(n_classes):
    fpr[i], tpr[i], _ = roc_curve(y_test[:, i], y_score[:, i])
#     print(fpr[i])
#     print(tpr[i])
    roc_auc[i] = auc(fpr[i], tpr[i])

# # Compute micro-average ROC curve and ROC area
# fpr["micro"], tpr["micro"], _ = roc_curve(y_test.ravel(), y_score.ravel())
# roc_auc["micro"] = auc(fpr["micro"], tpr["micro"])
# # print(roc_auc)
```

    Automatically created module for IPython interactive environment
    [0.         0.         0.         0.01851852 0.01851852 0.03703704
     0.03703704 0.05555556 0.05555556 0.07407407 0.07407407 0.09259259
     0.09259259 0.12962963 0.12962963 0.14814815 0.14814815 0.2037037
     0.2037037  0.27777778 0.27777778 1.        ]
    [0.         0.04761905 0.14285714 0.14285714 0.19047619 0.19047619
     0.33333333 0.33333333 0.38095238 0.38095238 0.61904762 0.61904762
     0.66666667 0.66666667 0.76190476 0.76190476 0.9047619  0.9047619
     0.95238095 0.95238095 1.         1.        ]
    [0.         0.         0.         0.02222222 0.02222222 0.11111111
     0.11111111 0.17777778 0.17777778 0.2        0.2        0.24444444
     0.24444444 0.26666667 0.26666667 0.37777778 0.37777778 0.42222222
     0.42222222 0.48888889 0.48888889 0.55555556 0.55555556 0.62222222
     0.62222222 0.64444444 0.64444444 0.66666667 0.66666667 0.73333333
     0.73333333 0.75555556 0.75555556 0.88888889 0.88888889 1.        ]
    [0.         0.03333333 0.13333333 0.13333333 0.16666667 0.16666667
     0.2        0.2        0.26666667 0.26666667 0.33333333 0.33333333
     0.4        0.4        0.43333333 0.43333333 0.5        0.5
     0.56666667 0.56666667 0.6        0.6        0.63333333 0.63333333
     0.7        0.7        0.73333333 0.73333333 0.9        0.9
     0.93333333 0.93333333 0.96666667 0.96666667 1.         1.        ]
    [0.         0.         0.         0.01960784 0.01960784 0.07843137
     0.07843137 0.09803922 0.09803922 0.11764706 0.11764706 0.1372549
     0.1372549  0.15686275 0.15686275 0.17647059 0.17647059 0.31372549
     0.31372549 0.33333333 0.33333333 0.35294118 0.35294118 0.41176471
     0.41176471 0.45098039 0.45098039 0.47058824 0.47058824 0.50980392
     0.50980392 0.56862745 0.56862745 1.        ]
    [0.         0.04166667 0.125      0.125      0.25       0.25
     0.29166667 0.29166667 0.33333333 0.33333333 0.41666667 0.41666667
     0.5        0.5        0.54166667 0.54166667 0.58333333 0.58333333
     0.70833333 0.70833333 0.75       0.75       0.79166667 0.79166667
     0.83333333 0.83333333 0.875      0.875      0.91666667 0.91666667
     0.95833333 0.95833333 1.         1.        ]
    

Plot of a ROC curve for a specific class




```python
plt.figure()
lw = 2
plt.plot(fpr[2], tpr[2], color='darkorange',
         lw=lw, label='ROC curve (area = %0.2f)' % roc_auc[2])
plt.plot([0, 1], [0, 1], color='navy', lw=lw, linestyle='--')
plt.xlim([0.0, 1.0])
plt.ylim([0.0, 1.05])
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('Receiver operating characteristic example')
plt.legend(loc="lower right")
plt.show()
```


![png](plot_roc_files/plot_roc_4_0.png)


Plot ROC curves for the multiclass problem




```python
# Compute macro-average ROC curve and ROC area

# First aggregate all false positive rates
all_fpr = np.unique(np.concatenate([fpr[i] for i in range(n_classes)]))

# Then interpolate all ROC curves at this points
mean_tpr = np.zeros_like(all_fpr)
for i in range(n_classes):
    mean_tpr += interp(all_fpr, fpr[i], tpr[i])

# Finally average it and compute AUC
mean_tpr /= n_classes

fpr["macro"] = all_fpr
tpr["macro"] = mean_tpr
roc_auc["macro"] = auc(fpr["macro"], tpr["macro"])

# Plot all ROC curves
plt.figure()
plt.plot(fpr["micro"], tpr["micro"],
         label='micro-average ROC curve (area = {0:0.2f})'
               ''.format(roc_auc["micro"]),
         color='deeppink', linestyle=':', linewidth=4)

plt.plot(fpr["macro"], tpr["macro"],
         label='macro-average ROC curve (area = {0:0.2f})'
               ''.format(roc_auc["macro"]),
         color='navy', linestyle=':', linewidth=4)

colors = cycle(['aqua', 'darkorange', 'cornflowerblue'])
for i, color in zip(range(n_classes), colors):
    plt.plot(fpr[i], tpr[i], color=color, lw=lw,
             label='ROC curve of class {0} (area = {1:0.2f})'
             ''.format(i, roc_auc[i]))

plt.plot([0, 1], [0, 1], 'k--', lw=lw)
plt.xlim([0.0, 1.0])
plt.ylim([0.0, 1.05])
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('Some extension of Receiver operating characteristic to multi-class')
plt.legend(loc="lower right")
plt.show()
```

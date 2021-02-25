---
author: "李昌"
title: "快速入门Matplotlib教程"
date: "2021-02-25"
tags: ["Python"]
categories: ["Python"]
ShowToc: true
TocOpen: true
---

# 快速入门Matplotlib教程 

## 介绍 

Matplotlib 可能是 Python 2D-绘图领域使用最广泛的套件。它能让使用者很轻松地将数据图形化，并且提供多样化的输出格式。

## pylab

pylab 是 matplotlib 面向对象绘图库的一个接口。它的语法和 Matlab 十分相近。也就是说，它主要的绘图命令和 Matlab 对应的命令有相似的参数。

## 初级绘制 

这一节中，我们将从简到繁：先尝试用默认配置在同一张图上绘制正弦和余弦函数图像，然后逐步美化它。

第一步，是取得正弦函数和余弦函数的值：


```python
import numpy as np
X = np.linspace(-np.pi, np.pi, 256,endpoint=True)
C,S = np.cos(X), np.sin(X)
```

X 是一个 numpy 数组，包含了从 −π−π 到 +π+π 等间隔的 256 个值。C和 S 则分别是这 256 个值对应的余弦和正弦函数值组成的 numpy 数组。  
[np.linspace](#np.linspace)

## 使用默认配置 

Matplotlib 的默认配置都允许用户自定义。你可以调整大多数的默认配置：图片大小和分辨率（dpi）、线宽、颜色、风格、坐标轴、坐标轴以及网格的属性、文字与字体属性等。不过，matplotlib 的默认配置在大多数情况下已经做得足够好，你可能只在很少的情况下才会想更改这些默认配置。  
[plot函数详解](https://blog.csdn.net/cjcrxzz/article/details/79627483)


```python
from pylab import *
plot(X,C)
plot(X,S)
show()
```


    <Figure size 640x480 with 1 Axes>


## 默认配置的具体内容 

下面的代码中，我们展现了 matplotlib 的默认配置并辅以注释说明，这部分配置包含了有关绘图样式的所有配置。代码中的配置与默认配置完全相同，你可以在交互模式中修改其中的值来观察效果。


```python
# 导入 matplotlib 的所有内容（nympy 可以用 np 这个名字来使用）
from pylab import *
# 创建一个 8 * 6 点（point）的图，并设置分辨率为 80
figure(figsize=(8,6), dpi=80)
# 创建一个新的 1 * 1 的子图，接下来的图样绘制在其中的第 1 块（也是唯一的一块）
subplot(1,1,1)
X = np.linspace(-np.pi, np.pi, 256,endpoint=True)
C,S = np.cos(X), np.sin(X)
# 绘制余弦曲线，使用蓝色的、连续的、宽度为 1 （像素）的线条
plot(X, C, color="blue", linewidth=1.0, linestyle="-")
# 绘制正弦曲线，使用绿色的、连续的、宽度为 1 （像素）的线条
plot(X, S, color="green", linewidth=1.0, linestyle="-")
# 设置横轴的上下限
xlim(-4.0,4.0)
# 设置横轴记号
xticks(np.linspace(-4,4,9,endpoint=True))
# 设置纵轴的上下限
ylim(-1.0,1.0)
# 设置纵轴记号
yticks(np.linspace(-1,1,5,endpoint=True))
# 以分辨率 72 来保存图片
# savefig("exercice_2.png",dpi=72)
# 在屏幕上显示
show()
```


    <Figure size 640x480 with 1 Axes>


## 改变线条的颜色和粗细


```python
figure(figsize=(10,6), dpi=80)
plot(X, C, color="blue", linewidth=10, linestyle="-")
plot(X, S, color="red",  linewidth=2.5, linestyle="-")

```




    [<matplotlib.lines.Line2D at 0x7f6c66774470>]




![png](matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_files/matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_17_1.png)


## 设置图片边界 

当前的图片边界设置得不好，所以有些地方看得不是很清楚。


```python

from pylab import *
figure(figsize=(8,6), dpi=80)
subplot(1,1,1)
X = np.linspace(-np.pi, np.pi, 256,endpoint=True)
C,S = np.cos(X), np.sin(X)
plot(X, C, color="blue", linewidth=1.0, linestyle="-")
plot(X, S, color="green", linewidth=1.0, linestyle="-")
xticks(np.linspace(-4,4,9,endpoint=True))

xlim(X.min()*1.1, X.max()*1.1)  # 此处发生改变
ylim(C.min()*1.1, C.max()*1.1)  # 此处发生改变

yticks(np.linspace(-1,1,5,endpoint=True))
show()
```


![png](matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_files/matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_20_0.png)


## 设置记号 

我们讨论正弦和余弦函数的时候，通常希望知道函数在 $±π±π$ 和 
$±π/2±π/2$ 的值。这样看来，当前的设置就不那么理想了。


```python
from pylab import *
figure(figsize=(8,6), dpi=80)
subplot(1,1,1)
X = np.linspace(-np.pi, np.pi, 256,endpoint=True)
C,S = np.cos(X), np.sin(X)
plot(X, C, color="blue", linewidth=1.0, linestyle="-")
plot(X, S, color="green", linewidth=1.0, linestyle="-")
xlim(X.min()*1.1, X.max()*1.1)  
ylim(C.min()*1.1, C.max()*1.1) 

xticks( [-np.pi, -np.pi/2, 0, np.pi/2, np.pi])  # 此处发生改变
yticks([-1, 0, +1])  # 此处发生改变

show()
```


![png](matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_files/matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_23_0.png)


## 设置记号的标签

记号现在没问题了，不过标签却不大符合期望。我们可以把 3.142 当做是 $π$，但毕竟不够精确。当我们设置记号的时候，我们可以同时设置记号的标签。注意这里使用了 LaTeX。(以后会讲LaTeX)


```python
from pylab import *
figure(figsize=(8,6), dpi=80)
subplot(1,1,1)
X = np.linspace(-np.pi, np.pi, 256,endpoint=True)
C,S = np.cos(X), np.sin(X)
plot(X, C, color="blue", linewidth=1.0, linestyle="-")
plot(X, S, color="green", linewidth=1.0, linestyle="-")
xlim(X.min()*1.1, X.max()*1.1)  
ylim(C.min()*1.1, C.max()*1.1)  

plt.xticks([-np.pi, -np.pi/2, 0, np.pi/2, np.pi],[r'$-\pi$', r'$-\pi/2$', r'$0$', r'$+\pi/2$', r'$+\pi$'])  # 此处发生改变
plt.yticks([-1, 0, +1],[r'$-1$', r'$0$', r'$+1$'])  # 此处发生改变

show()
```


![png](matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_files/matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_26_0.png)


## 移动脊柱 

坐标轴线和上面的记号连在一起就形成了脊柱（Spines，一条线段上有一系列的凸起，是不是很像脊柱骨啊~），它记录了数据区域的范围。它们可以放在任意位置，不过至今为止，我们都把它放在图的四边。

实际上每幅图有四条脊柱（上下左右），为了将脊柱放在图的中间，我们必须将其中的两条（上和右）设置为无色，然后调整剩下的两条到合适的位置——数据空间的 0 点。


```python
from pylab import *
figure(figsize=(8,6), dpi=80)
subplot(1,1,1)
X = np.linspace(-np.pi, np.pi, 256,endpoint=True)
C,S = np.cos(X), np.sin(X)
plot(X, C, color="blue", linewidth=1.0, linestyle="-")
plot(X, S, color="green", linewidth=1.0, linestyle="-")
xlim(X.min()*1.1, X.max()*1.1)  
plt.xticks([-np.pi, -np.pi/2, 0, np.pi/2, np.pi],[r'$-\pi$', r'$-\pi/2$', r'$0$', r'$+\pi/2$', r'$+\pi$']) 
ylim(C.min()*1.1, C.max()*1.1)  
plt.yticks([-1, 0, +1],[r'$-1$', r'$0$', r'$+1$'])  

# 移动脊柱
ax = plt.gca()
ax.spines['right'].set_color('none')
ax.spines['top'].set_color('none')
ax.xaxis.set_ticks_position('bottom')
ax.spines['bottom'].set_position(('data',0))
ax.yaxis.set_ticks_position('left')
ax.spines['left'].set_position(('data',0))
show()
```


![png](matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_files/matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_29_0.png)


## 添加图例

我们在图的左上角添加一个图例。为此，我们只需要在 plot 函数里以「键 - 值」的形式增加一个参数。


```python
from pylab import *
figure(figsize=(8,6), dpi=80)
subplot(1,1,1)
X = np.linspace(-np.pi, np.pi, 256,endpoint=True)
C,S = np.cos(X), np.sin(X)
plot(X, C, color="blue", linewidth=1.0, linestyle="-")
plot(X, S, color="green", linewidth=1.0, linestyle="-")
xlim(X.min()*1.1, X.max()*1.1)  
plt.xticks([-np.pi, -np.pi/2, 0, np.pi/2, np.pi],[r'$-\pi$', r'$-\pi/2$', r'$0$', r'$+\pi/2$', r'$+\pi$']) 
ylim(C.min()*1.1, C.max()*1.1)  
plt.yticks([-1, 0, +1],[r'$-1$', r'$0$', r'$+1$'])  
# 移动脊柱
ax = plt.gca()
ax.spines['right'].set_color('none')
ax.spines['top'].set_color('none')
ax.xaxis.set_ticks_position('bottom')
ax.spines['bottom'].set_position(('data',0))
ax.yaxis.set_ticks_position('left')
ax.spines['left'].set_position(('data',0))
# 增加图例
plt.plot(X, C, color="blue", linewidth=2.5, linestyle="-", label="cosine")
plt.plot(X, S, color="red",  linewidth=2.5, linestyle="-", label="sine")
plt.legend(loc='upper left', frameon=False)
show()
```


![png](matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_files/matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_32_0.png)


## 其他类型的图 

## 普通图


```python
import numpy as np
import matplotlib.pyplot as plt

n = 256
X = np.linspace(-np.pi,np.pi,n,endpoint=True)
Y = np.sin(2*X)

plt.axes([0.025,0.025,0.95,0.95])

plt.plot (X, Y+1, color='blue', alpha=1.00)
plt.fill_between(X, 1, Y+1, color='blue', alpha=.25)

plt.plot (X, Y-1, color='blue', alpha=1.00)
plt.fill_between(X, -1, Y-1, (Y-1) > -1, color='blue', alpha=.25)
plt.fill_between(X, -1, Y-1, (Y-1) < -1, color='red',  alpha=.25)

plt.xlim(-np.pi,np.pi), plt.xticks([])
plt.ylim(-2.5,2.5), plt.yticks([])
# savefig('../figures/plot_ex.png',dpi=48)
plt.show()
```


![png](matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_files/matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_35_0.png)


## 散点图


```python
import numpy as np
import matplotlib.pyplot as plt

n = 1024
X = np.random.normal(0,1,n)
Y = np.random.normal(0,1,n)
T = np.arctan2(Y,X)

plt.axes([0.025,0.025,0.95,0.95])
plt.scatter(X,Y, s=75, c=T, alpha=.5)

plt.xlim(-1.5,1.5), plt.xticks([])
plt.ylim(-1.5,1.5), plt.yticks([])
# savefig('../figures/scatter_ex.png',dpi=48)
plt.show()
```


![png](matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_files/matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_37_0.png)


##  条形图


```python
import numpy as np
import matplotlib.pyplot as plt

n = 12
X = np.arange(n)
Y1 = (1-X/float(n)) * np.random.uniform(0.5,1.0,n)
Y2 = (1-X/float(n)) * np.random.uniform(0.5,1.0,n)

plt.axes([0.025,0.025,0.95,0.95])
plt.bar(X, +Y1, facecolor='#9999ff', edgecolor='white')
plt.bar(X, -Y2, facecolor='#ff9999', edgecolor='white')

for x,y in zip(X,Y1):
    plt.text(x+0.4, y+0.05, '%.2f' % y, ha='center', va= 'bottom')

for x,y in zip(X,Y2):
    plt.text(x+0.4, -y-0.05, '%.2f' % y, ha='center', va= 'top')

plt.xlim(-.5,n), plt.xticks([])
plt.ylim(-1.25,+1.25), plt.yticks([])

# savefig('../figures/bar_ex.png', dpi=48)
plt.show()
```


![png](matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_files/matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_39_0.png)


## 等高线图


```python
import numpy as np
import matplotlib.pyplot as plt

def f(x,y):
    return (1-x/2+x**5+y**3)*np.exp(-x**2-y**2)

n = 256
x = np.linspace(-3,3,n)
y = np.linspace(-3,3,n)
X,Y = np.meshgrid(x,y)

plt.axes([0.025,0.025,0.95,0.95])

plt.contourf(X, Y, f(X,Y), 8, alpha=.75, cmap=plt.cm.hot)
C = plt.contour(X, Y, f(X,Y), 8, colors='black', linewidth=.5)
plt.clabel(C, inline=1, fontsize=10)

plt.xticks([]), plt.yticks([])
# savefig('../figures/contour_ex.png',dpi=48)
plt.show()
```

    /usr/lib/python3.7/site-packages/matplotlib/contour.py:1000: UserWarning: The following kwargs were not used by contour: 'linewidth'
      s)
    


![png](matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_files/matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_41_1.png)


## 灰度图


```python
import numpy as np
import matplotlib.pyplot as plt

def f(x,y):
    return (1-x/2+x**5+y**3)*np.exp(-x**2-y**2)

n = 10
x = np.linspace(-3,3,3.5*n)
y = np.linspace(-3,3,3.0*n)
X,Y = np.meshgrid(x,y)
Z = f(X,Y)

plt.axes([0.025,0.025,0.95,0.95])
plt.imshow(Z,interpolation='bicubic', cmap='bone', origin='lower')
plt.colorbar(shrink=.92)

plt.xticks([]), plt.yticks([])
# savefig('../figures/imshow_ex.png', dpi=48)
plt.show()
```

    /usr/lib/python3.7/site-packages/ipykernel_launcher.py:8: DeprecationWarning: object of type <class 'float'> cannot be safely interpreted as an integer.
      
    /usr/lib/python3.7/site-packages/ipykernel_launcher.py:9: DeprecationWarning: object of type <class 'float'> cannot be safely interpreted as an integer.
      if __name__ == '__main__':
    


![png](matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_files/matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_43_1.png)


## 饼状图 


```python
import numpy as np
import matplotlib.pyplot as plt

n = 20
Z = np.ones(n)
Z[-1] *= 2

plt.axes([0.025, 0.025, 0.95, 0.95])

plt.pie(Z, explode=Z*.05, colors=['%f' % (i/float(n)) for i in range(n)],
        wedgeprops={"linewidth": 1, "edgecolor": "black"})
plt.gca().set_aspect('equal')
plt.xticks([]), plt.yticks([])

# savefig('../figures/pie_ex.png',dpi=48)
plt.show()
```


![png](matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_files/matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_45_0.png)


## 极轴图


```python
import numpy as np
import matplotlib.pyplot as plt

ax = plt.axes([0.025,0.025,0.95,0.95], polar=True)

N = 20
theta = np.arange(0.0, 2*np.pi, 2*np.pi/N)
radii = 10*np.random.rand(N)
width = np.pi/4*np.random.rand(N)
bars = plt.bar(theta, radii, width=width, bottom=0.0)

for r,bar in zip(radii, bars):
    bar.set_facecolor( plt.cm.jet(r/10.))
    bar.set_alpha(0.5)

ax.set_xticklabels([])
ax.set_yticklabels([])
# savefig('../figures/polar_ex.png',dpi=48)
plt.show()
```


![png](matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_files/matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_47_0.png)


## 3D图 


```python
import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D

fig = plt.figure()
ax = Axes3D(fig)
X = np.arange(-4, 4, 0.25)
Y = np.arange(-4, 4, 0.25)
X, Y = np.meshgrid(X, Y)
R = np.sqrt(X**2 + Y**2)
Z = np.sin(R)

ax.plot_surface(X, Y, Z, rstride=1, cstride=1, cmap=plt.cm.hot)
ax.contourf(X, Y, Z, zdir='z', offset=-2, cmap=plt.cm.hot)
ax.set_zlim(-2,2)

# savefig('../figures/plot3d_ex.png',dpi=48)
plt.show()
```


![png](matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_files/matplotlib%E5%88%9D%E7%BA%A7%E6%95%99%E7%A8%8B_49_0.png)


## 作业 

熟悉以上画图方法，从其官方文档中了解“**子图**”的概念，将[散点图， 条形图， 等高线图， 灰度图， 饼状图， 极轴图， 3D图]放在一张$3*2$的子图中

## 附：参考资料 

* [matplotlib官方文档](https://matplotlib.org/contents.html)  
* [matplotlib中文文档](https://www.matplotlib.org.cn/)  
* [matplotlib官方教程](https://matplotlib.org/tutorials/index.html)  
* [详解图像各个部分 matplotlib](https://www.cnblogs.com/nju2014/p/5620776.html)

### np.linspace

numpy.linspace(start, stop, num=50, endpoint=True, retstep=False, dtype=None)  
在指定的间隔内返回均匀间隔的数字。  
返回num均匀分布的样本，在[start, stop]。  
这个区间的端点可以任意的被排除在外。  

* 参数：
    - start ： array_like  
        序列的起始值。

    - stop ： array_like
        序列的结束值，除非端点设置为False。在这种情况下，序列由除 均匀间隔的样本之外的所有样本组成，因此不包括停止。请注意，当端点为False 时，步长会发生变化。num + 1

    - num ： int，可选
        要生成的样本数。默认值为50.必须为非负数。

    - endpoint ： bool，可选
        如果为True，则stop是最后一个样本。否则，它不包括在内。默认为True。

    - retstep ： bool，可选
        如果为True，则返回（samples，step），其中step是样本之间的间距。

    - dtype ： dtype，可选
        输出数组的类型。如果dtype未给出，则从其他输入参数推断数据类型。
* 返回
    - samples : ndarray
        有NUM同样在闭区间隔开的样品 或半开间隔 （取决于是否端点是真或假）。[start, stop][start, stop)

    - step : float, optional
        仅在retstep为True 时才返回样本之间的间距大小。  
        
[返回初级配置](#初级绘制)

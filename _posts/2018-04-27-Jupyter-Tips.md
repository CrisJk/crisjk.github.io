---
layout: post
author: Kuang
title: Jupyter notebook的一些tips
categories: Tools
tags: jupyter
---
>  引用自[https://www.dataquest.io/blog/jupyter-notebook-tips-tricks-shortcuts/](https://www.dataquest.io/blog/jupyter-notebook-tips-tricks-shortcuts/)

工欲善其事必先利其器，Jupyter Notebook可以说是数据分析的一大杀器，所以熟悉一下这件武器的一些"机关"可以让我们达到事半功倍的效果。



### 1. 键盘快捷键

页面顶部的菜单栏中,`Help>Keyboard Shortcuts` ，或者在命令模式中按下`H`　，可以得到快捷键列表。注意经常检查这个列表的更新。

`ctrl+Shift+P` 非常有用，当你忘记了一些快捷键，或者你想做的事情没有快捷键，按下这个组合键，可以召唤出一个对话框，这个功能类似于Mac系统中的Spotlight。

一些常用的快捷键:

* `ESC` 进入命令模式
* 在命令模式中：
  * `A`在当前cell上面插入一个新的cell(Above)，`B` 在当前cell下面插入一个新的cell(Below)
  * 　`M` 将当前的cell改成markdown,`Y` 将当前cell改成code
  * 　`D + D` 删除当前cell
* `Enter` 从命令模式切回编辑模式
* `Ctrl + Shift + -` 将当前cell从光标所在位置分成两个cell
* `Esc + F` 在代码中寻找和替换
* `Esc + O` 收起输出
* 选择多个cells:
  * `Shift + J` 或者`Shift + Down`选择下一个cell, `Shift + K`或者`Shift + Up` 选择上一个cell
  * 当cells被选择时，可以批量进行删除/复制/剪切/粘贴/运行
  * 使用`Shift + M` 可以合并多个cells



### 2.展示

众所周知，jupyter不使用print语句就能展示变量。这对pandas Dataframes尤其有用，因为输出是表格的形式。

很少人知道的是，你可以修改`ast_note_interactivity` 的选项，使得jupyter对所有的语句和变量执行这一选项，这样一次能查看所有变量的值。（我在测试的时候发现好像不配置这个也可以显示所有变量)

```python
from IPython.core.interactiveshell import InteractiveShell
InteractiveShell.ast_node_interactivity = "all"
```

```python
from pydataset import data
quakes = data('quakes')
quakes.head()
quakes.tail()
```

| lat  | long   | depth  | mag  | stations |      |
| ---- | ------ | ------ | ---- | -------- | ---- |
| 1    | -20.42 | 181.62 | 562  | 4.8      | 41   |
| 2    | -20.62 | 181.03 | 650  | 4.2      | 15   |
| 3    | -26.00 | 184.10 | 42   | 5.4      | 43   |
| 4    | -17.97 | 181.66 | 626  | 4.1      | 19   |
| 5    | -20.42 | 181.96 | 649  | 4.0      | 11   |

|      | lat    | long   | depth | mag  | stations |
| ---- | ------ | ------ | ----- | ---- | -------- |
| 996  | -25.93 | 179.54 | 470   | 4.4  | 22       |
| 997  | -12.28 | 167.06 | 248   | 4.7  | 35       |
| 998  | -20.13 | 184.20 | 244   | 4.5  | 34       |
| 999  | -17.40 | 187.80 | 40    | 4.5  | 14       |
| 1000 | -21.59 | 170.56 | 165   | 6.0  | 119      |



如果你想对于每个Jupyter实例都应用该操作，却又不想每次都在开头进行修改，那么你可以修改配置文件,`~/.ipython/profile_default/ipython_config.py` ，填入以下内容

```
c = get_config()

# Run all nodes interactively
c.InteractiveShell.ast_node_interactivity = "all"
```

### 3.快速查找文档

在`Help` 菜单中，你能找到一些常用库的文档，包括Numpy，Pandas，SciPy和Matplotlib

还有一个更加方便的方法，就是在库、方法或者变量前加`?`

```python
?str.replace()
```

```python
Docstring:
S.replace(old, new[, count]) -> str

Return a copy of S with all occurrences of substring
old replaced by new.  If the optional argument count is
given, only the first count occurrences are replaced.
Type:      method_descriptor
```

### 4.在notebook中plotting

在notebook中有多种数据可视化的方法:

* **matplotlib** 是非常常用的绘图工具，使用`%matplotlib inline` 激活内嵌绘图功能，这样不用show也能看到结果(然而我在实验室不加`%matplotlib inline` 效果也一样，不知道是不是版本更新导致这些操作被省略)。([Matplotlib Tutorial](https://www.dataquest.io/blog/matplotlib-tutorial/))
* %matplotlib notebook提供了一个交互性更强的功能，但是速度上稍微有点慢
* [Seaborn](http://seaborn.pydata.org/) 是基于Matplotlib构建的，并且更容易构建更attractive的画面
* [mpld3](https://github.com/mpld3/mpld3) 提供了一个可替代的渲染器
* [bokeh](http://bokeh.pydata.org/en/latest/) 一个不错的选择for可交互的plots
* [plot.ly](https://plot.ly/) 可以产生很好的plots
* [Altair](https://github.com/altair-viz/altair) 是一个相对较新的可视化库。它使用方便，并且可以得到一些好看的plots

### 5. Ipython Magic Commands

```python
# This will list all magic commands
%lsmagic
```

[Magic Commands文档](http://ipython.readthedocs.io/en/stable/interactive/magics.html)

### 6. IPython Magic - %env: Set Environment Variables

不需要重启jupyter来设置环境变量。

```python
# Running %env without any arguments
# lists all environment variables

# The line below sets the environment
# variable OMP_NUM_THREADS
%env OMP_NUM_THREADS=4
```

```python
env: OMP_NUM_THREADS=4
```

### 7. IPython Magic - %run: Execute python code

`%run` 可以执行.py文件的python代码。更不为人知的是，该命令同样可以用来执行.ipynb文件。

```python
# this will execute and show the output from
# all code cells of the specified notebook
%run ./two-histograms.ipynb
```

![](https://www.dataquest.io/blog/content/images/two-hists.png)

### 8. IPython Magic - %load: Insert the code from an external script

`%load` 命令可以从外部脚本加载代码，你既能从本地也能从外部的url加载脚本

```python
# Before Running
%load ./hello_world.py
```



```python
# After Running
# %load ./hello_world.py
if __name__ == "__main__":
	print("Hello World!")
```

```python
Hello World!
```

### 9. IPython Magic - %store: Pass variables between notebooks.

`%store` 命令帮助你在两个notebook中传递变量

```python
data = 'this is the string I want to pass to different notebook'
%store data
del data # This has deleted the variable
```

```python
Stored 'data' (str)
```

现在，在一个新的notebook中

```python
%store -r data
print(data)
```

```python
this is the string I want to pass to different notebook
```

### 10. IPython Magic - %who: List all variables of global scope.

`%who` 命令如果不带任何参数，则会列出全局范围内的所有变量。如果输入参数类似于`str` 将会只列出特定类型的变量

```python
one = "for the money"
two = "for the show"
three = "to get ready now go cat go" 
%who str
```

```python
one	 three	 two
```

### 11. IPython Magic - Timing

`%%time` 和`%timeit` 是Ipython Magic commands中两个有用的关于时间的命令。

`%%time` 告诉你单个核运行的时间

```python
%%time
import time
for _ in range(1000):
    time.sleep(0.01)# sleep for 0.01 seconds
```

```python
CPU times: user 21.5 ms, sys: 14.8 ms, total: 36.3 ms
Wall time: 11.6 s
```

`%%timeit` 使用python 的timeit module，它将一条语句执行100,000次，然后给出运行最快的3次的平均值

```python
import numpy
%timeit numpy.random.normal(size=100)
```

```python
The slowest run took 7.29 times longer than the fastest. This could mean that an intermediate result is being cached.
100000 loops, best of 3: 5.5 µs per loop
```

### 12. IPython Magic - %%writefile and %pycat: Export the contents of a cell/Show the contents of an external script

使用`%%writefile` magic来保存当前cell到一个外部文件。`%pycat` 则是做相反的操作

```python
%%writefile pythoncode.py

import numpy
def append_if_not_exists(arr, x):
    if x not in arr:
        arr.append(x)

def some_useless_slow_function():
    arr = list()
    for i in range(10000):
        x = numpy.random.randint(0, 10000)
        append_if_not_exists(arr, x)
```

```python
Writing pythoncode.py
```

```python
%pycat pythoncode.py
```

```python
import numpy
def append_if_not_exists(arr, x):
    if x not in arr:
        arr.append(x)

def some_useless_slow_function():
    arr = list()
    for i in range(10000):
        x = numpy.random.randint(0, 10000)
        append_if_not_exists(arr, x)
```



### 13. IPython Magic - %prun: Show how much time your program spent in each function.

使用 `%prun statement_name` 将得到一个有序的表格，该表格展示了每个函数被调用的次数和运行的时间

```python
%prun some_useless_slow_function()
```

```python
26324 function calls in 0.556 seconds

Ordered by: internal time

ncalls  tottime  percall  cumtime  percall filename:lineno(function)
 10000    0.527    0.000    0.528    0.000 <ipython-input-46-b52343f1a2d5>:2(append_if_not_exists)
 10000    0.022    0.000    0.022    0.000 {method 'randint' of 'mtrand.RandomState' objects}
     1    0.006    0.006    0.556    0.556 <ipython-input-46-b52343f1a2d5>:6(some_useless_slow_function)
  6320    0.001    0.000    0.001    0.000 {method 'append' of 'list' objects}
     1    0.000    0.000    0.556    0.556 <string>:1(<module>)
     1    0.000    0.000    0.556    0.556 {built-in method exec}
     1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
```

### 14. IPython Magic - Debugging with %pdb

Jupyter拥有自己的python debugger(pdb)接口. 这使得在Jupyter notebook中调试变成可能。

You can view [a list of accepted commands for `pdb` here](https://docs.python.org/3.5/library/pdb.html#debugger-commands).

```python
%pdb

def pick_and_take():
    picked = numpy.random.randint(0, 1000)
    raise NotImplementedError()

pick_and_take()
```

```python
Automatic pdb calling has been turned ON
```

```python
---------------------------------------------------------------------------
NotImplementedError                       Traceback (most recent call last)
<ipython-input-24-0f6b26649b2e> in <module>()
      5     raise NotImplementedError()
      6 
----> 7 pick_and_take()

<ipython-input-24-0f6b26649b2e> in pick_and_take()
      3 def pick_and_take():
      4     picked = numpy.random.randint(0, 1000)
----> 5     raise NotImplementedError()
      6 
      7 pick_and_take()

NotImplementedError:
```

```python
> <ipython-input-24-0f6b26649b2e>(5)pick_and_take()
      3 def pick_and_take():
      4     picked = numpy.random.randint(0, 1000)
----> 5     raise NotImplementedError()
      6 
      7 pick_and_take()
```

```python
ipdb>
```

### 15. IPython Magic - High-resolution plot outputs for Retina notebooks

使画出来的图支持retina

```python
x = range(1000)

y = [i ** 2 for i in x]

plt.plot(x,y)

plt.show();

```

```python
%config InlineBackend.figure_format = 'retina'
plt.plot(x,y)
plt.show();
```

## 16. Suppress the output of a final function.

有的时候我们只要输出图，而不需要其他输出。这时候一个分号就能解决问题。

```python
%matplotlib inline
from matplotlib import pyplot as plt
import numpy
x = numpy.linspace(0, 1, 1000)**1.5
```

```python
# Here you get the output of the function
plt.hist(x)
```

```python
(array([216., 126., 106.,  95.,  87.,  81.,  77.,  73.,  71.,  68.]),
 array([0. , 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1. ]),
 <a list of 10 Patch objects>)
```

![](https://www.dataquest.io/blog/content/images/hist.png)

```python
# Here you get the output of the function
plt.hist(x);
```

![](https://www.dataquest.io/blog/content/images/hist.png)

## 17. Executing Shell Commands

在jupyter notebook中执行Shell命令，想想就很方便

```python
!ls *.csv
```

```python
nba_2016.csv             titanic.csv
pixar_movies.csv         whitehouse_employees.csv
```

```python
!pip install numpy
!pip list | grep pandas
```

```python
Requirement already satisfied (use --upgrade to upgrade): numpy in /Library/Frameworks/Python.framework/Versions/3.4/lib/python3.4/site-packages
pandas (0.18.1)
```

## 18. Using LaTeX for forumlas

在markdown中插入公式

```python
\\( P(A \mid B) = \frac{P(B \mid A) \, P(A)}{P(B)} \\)
```

得到

$$P(A \mid B) = \frac{P(B \mid A) , P(A)}{P(B)}$$

## 19. Run code from a different kernel in a notebook

你可以在同一个notebook中运行不同的语言

只需要在每个cell前面加上一个magic command

- `%%bash`
- `%%HTML`
- `%%python2`
- `%%python3`
- `%%ruby`
- `%%perl`

```python
%%bash
for i in {1..5}
do
   echo "i is $i"
done
```

```python
i is 1
i is 2
i is 3
i is 4
i is 5
```



## 20. Running R and Python in the same notebook.

最好的解决方案是安装rpy2(同样也需要R语言的环境)

```python
pip install rpy2
```

你能同时使用这两种语言，甚至在它们之间传递变量

```python
%load_ext rpy2.ipython
```

```python
%R require(ggplot2)
```

```python
array([1], dtype=int32)
```

```python
import pandas as pd
df = pd.DataFrame({
        'Letter': ['a', 'a', 'a', 'b', 'b', 'b', 'c', 'c', 'c'],
        'X': [4, 3, 5, 2, 1, 7, 7, 5, 9],
        'Y': [0, 4, 3, 6, 7, 10, 11, 9, 13],
        'Z': [1, 2, 3, 1, 2, 3, 1, 2, 3]
    })
```

```python
%%R -i df
ggplot(data = df) + geom_point(aes(x = X, y= Y, color = Letter, size = Z))
```

![](https://www.dataquest.io/blog/content/images/ggplot.png)

*Example courtesy Revolutions Blog*

## 21. Writing functions in other languages

有些时候numpy的速度不够快，因此有时需要一些其他的语言。当然，你可以把这块代码打包成dynamic library。但这个过程比较繁琐，因此jupyter notebook允许直接在代码里使用Cython或fortran.

首先，需要安装

```python
!pip install cython fortran-magic 
```

```python
%load_ext Cython
```

```python
%%cython
def myltiply_by_2(float x):
    return 2.0 * x
```

```python
myltiply_by_2(23.)
```

## 23. Multicursor support

按下`Alt`键

## 24. Jupyter-contrib extensions

[Jupyter-contrib extensions](https://github.com/ipython-contrib/jupyter_contrib_nbextensions) 包含一系列扩展，例如`jupyter spell-checker` 和`code-formatter`

可以利用命令安装扩展，之后可以利用浏览器上的可视化工具来管理扩展.（这里我并不是按照如下命令执行，而是在bash中执行相应命令，因为我需要用到root权限)

```python
!pip install https://github.com/ipython-contrib/jupyter_contrib_nbextensions/tarball/master
!pip install jupyter_nbextensions_configurator
!jupyter contrib nbextension install --user
!jupyter nbextensions_configurator enable --user
```

![](https://www.dataquest.io/blog/content/images/nbextensions.png)

## 25. Create a presentation from a Jupyter notebook.

Damian Avila's [RISE](https://github.com/damianavila/RISE) 让你可以从现有的notebook中创建powerpoint风格的presentation.

可以利用conda安装RISE:

```python
conda install -c damianavila82 rise
```

当然我本人不太喜欢用conda这些工具，所以可以选择pip来安装

```python
pip install RISE
```



运行以下命令来开启这个扩展

```python
jupyter-nbextension install rise --py --sys-prefix
jupyter-nbextension enable rise --py --sys-prefix
```

## 26. The Jupyter output system

Notebook的display是基于HTML的，因此你可以输出任何东西:包括视频/音频/图像

下面这个例子展示的是文件系统中前5个缩略图

```python
import os
from IPython.display import display, Image
names = [f for f in os.listdir('../images/ml_demonstrations/') if f.endswith('.png')]
for name in names[:5]:
    display(Image('../images/ml_demonstrations/' + name, width=100)
```

![](https://www.dataquest.io/blog/content/images/img1.png)

![](https://www.dataquest.io/blog/content/images/img2.png)



![](https://www.dataquest.io/blog/content/images/img3.png)

![](https://www.dataquest.io/blog/content/images/img4.png)

![img](https://www.dataquest.io/blog/content/images/img5.png)

你能使用bash命令创建一个同样的列表，因为magics和bash calls返回的是python变量

```python
names = !ls ../images/ml_demonstrations/*.png
names[:5]
```

```python
['../images/ml_demonstrations/colah_embeddings.png',
 '../images/ml_demonstrations/convnetjs.png',
 '../images/ml_demonstrations/decision_tree.png',
 '../images/ml_demonstrations/decision_tree_in_course.png',
 '../images/ml_demonstrations/dream_mnist.png']
```

## 27. 'Big data' analysis

- [ipyparallel (formerly ipython cluster)](https://github.com/ipython/ipyparallel) is a good option for simple map-reduce operations in python. We use it in [rep](https://github.com/yandex/rep) to train many machine learning models in parallel
- [pyspark](http://www.cloudera.com/documentation/enterprise/5-5-x/topics/spark_ipython.html)
- spark-sql magic [%%sql](https://github.com/jupyter-incubator/sparkmagic)

## 28. Sharing notebooks

最简单的方法是使用.ipynb文件,但是你想分享给不适用jupyter的其他人的话，可以采用一下方法:

* 将其转换成HTML,　`File > Dowload as> HTML `
* 使用gist或者github，上传即可
* 使用[jupyterhub](https://github.com/jupyterhub/jupyterhub) 在你自己的系统上部署
* Store your notebook e.g. in dropbox and put the link to [nbviewer](http://nbviewer.jupyter.org/). nbviewer will render the notebook from whichever source you host it.
* Use the `File > Download as > PDF` menu to save your notebook as a PDF. If you're going this route, I highly recommend reading Julius Schulz's excellent article [Making publication ready Python notebooks](http://blog.juliusschulz.de/blog/ultimate-ipython-notebook).
* [Create a blog using Pelican from your Jupyter notebooks](https://www.dataquest.io/blog/how-to-setup-a-data-science-blog/).(这个可能比较因吹斯听，有空可以看看)

# Jupyter的运行结果与本地的运行结果不符

于是我花了近1个小时的时间排查问题，我的内心是崩溃的!!!





问题起源于，我需要调用PIL来将一个numpy像素矩阵保存为图片

```python
import matplotlib
matplotlib.image.imsave(img_path,img.numpy(),format='jpg')
```



就是这么简单的两行代码，竟然报错，说

```
jpg is an unknown file extension
错误定位在PIL/image.py的save函数中
```



WTF, 难道大名鼎鼎的PIL居然不支持jpg。

当然这是不可能的，我在终端查看了一下Image库的文件位置以及其save函数中使用的字典

```python
import platform
print(platform.python_version())
from PIL import Image
print(Image.__file__)
Image.init()
Image.SAVE.keys()
```

![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/terminal.png)



得到的结果很正常，包含了'jpg'

然后我在notebook中使用相同的函数测试，发现了问题，两个输出竟然不一样

![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/ipythonError.png)

最后发现问题处在notebook kernel的python对应的库位置和终端不一样，且save字典也不同，ipython中save字典为空，难怪说unknown file extension



我把`~/.local/share/jupyter/kernels/python3` 中的kernel.json中的python位置改成与终端相同，重启notebook，问题解决



后记:其实原本我的kernel.json配置的python位置是与我终端中使用的一样，只不过名字不同，为什么使用的包的位置不一样我还没有细查。


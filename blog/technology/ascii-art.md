## 字符画

今日依旧无事，不想搞毕设。

无聊的人想法多，今日就想到把一只 Super Mario 在终端中输出。

<img src="https://blog-static.cnblogs.com/files/sinkinben/Mario.gif" width="20%"></img>

具体做法十分「老土」，就是玩字符画那一套，但我这次想把这个字符串输出成彩色的。

## 准备工作

第一步当然是把图片转换为 24 位的位图，即 `bmp` 格式的图片，使用 Windows 自带的画图工具即可。

> **Aside**
> 之所以叫 24 位图，是因为在这种格式的图片中，一个像素由三个整数 (R, G, B) 表示，每个整数均为 8 bit 的整型。R 是 Red，G 是 Green，B 是 Blue，光学三原色是也。

这样使用合适的库打开某个图片，访问 `image[i][j]` 就可以获得一个三元组 (R, G, B) ，后面的事情就是对这些三元组进行操作输出到终端。也就是说，一个彩色图片可以等价于一个三维数组 `image[m][n][3]` 。

打开图片

```python
from PIL import Image
image = Image.open(os.sys.argv[1])
image = image.resize((int(80), int(80)), Image.ANTIALIAS)
# 可以通过 resize 调整高度和宽度
```

获取一个像素点

```python
image.getpixel(i,j)
```

预处理为可操作的 `list` 类型

```python
rgb_data = parse_image(image)
def parse_image(image: Image):
    rgb_tuple_list = list()
    width, height = image.size
    for j in range(height):
        l = list()
        for i in range(width):
            l.append(image.getpixel((i, j)))
        rgb_tuple_list.append(list(l))
    return rgb_tuple_list
```



## 终端带颜色输出

[参考这篇文章](https://www.cnblogs.com/easypython/p/9084426.html) 。

终端字符颜色实际上是通过转移字符序列来控制的。也就是说，我们在需要输出的字符串前面加入特定的 ASCII 序列就能够改变字体的颜色。

输出有以下 3 种控制方式：

+ 显示方式：默认值 (0)，高亮 (1)，下划线 (4)，闪烁 (5)，反显 (7)
+ 前景颜色：即字体颜色。红色 (31)，绿色 (32)，黄色 (33)，蓝色 (34)，洋红色 (35)，青色 (36)，白色 (37)
+ 背景颜色：黑色 (40)，红色 (41)，绿色 (42)，黄色 (43)，蓝色 (44)，洋红色 (45)，青色 (46)，灰白色 (47)

控制 ASCII 序列为 `format_str = '\033[{};{};{}m'`，例如下面的 python 代码可以输出🌈颜色：

```python
format_str = '\033[{};{};{}m'
ctl = [0, 1, 4, 5, 7]
for i in ctl:
    for j in range(31, 37 + 1):
        for k in range(40, 47 + 1):
            print(format_str.format(i, j, k) + 'sinkinben', end=' ')
    print('')
```



## 灰度图字符画

原本一个有颜色的像素点用 `(r,g,b)` 三个 8 位整型数值表示，灰度图就是把 `(r,g,b)` 转换为一个代表黑白深浅的数值，这样 `image[m][n]` 一个整型二维数组可以表示一个黑白的图片。

转换函数是一个固定的公式：

```python
def gray_val(r, g, b):
    return int(0.2126 * r + 0.7152 * g + 0.0722 * b)
```

但是，这仍然是一个图片，只不过是黑白的，我们无法在普通的终端输出。因此需要把灰度值映射为一个字符，这样就能做出网上常见的字符画。

```python
table = list("@W#$%0OEXC[(/?=^~_.` ")
# table = list("MNHQ$OC67)oa+>!:+. ")
def get_char(r, g, b):
    step = int(256 / len(table)) + 1
    return table[int(gray_val(r, g, b) / step)]
```

灰度值的范围是 $[0,255]$ ，相邻的灰度值呈现的灰度在视觉上是相近的，因此我们就用一个字符 `table[i]` 来表示某个区间的灰度。为什么代码是这么写？举个例子说明。假设灰度值的范围是 $[0,16]$，使用四个字符 `table = '#@*O'` 来表示。也就是说：

```bash
[0, 3]   => table[0]
[4, 7]   => table[1]
[8, 11]  => table[2]
[12, 15] => table[3]
```

在这里灰度值映射得到的字符为 `table[gray_val / 4] ` ，4 是区间的长度，表示一个字符表示灰度值的个数。

`table` 可以根据输出的字体手动修改，这是影响「字符画」美观的主要因素之一（另外一个因素是宽度和高度的比值，因为终端的字体都是长方形的，如果不调整，输出的字符画也是长不拉几的）。

输出纯字符画代码：

```python
def gray_ascii_picture(rgb_data: list):
    ascii_pic = ''
    for row in rgb_data:
        for t in row:
            ascii_pic += get_char(*t) * 3
        ascii_pic += '\n'
    return ascii_pic
```

 `ascii_pic += get_char(*t) * 3` 表示用 3 个字符表示一个像素点，这是调整宽度的一个技巧。

## 上色字符画

首先我们解决一个问题，获取终端颜色的 RGB 表示，这里使用的是 `webcolors` 这个库：

```python
color_list = ['black', 'red', 'green', 'yellow', 'blue',
              'purple', 'skyblue']
color_dict = dict()
for i in range(len(color_list)):
    t = tuple(webcolors.name_to_rgb(color_list[i]))
    color_dict[t] = int(i)
# color_dict is {(0, 0, 0): 0, (255, 0, 0): 1, (0, 128, 0): 2, (255, 255, 0): 3, (0, 0, 255): 4, (128, 0, 128): 5, (135, 206, 235): 6}
```

图片中的颜色数目是远多于终端中可输出的颜色，因此我们需要用 8 种终端颜色来表示所有的 (R, G, B) 颜色，这里采取的策略是，从终端颜色中挑选一个几何距离最近的颜色：

```python
# rgb = get_closest_rgb(terminal_colors=color_dict.keys(), rgb=tuple(r,g,b))
def get_closest_rgb(terminal_colors, rgb: tuple):
    min_val = 255 * 255 * 3
    result = None
    for t in terminal_colors:
        val = int(rgb[0] - t[0]) ** 2 + int(rgb[1] - t[1]) ** 2 + int(rgb[2] - t[2])
        if val < min_val:
            min_val = val
            result = t
    return result
```

最后对每一个像素处理，通过格式化字符串输出一个「色块」。

```python
def colorful_ascii_picture(rgb_data: list):
    format_str = '\033[{};{};40m{}\033[0m'
    color_list = ['black', 'red', 'green', 'yellow', 'blue',
                  'purple', 'skyblue']
    color_dict = dict()
    for i in range(len(color_list)):
        t = tuple(webcolors.name_to_rgb(color_list[i]))
        color_dict[t] = int(i)

    print('')
    for row in rgb_data:
        line = ''
        for t in row:
            rgb = get_closest_rgb(terminal_colors=color_dict.keys(), rgb=t)
            icolor = color_dict[rgb]
            print(format_str.format(1, icolor+30, get_char(*t)), end='')
        print('')
    return
```



## 效果图

+ 黑白 Doraemon ：使用一个字符和一个空格来表示一个像素，在记事本中缩放查看的效果
  <img src="https://images.cnblogs.com/cnblogs_com/sinkinben/1648063/o_200308092230dora-gray.jpg" width="30%"/>
  
  
  
+ 彩色 Doraemon ：使用 2 个字符表示一个像素，很不幸蓝色映射为绿色了😅，终端字体调整为 1 的效果
  <img src="https://images.cnblogs.com/cnblogs_com/sinkinben/1648063/o_200308092703dora-colorful.jpg" width="45%"></img>
  
+ 彩色 Sun Xiaochuan：图片缩放 200 × 200，2 个字符表示 1 个像素
  <img src="https://images.cnblogs.com/cnblogs_com/sinkinben/1648063/o_200308093326sun-colorful.jpg" width="50%"></img>

+ 彩色皮卡丘，背景是 Windows Terminal 自带的 Acrylic 效果，如果在纯黑色背景的终端，效果应该更好，下次用 Ubuntu 试试
  <img src="https://images.cnblogs.com/cnblogs_com/sinkinben/1648063/o_200308094023pikaqiu-colorful.jpg" width="50%"></img>

+ 彩色 Mario
  <img src="https://images.cnblogs.com/cnblogs_com/sinkinben/1648063/o_200308095923mario-colorful.jpg"  width="40%"></img>

## 完整代码

Usage: python xxx.bmp

```python
from PIL import Image
import numpy
import os
import matplotlib.pyplot as pyplot
import webcolors

table = list("@W#$%0OEXC[(/?=^~_.` ")
# table = list("MNHQ$OC67)oa+>!:+. ")

kernel_size = 2
merge_kernel = [[1 for i in range(kernel_size)] for j in range(kernel_size)]


def gray_val(r, g, b):
    return int(0.2126 * r + 0.7152 * g + 0.0722 * b)


def get_char(r, g, b):
    step = int(256 / len(table)) + 1
    return table[int(gray_val(r, g, b) / step)]


def parse_image(image: Image):
    rgb_tuple_list = list()
    width, height = image.size
    for j in range(height):
        l = list()
        for i in range(width):
            l.append(image.getpixel((i, j)))
        rgb_tuple_list.append(list(l))
    return rgb_tuple_list


def show_in_gui(rgb_data: list):
    # 在 pyplot 中显示图片
    pyplot.subplot()
    pyplot.imshow(rgb_data)
    pyplot.show()


def gray_ascii_picture(rgb_data: list):
    ascii_pic = ''
    for row in rgb_data:
        for t in row:
            ascii_pic += get_char(*t) + ' '
        ascii_pic += '\n'
    return ascii_pic


def get_closest_rgb(terminal_colors, rgb: tuple):
    min_val = 255 * 255 * 3
    result = None
    for t in terminal_colors:
        val = int(rgb[0] - t[0]) ** 2 + \
            int(rgb[1] - t[1]) ** 2 + int(rgb[2] - t[2])
        if val < min_val:
            min_val = val
            result = t
    return result


def colorful_ascii_picture(rgb_data: list):
    format_str = '\033[{};{};40m{}\033[0m'
    color_list = ['black', 'red', 'green', 'yellow', 'blue',
                  'purple', 'skyblue']
    color_dict = dict()
    for i in range(len(color_list)):
        t = tuple(webcolors.name_to_rgb(color_list[i]))
        color_dict[t] = int(i)

    print('')
    for row in rgb_data:
        line = ''
        for t in row:
            rgb = get_closest_rgb(terminal_colors=color_dict.keys(), rgb=t)
            icolor = color_dict[rgb]
            # print(icolor, end=' ')
            print(format_str.format(1, icolor+30, get_char(*t))*2, end='')
        print('')
    return


if __name__ == '__main__':
    image = Image.open(os.sys.argv[1])
    image = image.resize((int(200), int(150)), Image.ANTIALIAS)
    rgb_data = parse_image(image)

    # preview
    # show_in_gui(rgb_data)

    # print gray picture in terminal
    # print(gray_ascii_picture(rgb_data))

    colorful_ascii_picture(rgb_data)

```


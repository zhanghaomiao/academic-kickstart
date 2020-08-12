---
title: 用 PyQT 创建一个识别公式的应用
date: 2020-03-22
tags:
    - computer
    - qt
    - python
categories:
    - Learn
summary: 给定识别图像公式的api, 返回需要的数学公式
---

在科研论文中，经常需要输入LaTeX公式，这是一件很令人头疼的事，因为公式输入很复杂且容易出错。MathPix这个软件简化了这个过程，可以直接通过图片识别出 LaTeX 公式。以前这个软件是免费的, 后来开始收费且价格不菲. 不过推出了 API 调用方式，一个月3000条免费，后面超出的价格也不贵。

为了更好的使用这个API，一键操作，仿照以前的 MathPix界面，用 pyqt 写了一个简略的界面，不过也可以实现基本功能。

## IDE

用的是 PyCharm, 不像 Qt 有 QtCreater, python 版本的 pyqt 没找到官方的 IDE, 因此用的是PyCharm. 在 PyCharm 上加了两个 External Tools:

1. qt designer, 设计 UI 界面
   <img src="/img/pyQT/pyqt_tool.png" width="450px" />
2. pyUIC5, 将 UI file 转为 .py file
   <img src="/img/pyQT/pyuic_tool.png" width="450px" />

## Qt Designer

主要用来设计UI界面，设计好的界面如下
   <img src="/img/pyQT/pyqt_ui.png" width="450px" />

1. 图片显示界面， 可以从剪切板里导入图片，或者是从应用外拖进图片
2. Convert, 触发调用API命令，并将结果显示
3. Qt Web, 一个网页显示界面，在 convert， 生成 LaTeX 代码之后，通过调用 MathJax Api， 显示生成后的 LaTeX 公式， 方便与原始图片进行比较
4. PasteBoard 从剪切板导入图片
5. 生成后的 LaTeX 代码，通过 copy 将文字导入到剪切板， 三个copy 对应三种不同返回格式

用到的对象如下

   <img src="/img/pyQT/object.png" width="450px" />

这里有一点需要注意的是，label 标签对应的类不是 QLabel, 因为如果 QLabel 要支持拖拽功能，需要在 QLabel 下重新定义一个子类，并让子类重写`dragEnterEvent` 和 `dropEvent` 两个函数，这样，需要对 Qt Designer 的 QLabel 进行提升，这里我将提升的类命名为 `ImageArea`.

## 代码

<img src="/img/pyQT/code.png" width="650px" />

* `app_run.py` 主程序，程序代码的入口，定义了整个对象，以及对象之间如何交互
* `customClass.py` 定义了 `ImageArea` 类， 让 `QLabel` 支持拖拽
* `dealWithApi.py`  MathPix 的调用接口
* `qtDesigner.py` 通过 `pyUIC` 将 ui 文件转换为 py 文件
* `showLaTeXImage.py` 将生成的LaTeX 公式文本，用 MathJax 显示

## 演示

<img src="/img/pyQT/moive.gif" width="650px" />

For the source code, see at https://github.com/zhanghaomiao/image_to_latex_app
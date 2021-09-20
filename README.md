# Introduction
原神自动钓鱼AI由[YOLOX](https://github.com/Megvii-BaseDetection/YOLOX), DQN两部分模型组成。使用迁移学习，半监督学习进行训练。
模型也包含一些使用opencv等传统数字图像处理方法实现的不可学习部分。

其中YOLOX用于鱼的定位和类型的识别以及鱼竿落点的定位。DQN用于自适应控制钓鱼过程的点击，让力度落在最佳区域内。

# 安装使用流程
安装python运行环境（解释器），推荐使用 [anaconda](https://www.anaconda.com/products/individual#Downloads).

## python环境配置

打开anaconda prompt(命令行界面)，创建新python环境并激活:
```shell
conda create -n ysfish python=3.6
conda activate ysfish
```

## pyHook安装
安装PyUserInput前需要安装<font color=#66CCFF>**pyHook**</font>适用于win10的版本可以在 [这里](https://www.lfd.uci.edu/~gohlke/pythonlibs/) 找到。

适用于python3.6的文件为：\
64位：[pyHook‑1.5.1‑cp36‑cp36m‑win_amd64.whl](https://download.lfd.uci.edu/pythonlibs/y2rycu7g/cp36/pyHook-1.5.1-cp36-cp36m-win_amd64.whl) \
32位：[pyHook‑1.5.1‑cp36‑cp36m‑win32.whl](https://download.lfd.uci.edu/pythonlibs/y2rycu7g/cp36/pyHook-1.5.1-cp36-cp36m-win32.whl)

下载文件后将命令行切换到文件所在目录，执行:
```shell
pip install [文件名].whl
```
例如在64位的python3.6下安装：
```shell
pip install pyHook-1.5.1-cp36-cp36m-win_amd64.whl
```

## 下载工程代码
使用git下载:
```shell
git clone https://github.com/7eu7d7/genshin_auto_fish.git
```
或直接在**github网页端**下载后直接解压。

## 依赖库安装
切换命令行到本工程所在目录:
```shell
cd genshin_auto_fish
```
执行以下命令安装依赖:
```shell
pip install -U pip
python requirements.py
```
如果要使用显卡进行加速需要 [安装CUDA和cudnn](https://zhuanlan.zhihu.com/p/94220564?utm_source=wechat_session&ivk_sa=1024320u) 安装后无视上面的命令用下面这条安装gpu版:
```shell
pip install -U pip
python requirements.py --cuda [cuda版本]
#例如安装的CUDA11.0
python requirements.py --cuda 110
```
可能会有Time out之类的报错，多试几遍，github太卡。

## 安装yolox
切换命令行到本工程所在目录，执行以下命令安装yolox:
```shell
python setup.py develop
```

## 预训练权重下载
下载预训练[权重](https://github.com/7eu7d7/genshin_auto_fish/releases/tag/weights) (.pth文件),
下载后将权重文件放在 <font color=#66CCFF>**工程目录/weights**</font> 下

## 可能的问题
如果后续运行出现<font color=red>ModuleNotFoundError: No module named 'windows'</font>\
把 [anaconda安装路径]/envs/ysfish/Lib/site-packages/pymouse/__init__.py中的\
```python
from windows import PyMouse, PyMouseEvent
```
改为\
```python
from .windows import PyMouse, PyMouseEvent
```

# YOLOX训练工作流程
YOLOX部分因为打标签太累所以用半监督学习。标注少量样本后训练模型生成其余样本伪标签再人工修正，不断迭代提高精度。
样本量较少所以使用迁移学习，在COCO预训练的模型上进行fine-tuning.

训练代码:
```shell
python yolox_tools/train.py -f yolox/exp/yolox_tiny_fish.py -d 1 -b 8 --fp16 -o -c weights/yolox_tiny.pth
```

# DQN训练工作流程
控制力度使用强化学习模型DQN进行训练。两次进度的差值作为reward为模型提供学习方向。模型与环境间交互式学习。

直接在原神内训练耗时较长，太累了。首先制作一个仿真环境，大概模拟钓鱼力度控制操作。在仿真环境内预训练一个模型。
随后将这一模型迁移至原神内，实现域间迁移。

仿真环境预训练代码:
```shell
python train_sim.py
```
原神游戏内训练:
```shell
python train.py
```

# 运行钓鱼AI
显卡加速
```shell
python fishing.py image -f yolox/exp/yolox_tiny_fish.py -c weights/best_tiny3.pth --conf 0.25 --nms 0.45 --tsize 640 --device gpu
```
cpu运行
```shell
python fishing.py image -f yolox/exp/yolox_tiny_fish.py -c weights/best_tiny3.pth --conf 0.25 --nms 0.45 --tsize 640 --device cpu
```
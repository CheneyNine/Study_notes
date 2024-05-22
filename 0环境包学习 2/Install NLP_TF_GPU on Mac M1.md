https://juejin.cn/post/7115210948066951176  

# 安装
## Tensorflow2深度学习环境安装和配置

首先并不需要任何虚拟环境，直接本地安装Python3.10即可，请参见：[一网成擒全端涵盖，在不同架构(Intel x86/Apple m1 silicon)不同开发平台(Win10/Win11/Mac/Ubuntu)上安装配置Python3.10开发环境](https://link.zhihu.com/?target=https%3A//v3u.cn/a_id_200) ，这里不再赘述。 

随后安装Tensorflow本体： 

```
pip3 install tensorflow-macos
```

这里系统会自动选择当前Python版本的Tensorflow安装包： 

```text
➜  ~ pip install tensorflow-macos  
Collecting tensorflow-macos  
  Downloading tensorflow_macos-2.12.0-cp310-cp310-macosx_12_0_arm64.whl (200.8 MB)  
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 200.8/200.8 MB 4.7 MB/s eta 0:00:00
```

安装包大小为200兆左右，如果下载不了，可以选择在pip官网直接下载基于python3.10的安装包：[http://pypi.org/project/tensorflow-macos/#files](https://link.zhihu.com/?target=http%3A//pypi.org/project/tensorflow-macos/%23files)

![](https://pic4.zhimg.com/80/v2-65f4c41557042739c3f8e9e0bd26bef7_1440w.jpg)

然后直接将whl文件拖拽到终端安装即可。

接着安装Tensorflow的GPU插件：tensorflow-metal，它是一个TensorFlow的后端，使用苹果的Metal图形API来加速神经网络计算。Metal是一种高性能图形和计算API，专门为苹果设备的GPU设计，可以实现更快的神经网络计算。使用tensorflow-metal可以显著提高在苹果设备上运行TensorFlow的性能，尤其是在使用Macs M1和M2等基于苹果芯片的设备时。 

```text
pip3 install --user tensorflow-metal
```

注意这里安装命令必须带上--user参数，否则可能会报这个错误： 

```text
Non-OK-status: stream_executor::MultiPlatformManager::RegisterPlatform( std::move(cplatform)) status: INTERNAL: platform is already registered with name: "METAL"
```

安装好之后，在Python终端运行命令: 

```text
import tensorflow  
tensorflow.config.list_physical_devices()
```

程序返回： 

```text
>>> import tensorflow  
>>> tensorflow.config.list_physical_devices()  
[PhysicalDevice(name='/physical_device:CPU:0', device_type='CPU'), PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]
```

可以看到，Tensorflow用于计算的物理设备既支持CPU，也支持GPU，也就是显卡。 

接着，在编写一个完整的测试脚本 test.py: 

```text
import sys  
import tensorflow.keras  
import pandas as pd  
import sklearn as sk  
import scipy as sp  
import tensorflow as tf  
import platform  
print(f"Python Platform: {platform.platform()}")  
print(f"Tensor Flow Version: {tf.__version__}")  
print(f"Keras Version: {tensorflow.keras.__version__}")  
print()  
print(f"Python {sys.version}")  
print(f"Pandas {pd.__version__}")  
print(f"Scikit-Learn {sk.__version__}")  
print(f"SciPy {sp.__version__}")  
gpu = len(tf.config.list_physical_devices('GPU'))>0  
print("GPU is", "available" if gpu else "NOT AVAILABLE")
```

这里打印出深度学习场景下常用的库和版本号： 

```text
➜  chatgpt_async git:(main) ✗ /opt/homebrew/bin/python3.10 "/Users/liuyue/wodfan/work/chatgpt_async/tensof_test.py"  
Python Platform: macOS-13.3.1-arm64-arm-64bit  
Tensor Flow Version: 2.12.0  
Keras Version: 2.12.0  

Python 3.10.9 (main, Dec 15 2022, 17:11:09) [Clang 14.0.0 (clang-1400.0.29.202)]  
Pandas 1.5.2  
Scikit-Learn 1.2.0  
SciPy 1.10.0  
GPU is available
```

一望而知，在最新的macOS-13.3.1系统中，基于Python3.10.9玩儿Tensorflow2.1没有任何问题。 

至此，Tensorflow2就配置好了。

## 个人安装

```python
import sys  
import keras  
import pandas as pd  
import sklearn as sk  
import scipy as sp  
import tensorflow as tf  
import platform  
print(f"Python Platform: {platform.platform()}")  
print(f"Tensor Flow Version: {tf.__version__}")  
print(f"Keras Version: {keras.__version__}")  
print()  
print(f"Python {sys.version}")  
print(f"Pandas {pd.__version__}")  
print(f"Scikit-Learn {sk.__version__}")  
print(f"SciPy {sp.__version__}")  
gpu = len(tf.config.list_physical_devices('GPU'))>0  
print("GPU is", "available" if gpu else "NOT AVAILABLE")
```

最后结果如下：

```text
Python Platform: macOS-14.1.2-arm64-arm-64bit
Tensor Flow Version: 2.13.0
Keras Version: 2.13.1

Python 3.8.18 (default, Sep 11 2023, 08:17:16) 
[Clang 14.0.6 ]
Pandas 2.0.3
Scikit-Learn 1.3.2
SciPy 1.10.1
GPU is available
```
# Keras的导入

 目前无论是中文还是国外网站对于如何正确的导入keras，如何从tensorflow中导入keras，如何在pycharm中从tensorflow里导入keras，这几个问题都众说纷纭，往往是互相借鉴给出一个可用的解决方法，但没有更进一步的解释了。常见因为keras导入引发的问题有以下几个：

1. `from tensorflow import keras`: pycharm中使用keras相关的包没有自动补全
2. `from tensorflow.keras.layers import Conv2D`: pycharm中如此导入会发生`Cannot find reference 'keras' in '__init__.py | __init__.py'` 问题。

  
## 解决方法

 首先给出这些问题的**解决方法**：

1. 使用如下方式导入keras：

`from tensorflow.python import keras`

2. 不从tensorflow里导入keras: `import keras`
    
3. 不导入keras，改用`tf.keras.xxx`来使用keras的相关函数；
    

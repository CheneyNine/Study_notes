# 基于Python3.8版本的pytorch与transformers安装



最近开始接触NLP相关的研发项目，又是一次环境安装各种踩坑环节，记录一下；

1、anaconda创建虚拟环境，我这里选择的是安装python 3.8

```text
conda create -n transformers_pyenv38 python=3.8
```

随后，激活环境

```text
activate transformers_pyenv38 
```

2、安装transformers及其他包，这里如果安装得比较慢，可以手动切换pip源[[1 introduction to HCI & ID]](https://zhuanlan.zhihu.com/p/625201188#ref_1)

```text
pip install transformers -i https://pypi.tuna.tsinghua.edu.cn/simple //可切换其他pip源
pip install datasets
pip install nltk
conda install pytorch tensorflow //安装pytorch跟tensorflow，一起安装避免这两个包的版本兼容问题
```

这里列举国内几个名气比较大的源，建议优先使用清华大学的，比较稳定：

- 清华大学 [https://pypi.tuna.tsinghua.edu.cn/simple/](https://link.zhihu.com/?target=https%3A//pypi.tuna.tsinghua.edu.cn/simple/)
- 豆瓣 [http://pypi.douban.com/simple/](https://link.zhihu.com/?target=http%3A//pypi.douban.com/simple/)
- 阿里云 [http://mirrors.aliyun.com/pypi/simple/](https://link.zhihu.com/?target=http%3A//mirrors.aliyun.com/pypi/simple/)
- 中国科技大学 [https://pypi.mirrors.ustc.edu.cn/simple/](https://link.zhihu.com/?target=https%3A//pypi.mirrors.ustc.edu.cn/simple/)
- 中国科学技术大学 [http://pypi.mirrors.ustc.edu.cn/simple/](https://link.zhihu.com/?target=http%3A//pypi.mirrors.ustc.edu.cn/simple/)

3、安装transformers及其他包，直接指定版本安装所有的包

```text
pip install tensorflow==2.7 keras==2.7 torch==1.13 protobuf==3.19.5 numpy==1.19.5  transformers  datasets evaluate scikit-learn torchvision tensorflow-dataset pyspark==3.2 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

4、根据huggingface的一个中文Text Classification任务[[2]](https://zhuanlan.zhihu.com/p/625201188#ref_2)作为例子测试下环境是否欧克？

```text
from transformers import AutoModelForSequenceClassification,AutoTokenizer,pipeline
model = AutoModelForSequenceClassification.from_pretrained('uer/roberta-base-finetuned-chinanews-chinese')
tokenizer = AutoTokenizer.from_pretrained('uer/roberta-base-finetuned-chinanews-chinese')
text_classification = pipeline('sentiment-analysis', model=model, tokenizer=tokenizer)
text_classification("北京上个月召开了两会")
```

![](https://pic3.zhimg.com/80/v2-fa5109315ab6e44d739d6f171cd258ba_1440w.jpg)

5、为了避免日后重新安装出现版本升级后带来的问题，将各个包pip/conda 导入导出[[3]](https://zhuanlan.zhihu.com/p/625201188#ref_3)为transformersenv_requirement.txt做备份

```text
pip list --format=freeze> transformersenv_requirements1.txt
conda list -e > transformersenv_requirement2.txt
```

复现可以通过如下操作进行

```text
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple -r transformersenv_requirements1.txt
conda install --yes --file transformersenv_requirement2.txt
```

6、这里披露下重要依赖包的版本，以防万一

![](https://pic1.zhimg.com/80/v2-642794495a0287714c02dad28ca3d7b4_1440w.webp)

transformersenv_requirement

7、NLP项目浅尝实践[[4 CONCEPTUALIZING INTERACTION DESIGN]](https://zhuanlan.zhihu.com/p/625201188#ref_4)

试着运行大佬们开源的项目《CCKS2021中文nlp地址相关性任务》，合格成为一名环境配置调度跑起来的工具人哈哈~跑的太慢五一要出去玩就kill掉了，读者可以自行尝试。

![](https://pic2.zhimg.com/80/v2-ee25405f637d54ba23b0796ac6d98c05_1440w.webp)

## 参考

1. [^](https://zhuanlan.zhihu.com/p/625201188#ref_1_0)手动指定pip源 [https://blog.csdn.net/weixin_39334709/article/details/105265951](https://blog.csdn.net/weixin_39334709/article/details/105265951)
2. [^](https://zhuanlan.zhihu.com/p/625201188#ref_2_0)huggingface中文示例 [https://huggingface.co/uer/roberta-base-finetuned-jd-binary-chinese](https://huggingface.co/uer/roberta-base-finetuned-jd-binary-chinese)
3. [^](https://zhuanlan.zhihu.com/p/625201188#ref_3_0)pip/conda 导入导出 [https://blog.csdn.net/weixin_43913261/article/details/124687789](https://blog.csdn.net/weixin_43913261/article/details/124687789)
4. [^](https://zhuanlan.zhihu.com/p/625201188#ref_4_0)cks2021中文nlp地址相关性任务 [https://github.com/xueyouluo/ccks2021-track2-code](https://github.com/xueyouluo/ccks2021-track2-code)
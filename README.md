# PaddleX-for-facemask-detection
this is an example for detecting facemask by PaddleX.
# PaddleX实现口罩检测

基于PaddleX实现目标检测。实现以下任务：

> 1. 配置数据集（数据集选择、数据处理）
> 2. 配置模型并训练

# 一、数据集说明

本项目使用的数据集是：[[AI训练营]目标检测数据集合集](https://aistudio.baidu.com/aistudio/datasetdetail/103743)，中的口罩识别 。

该数据集位于：**data/data103743/objDataset.zip**


```python
# 解压数据集（解压一次即可，请勿重复解压）
!unzip -oq /home/aistudio/data/data103743/objDataset.zip
```

解压完成后，左侧文件夹处会多一个名为**objDataset**的文件夹，该文件夹下有5个子文件夹：

- **barricade**——Gazebo锥桶检测
- **facemask**——口罩检测
- **fire**——火焰检测
- **MidAutumn**——中秋元素检测
- **roadsign_voc**——交通路标检测

每个子文件夹下有2个文件夹，分别存放着图像（**JPEGImages**）和标注文件（**Annotations**），如下所示：


```python
# 查看数据集文件结构
!tree objDataset -L 2
```

    objDataset
    ├── barricade
    │   ├── Annotations
    │   └── JPEGImages
    ├── facemask
    │   ├── Annotations
    │   └── JPEGImages
    ├── fire
    │   ├── Annotations
    │   └── JPEGImages
    ├── MidAutumn
    │   ├── Annotations
    │   └── JPEGImages
    └── roadsign_voc
        ├── Annotations
        └── JPEGImages
    
    15 directories, 0 files


# 二、数据准备

本基线系统使用的数据格式是PascalVOC格式，开发者基于PaddleX开发目标检测模型时，无需对数据格式进行转换，开箱即用。

但为了进行训练，还需要将数据划分为训练集、验证集和测试集。划分之前首先需要**安装PaddleX**。


```python
# 安装PaddleX
!pip install paddlex
```

    Looking in indexes: https://mirror.baidu.com/pypi/simple/
    Collecting paddlex
    [?25l  Downloading https://mirror.baidu.com/pypi/packages/d6/a2/07435f4aa1e51fe22bdf06c95d03bf1b78b7bc6625adbb51e35dc0804cc7/paddlex-1.3.11-py3-none-any.whl (516kB)
    [K     |████████████████████████████████| 522kB 16.0MB/s eta 0:00:01
    [?25hRequirement already satisfied: sklearn in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlex) (0.0)
    Requirement already satisfied: visualdl>=2.0.0 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlex) (2.2.0)
    Collecting pycocotools; platform_system != "Windows" (from paddlex)
      Downloading https://mirror.baidu.com/pypi/packages/de/df/056875d697c45182ed6d2ae21f62015896fdb841906fe48e7268e791c467/pycocotools-2.0.2.tar.gz
    Requirement already satisfied: opencv-python in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlex) (4.1.1.26)
    Collecting paddleslim==1.1.1 (from paddlex)
    [?25l  Downloading https://mirror.baidu.com/pypi/packages/d1/77/e257227bed9a70ff0d35a4a3c4e70ac2d2362c803834c4c52018f7c4b762/paddleslim-1.1.1-py2.py3-none-any.whl (145kB)
    [K     |████████████████████████████████| 153kB 12.1MB/s eta 0:00:01
    [?25hCollecting paddlehub==2.1.0 (from paddlex)
    [?25l  Downloading https://mirror.baidu.com/pypi/packages/7a/29/3bd0ca43c787181e9c22fe44b944b64d7fcb14ce66d3bf4602d9ad2ac76c/paddlehub-2.1.0-py3-none-any.whl (211kB)
    [K     |████████████████████████████████| 215kB 17.2MB/s eta 0:00:01
    [?25hCollecting shapely>=1.7.0 (from paddlex)
    [?25l  Downloading https://mirror.baidu.com/pypi/packages/98/f8/db4d3426a1aba9d5dfcc83ed5a3e2935d2b1deb73d350642931791a61c37/Shapely-1.7.1-cp37-cp37m-manylinux1_x86_64.whl (1.0MB)
    [K     |████████████████████████████████| 1.0MB 30.1MB/s eta 0:00:01
    [?25hRequirement already satisfied: psutil in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlex) (5.7.2)
    Requirement already satisfied: flask-cors in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlex) (3.0.8)
    Collecting xlwt (from paddlex)
    [?25l  Downloading https://mirror.baidu.com/pypi/packages/44/48/def306413b25c3d01753603b1a222a011b8621aed27cd7f89cbc27e6b0f4/xlwt-1.3.0-py2.py3-none-any.whl (99kB)
    [K     |████████████████████████████████| 102kB 12.3MB/s ta 0:00:01
    [?25hRequirement already satisfied: tqdm in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlex) (4.36.1)
    Requirement already satisfied: colorama in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlex) (0.4.4)
    Requirement already satisfied: pyyaml in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlex) (5.1.2)
    Requirement already satisfied: scikit-learn in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from sklearn->paddlex) (0.22.1)
    Requirement already satisfied: matplotlib in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from visualdl>=2.0.0->paddlex) (2.2.3)
    Requirement already satisfied: Pillow>=7.0.0 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from visualdl>=2.0.0->paddlex) (7.1.2)
    Requirement already satisfied: requests in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from visualdl>=2.0.0->paddlex) (2.22.0)
    Requirement already satisfied: protobuf>=3.11.0 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from visualdl>=2.0.0->paddlex) (3.14.0)
    Requirement already satisfied: numpy in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from visualdl>=2.0.0->paddlex) (1.16.4)
    Requirement already satisfied: flask>=1.1.1 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from visualdl>=2.0.0->paddlex) (1.1.1)
    Requirement already satisfied: Flask-Babel>=1.0.0 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from visualdl>=2.0.0->paddlex) (1.0.0)
    Requirement already satisfied: pre-commit in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from visualdl>=2.0.0->paddlex) (1.21.0)
    Requirement already satisfied: shellcheck-py in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from visualdl>=2.0.0->paddlex) (0.7.1.1)
    Requirement already satisfied: flake8>=3.7.9 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from visualdl>=2.0.0->paddlex) (3.8.2)
    Requirement already satisfied: pandas in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from visualdl>=2.0.0->paddlex) (1.1.5)
    Requirement already satisfied: six>=1.14.0 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from visualdl>=2.0.0->paddlex) (1.15.0)
    Requirement already satisfied: bce-python-sdk in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from visualdl>=2.0.0->paddlex) (0.8.53)
    Requirement already satisfied: setuptools>=18.0 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from pycocotools; platform_system != "Windows"->paddlex) (41.4.0)
    Requirement already satisfied: cython>=0.27.3 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from pycocotools; platform_system != "Windows"->paddlex) (0.29)
    Requirement already satisfied: pyzmq in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddleslim==1.1.1->paddlex) (18.1.1)
    Requirement already satisfied: filelock in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlehub==2.1.0->paddlex) (3.0.12)
    Requirement already satisfied: gunicorn>=19.10.0; sys_platform != "win32" in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlehub==2.1.0->paddlex) (20.0.4)
    Requirement already satisfied: gitpython in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlehub==2.1.0->paddlex) (3.1.14)
    Collecting paddle2onnx>=0.5.1 (from paddlehub==2.1.0->paddlex)
    [?25l  Downloading https://mirror.baidu.com/pypi/packages/37/80/aa6134b5f36aea45dc1b363e7af941dccabe4d7e167ac391ff046f34baf1/paddle2onnx-0.7-py3-none-any.whl (94kB)
    [K     |████████████████████████████████| 102kB 15.2MB/s ta 0:00:01
    [?25hRequirement already satisfied: rarfile in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlehub==2.1.0->paddlex) (3.1)
    Requirement already satisfied: paddlenlp>=2.0.0rc5 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlehub==2.1.0->paddlex) (2.0.7)
    Requirement already satisfied: colorlog in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlehub==2.1.0->paddlex) (4.1.0)
    Requirement already satisfied: packaging in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlehub==2.1.0->paddlex) (20.9)
    Requirement already satisfied: easydict in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlehub==2.1.0->paddlex) (1.9)
    Requirement already satisfied: joblib>=0.11 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from scikit-learn->sklearn->paddlex) (0.14.1)
    Requirement already satisfied: scipy>=0.17.0 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from scikit-learn->sklearn->paddlex) (1.3.0)
    Requirement already satisfied: pytz in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from matplotlib->visualdl>=2.0.0->paddlex) (2019.3)
    Requirement already satisfied: python-dateutil>=2.1 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from matplotlib->visualdl>=2.0.0->paddlex) (2.8.0)
    Requirement already satisfied: pyparsing!=2.0.4,!=2.1.2,!=2.1.6,>=2.0.1 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from matplotlib->visualdl>=2.0.0->paddlex) (2.4.2)
    Requirement already satisfied: cycler>=0.10 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from matplotlib->visualdl>=2.0.0->paddlex) (0.10.0)
    Requirement already satisfied: kiwisolver>=1.0.1 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from matplotlib->visualdl>=2.0.0->paddlex) (1.1.0)
    Requirement already satisfied: urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from requests->visualdl>=2.0.0->paddlex) (1.25.6)
    Requirement already satisfied: certifi>=2017.4.17 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from requests->visualdl>=2.0.0->paddlex) (2019.9.11)
    Requirement already satisfied: chardet<3.1.0,>=3.0.2 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from requests->visualdl>=2.0.0->paddlex) (3.0.4)
    Requirement already satisfied: idna<2.9,>=2.5 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from requests->visualdl>=2.0.0->paddlex) (2.8)
    Requirement already satisfied: Werkzeug>=0.15 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from flask>=1.1.1->visualdl>=2.0.0->paddlex) (0.16.0)
    Requirement already satisfied: Jinja2>=2.10.1 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from flask>=1.1.1->visualdl>=2.0.0->paddlex) (2.10.3)
    Requirement already satisfied: itsdangerous>=0.24 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from flask>=1.1.1->visualdl>=2.0.0->paddlex) (1.1.0)
    Requirement already satisfied: click>=5.1 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from flask>=1.1.1->visualdl>=2.0.0->paddlex) (7.0)
    Requirement already satisfied: Babel>=2.3 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from Flask-Babel>=1.0.0->visualdl>=2.0.0->paddlex) (2.8.0)
    Requirement already satisfied: nodeenv>=0.11.1 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from pre-commit->visualdl>=2.0.0->paddlex) (1.3.4)
    Requirement already satisfied: identify>=1.0.0 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from pre-commit->visualdl>=2.0.0->paddlex) (1.4.10)
    Requirement already satisfied: cfgv>=2.0.0 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from pre-commit->visualdl>=2.0.0->paddlex) (2.0.1)
    Requirement already satisfied: importlib-metadata; python_version < "3.8" in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from pre-commit->visualdl>=2.0.0->paddlex) (0.23)
    Requirement already satisfied: virtualenv>=15.2 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from pre-commit->visualdl>=2.0.0->paddlex) (16.7.9)
    Requirement already satisfied: toml in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from pre-commit->visualdl>=2.0.0->paddlex) (0.10.0)
    Requirement already satisfied: aspy.yaml in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from pre-commit->visualdl>=2.0.0->paddlex) (1.3.0)
    Requirement already satisfied: pyflakes<2.3.0,>=2.2.0 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from flake8>=3.7.9->visualdl>=2.0.0->paddlex) (2.2.0)
    Requirement already satisfied: pycodestyle<2.7.0,>=2.6.0a1 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from flake8>=3.7.9->visualdl>=2.0.0->paddlex) (2.6.0)
    Requirement already satisfied: mccabe<0.7.0,>=0.6.0 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from flake8>=3.7.9->visualdl>=2.0.0->paddlex) (0.6.1)
    Requirement already satisfied: future>=0.6.0 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from bce-python-sdk->visualdl>=2.0.0->paddlex) (0.18.0)
    Requirement already satisfied: pycryptodome>=3.8.0 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from bce-python-sdk->visualdl>=2.0.0->paddlex) (3.9.9)
    Requirement already satisfied: gitdb<5,>=4.0.1 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from gitpython->paddlehub==2.1.0->paddlex) (4.0.5)
    Requirement already satisfied: multiprocess in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlenlp>=2.0.0rc5->paddlehub==2.1.0->paddlex) (0.70.11.1)
    Requirement already satisfied: seqeval in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlenlp>=2.0.0rc5->paddlehub==2.1.0->paddlex) (1.2.2)
    Requirement already satisfied: h5py in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlenlp>=2.0.0rc5->paddlehub==2.1.0->paddlex) (2.9.0)
    Requirement already satisfied: jieba in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from paddlenlp>=2.0.0rc5->paddlehub==2.1.0->paddlex) (0.42.1)
    Requirement already satisfied: MarkupSafe>=0.23 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from Jinja2>=2.10.1->flask>=1.1.1->visualdl>=2.0.0->paddlex) (1.1.1)
    Requirement already satisfied: zipp>=0.5 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from importlib-metadata; python_version < "3.8"->pre-commit->visualdl>=2.0.0->paddlex) (0.6.0)
    Requirement already satisfied: smmap<4,>=3.0.1 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from gitdb<5,>=4.0.1->gitpython->paddlehub==2.1.0->paddlex) (3.0.5)
    Requirement already satisfied: dill>=0.3.3 in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from multiprocess->paddlenlp>=2.0.0rc5->paddlehub==2.1.0->paddlex) (0.3.3)
    Requirement already satisfied: more-itertools in /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages (from zipp>=0.5->importlib-metadata; python_version < "3.8"->pre-commit->visualdl>=2.0.0->paddlex) (7.2.0)
    Building wheels for collected packages: pycocotools
      Building wheel for pycocotools (setup.py) ... [?25ldone
    [?25h  Created wheel for pycocotools: filename=pycocotools-2.0.2-cp37-cp37m-linux_x86_64.whl size=278364 sha256=45ada9d078ed17f4c48317cc5dfd38d3a2bddf3eaef0d3427e1120a4448d340e
      Stored in directory: /home/aistudio/.cache/pip/wheels/fb/44/67/8baa69040569b1edbd7776ec6f82c387663e724908aaa60963
    Successfully built pycocotools
    Installing collected packages: pycocotools, paddleslim, paddle2onnx, paddlehub, shapely, xlwt, paddlex
      Found existing installation: paddlehub 2.0.4
        Uninstalling paddlehub-2.0.4:
          Successfully uninstalled paddlehub-2.0.4
    Successfully installed paddle2onnx-0.7 paddlehub-2.1.0 paddleslim-1.1.1 paddlex-1.3.11 pycocotools-2.0.2 shapely-1.7.1 xlwt-1.3.0


使用如下命令即可将数据划分为70%训练集，20%验证集和10%的测试集。


```python
# 划分数据集
!paddlex --split_dataset --format VOC --dataset_dir objDataset/facemask --val_value 0.2 --test_value 0.1
```

    /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages/setuptools/depends.py:2: DeprecationWarning: the imp module is deprecated in favour of importlib; see the module's documentation for alternative uses
      import imp
    Dataset Split Done.[0m
    [0mTrain samples: 598[0m
    [0mEval samples: 170[0m
    [0mTest samples: 85[0m
    [0mSplit files saved in objDataset/facemask[0m
    [0m[0m

划分完成后，该数据集下会生成**labels.txt**, **train_list.txt**, **val_list.txt**和**test_list.txt**，分别存储类别信息，训练样本列表，验证样本列表，测试样本列表。如下图所示：

![image](https://user-images.githubusercontent.com/18440163/129161946-0052984f-2479-4fbf-afdf-9b9e93727c5a.png)


在这里，**你需要将path to dataset部分(objDataset/facemask)替换成你选择的数据集路径**。在左侧文件夹处，将鼠标放到你想选择的数据集文件夹上，会出现三个图标，第一个图标表示复制该文件夹路径，点击即可获得该文件夹路径，用这个路径替换path to dataset即可。

![](https://ai-studio-static-online.cdn.bcebos.com/c28ed88586644f64b34709a592fea0b97ec80470c0e041fd9aa6b8da21c8e283)


# 三、数据预处理

在训练模型之前，对目标检测任务的数据进行操作，从而提升模型效果。可用于数据处理的API有：

- **Normalize**：对图像进行归一化
- **ResizeByShort**：根据图像的短边调整图像大小
- **RandomHorizontalFlip**：以一定的概率对图像进行随机水平翻转
- **RandomDistort**：以一定的概率对图像进行随机像素内容变换

更多关于数据处理的API及使用说明可查看文档：
[https://paddlex.readthedocs.io/zh_CN/release-1.3/apis/transforms/det_transforms.html](https://paddlex.readthedocs.io/zh_CN/release-1.3/apis/transforms/det_transforms.html)


```python
from paddlex.det import transforms

# 定义训练和验证时的transforms
# API说明 https://paddlex.readthedocs.io/zh_CN/develop/apis/transforms/det_transforms.html
train_transforms = transforms.Compose([
    # 此处需要补充图像预处理代码
    transforms.Normalize(),
])

eval_transforms = transforms.Compose([
    # 此处需要补充图像预处理代码
    transforms.Normalize(),
])
```

读取PascalVOC格式的检测数据集，并对样本进行相应的处理。


```python
import paddlex as pdx

# 定义训练和验证所用的数据集
# API说明：https://paddlex.readthedocs.io/zh_CN/develop/apis/datasets.html#paddlex-datasets-vocdetection
train_dataset = pdx.datasets.VOCDetection(
    data_dir='objDataset/facemask',
    file_list='objDataset/facemask/train_list.txt',
    label_list='objDataset/facemask/labels.txt',
    transforms=train_transforms,
    shuffle=True)

eval_dataset = pdx.datasets.VOCDetection(
    data_dir='objDataset/facemask',
    file_list='objDataset/facemask/val_list.txt',
    label_list='objDataset/facemask/labels.txt',
    transforms=eval_transforms)
```

    /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages/matplotlib/__init__.py:107: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated, and in 3.8 it will stop working
      from collections import MutableMapping
    /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages/matplotlib/rcsetup.py:20: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated, and in 3.8 it will stop working
      from collections import Iterable, Mapping
    /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages/matplotlib/colors.py:53: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated, and in 3.8 it will stop working
      from collections import Sized


    2021-08-12 10:41:37 [INFO]	Starting to read file list from dataset...
    2021-08-12 10:41:39 [INFO]	598 samples in file objDataset/facemask/train_list.txt
    creating index...
    index created!
    2021-08-12 10:41:39 [INFO]	Starting to read file list from dataset...
    2021-08-12 10:41:40 [INFO]	170 samples in file objDataset/facemask/val_list.txt
    creating index...
    index created!


需要注意的是：

- **data_dir** (str): 数据集所在的目录路径。
- **file_list** (str): 描述数据集图片文件和对应标注文件的文件路径（文本内每行路径为相对data_dir的相对路径）。
- **label_list** (str): 描述数据集包含的类别信息文件路径。

需要将第二步数据准备时生成的labels.txt, train_list.txt, val_list.txt和test_list.txt配置到以上变量中，如下图所示：

![image](https://user-images.githubusercontent.com/18440163/129162326-9d257a1a-9cf7-4999-8d26-02e6837174c2.png)


# 四、模型训练

PaddleX目前提供了FasterRCNN和YOLOv3两种检测结构，多种backbone模型。本基线系统以骨干网络为MobileNetV1的YOLOv3算法为例。


```python
# 初始化模型
# API说明: https://paddlex.readthedocs.io/zh_CN/develop/apis/models/detection.html#paddlex-det-yolov3

# 此处需要补充目标检测模型代码
model = pdx.det.YOLOv3(num_classes=len(train_dataset.labels), backbone='MobileNetV1')
```


```python
# 模型训练
# API说明: https://paddlex.readthedocs.io/zh_CN/develop/apis/models/detection.html#id1
# 各参数介绍与调整说明：https://paddlex.readthedocs.io/zh_CN/develop/appendix/parameters.html

# 此处需要补充模型训练参数
model.train(
    num_epochs=200,
    train_dataset=train_dataset,
    train_batch_size=8,
    eval_dataset=eval_dataset,
    learning_rate=0.000125,
    lr_decay_epochs=[210, 240],
    save_dir='output/yolov3_mobilenetv1')
```

    2021-08-12 10:42:30 [INFO]	Downloading MobileNetV1_pretrained.tar from http://paddle-imagenet-models-name.bj.bcebos.com/MobileNetV1_pretrained.tar


      0%|          | 0/16760 [00:00<?, ?KB/s]100%|██████████| 16760/16760 [00:00<00:00, 63984.23KB/s]


    2021-08-12 10:42:30 [INFO]	Decompressing output/yolov3_mobilenetv1/pretrain/MobileNetV1_pretrained.tar...
    2021-08-12 10:42:31 [INFO]	Load pretrain weights from output/yolov3_mobilenetv1/pretrain/MobileNetV1_pretrained.
    2021-08-12 10:42:31 [INFO]	There are 135 varaibles in output/yolov3_mobilenetv1/pretrain/MobileNetV1_pretrained are loaded.






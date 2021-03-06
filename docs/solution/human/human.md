# 人像分割 PP-HumanSeg

本教程基于PaddleSeg提供高精度人像分割模型，从训练到部署的全流程应用指南，以及视频流人像分割、背景替换的实际效果体验。最新发布超轻量级人像分割模型，支持Web端、移动端场景的实时分割。[代码链接](../../../contrib/PP-HumanSeg)

- [人像分割模型](#人像分割模型)
- [安装](#安装)
- [快速体验](#快速体验)
  - [视频流人像分割](#视频流人像分割)
  - [视频流背景替换](#视频流背景替换)
- [训练评估预测](#训练评估预测)
- [模型导出](#模型导出)
- [Web端部署](#Web端部署)
- [移动端部署](#移动端部署)

## 人像分割模型
PP-HumanSeg开放了在大规模人像数据上训练的三个人像模型，满足服务端、移动端、Web端多种使用场景的需求。

| 模型类型 | 适用场景 | Checkpoint | Inference Model |
| --- | --- | --- | ---|
| 高精度模型  | 适用于服务端GPU且背景复杂的人像场景， 模型结构为Deeplabv3+/ResNet50, 输入大小（512， 512） |[pp_humanseg_server_ckpt](https://paddleseg.bj.bcebos.com/dygraph/humanseg/train/deeplabv3p_resnet50_os8_humanseg_512x512_100k.zip) | [pp_humanseg_server_inference](https://paddleseg.bj.bcebos.com/dygraph/humanseg/export/deeplabv3p_resnet50_os8_humanseg_512x512_100k_with_softmax.zip) |
| 轻量级模型 | 适用于移动端或服务端CPU的前置摄像头场景，模型结构为HRNet_w18_samll_v1，输入大小（192， 192）  | [pp_humanseg_mobile_ckpt](https://paddleseg.bj.bcebos.com/dygraph/humanseg/train/fcn_hrnetw18_small_v1_humanseg_192x192.zip) | [pp_humanseg_mobile_inference](https://paddleseg.bj.bcebos.com/dygraph/humanseg/export/fcn_hrnetw18_small_v1_humanseg_192x192_with_softmax.zip) |
| 超轻量级模型 | 适用于Web端或移动端实时分割场景，例如手机自拍、Web视频会议，模型结构为优化的ShuffleNetV2，输入大小（192， 192） | [pp_humanseg_lite_ckpt](https://paddleseg.bj.bcebos.com/dygraph/humanseg/train/shufflenetv2_humanseg_192x192.zip) | [pp_humanseg_lite_inference](https://paddleseg.bj.bcebos.com/dygraph/humanseg/export/shufflenetv2_humanseg_192x192_with_softmax.zip) |

**NOTE:**
* 其中Checkpoint为模型权重，用于Fine-tuning场景。

* Inference Model为预测部署模型，包含`model.pdmodel`计算图结构、`model.pdiparams`模型参数和`deploy.yaml`基础的模型配置信息。

* 其中Inference Model适用于服务端的CPU和GPU预测部署，适用于通过Paddle Lite进行移动端等端侧设备部署。更多Paddle Lite部署说明查看[Paddle Lite文档](https://paddle-lite.readthedocs.io/zh/latest/)


### 计算复杂度和参数量
| 模型类型 | Network | Input Size | FLOPS | Parameters |
|-|-|-|-|-|
| 超轻量级模型 | ShuffleNetV2 | 192x192 | 121272192 (121M) | 137012 (137K) |
| 轻量级模型 | HRNet w18 small v1 | 192x192 | 584182656 (584M) | 1543954 (1.54M) |
| 高精度模型  | Deeplabv3+/ResNet50 | 512x512 | 114148802560 (114G) | 26789874 (26.8M)


<!-- | ShuffleNetV2 | 398x224 | 293631872(294M) | 137012(137K) | -->

## 安装

#### 1. 安装PaddlePaddle

版本要求

* PaddlePaddle >= 2.0.2

* Python >= 3.7+

由于图像分割模型计算开销大，推荐在GPU版本的PaddlePaddle下使用PaddleSeg。推荐安装10.0以上的CUDA环境。安装教程请见[PaddlePaddle官网](https://www.paddlepaddle.org.cn/install/quick?docurl=/documentation/docs/zh/install/pip/linux-pip.html)。


#### 2. 安装PaddleSeg包

```shell
pip install paddleseg
```

#### 3. 下载PaddleSeg仓库

```shell
git clone https://github.com/PaddlePaddle/PaddleSeg
```

## 快速体验
以下所有命令均在`PaddleSeg/contrib/PP-HumanSeg`目录下执行。
```shell
cd PaddleSeg/contrib/PP-HumanSeg
```

### 下载Inference Model

执行以下脚本快速下载所有Inference Model
```bash
python export_model/download_export_model.py
```

### 下载测试数据
我们提供了[supervise.ly](https://supervise.ly/)发布人像分割数据集**Supervisely Persons**, 从中随机抽取一小部分并转化成PaddleSeg可直接加载数据格式，同时提供了手机前置摄像头的人像测试视频`video_test.mp4`。通过运行以下代码进行快速下载：

```bash
python data/download_data.py
```

### 视频流人像分割
结合DIS（Dense Inverse Search-basedmethod）光流算法预测结果与分割结果，改善视频流人像分割。
```bash
# 通过电脑摄像头进行实时分割处理
python video_infer.py \
--cfg export_model/shufflenetv2_humanseg_192x192_with_softmax/deploy.yaml

# 对人像视频进行分割处理
python video_infer.py \
--cfg export_model/deeplabv3p_resnet50_os8_humanseg_512x512_100k_with_softmax/deploy.yaml \
--video_path data/video_test.mp4
```

视频分割结果如下：

<img src="https://paddleseg.bj.bcebos.com/humanseg/data/video_test.gif" width="20%" height="20%"><img src="https://paddleseg.bj.bcebos.com/humanseg/data/result.gif" width="20%" height="20%">

### 视频流背景替换
根据所选背景进行背景替换，背景可以是一张图片，也可以是一段视频。
```bash
# 通过电脑摄像头进行实时背景替换处理, 也可通过'--background_video_path'传入背景视频
python bg_replace.py \
--cfg export_model/shufflenetv2_humanseg_192x192_with_softmax/deploy.yaml \
--background_image_path data/background.jpg

# 对人像视频进行背景替换处理, 也可通过'--background_video_path'传入背景视频
python bg_replace.py \
--cfg export_model/deeplabv3p_resnet50_os8_humanseg_512x512_100k_with_softmax/deploy.yaml \
--background_image_path data/background.jpg \
--video_path data/video_test.mp4

# 对单张图像进行背景替换
python bg_replace.py \
--cfg export_model/deeplabv3p_resnet50_os8_humanseg_512x512_100k_with_softmax/deploy.yaml \
--background_image_path data/background.jpg \
--image_path data/human_image.jpg

```

背景替换结果如下：

<img src="https://paddleseg.bj.bcebos.com/humanseg/data/video_test.gif" width="20%" height="20%"><img src="https://paddleseg.bj.bcebos.com/humanseg/data/bg_replace.gif" width="20%" height="20%">


**NOTE**:

视频分割处理时间需要几分钟，请耐心等待。

提供的模型适用于手机摄像头竖屏拍摄场景，宽屏效果会略差一些。

## 训练评估预测
### 下载预训练模型

执行以下脚本快速下载所有Checkpoint作为预训练模型
```bash
python pretrained_model/download_pretrained_model.py
```

### 训练
基于上述大规模数据预训练的模型，在抽取的部分[supervise.ly](https://supervise.ly/)数据集上进行Fine-tuning，以HRNet w18 small v1为例，训练命令如下：
```bash
export CUDA_VISIBLE_DEVICES=0 # 设置1张可用的卡
# windows下请执行以下命令
# set CUDA_VISIBLE_DEVICES=0
python train.py \
--config configs/fcn_hrnetw18_small_v1_humanseg_192x192_mini_supervisely.yml \
--save_dir saved_model/fcn_hrnetw18_small_v1_humanseg_192x192_mini_supervisely \
--save_interval 100 --do_eval --use_vdl
```

更多命令行帮助可运行下述命令进行查看：
```bash
python train.py --help
```

### 评估
使用下述命令进行评估
```bash
python val.py \
--config configs/fcn_hrnetw18_small_v1_humanseg_192x192_mini_supervisely.yml \
--model_path saved_model/fcn_hrnetw18_small_v1_humanseg_192x192_mini_supervisely/best_model/model.pdparams
```

### 预测
使用下述命令进行预测， 预测结果默认保存在`./output/result/`文件夹中。
```bash
python predict.py \
--config configs/fcn_hrnetw18_small_v1_humanseg_192x192_mini_supervisely.yml \
--model_path saved_model/fcn_hrnetw18_small_v1_humanseg_192x192_mini_supervisely/best_model/model.pdparams \
--image_path data/human_image.jpg
```

## 模型导出
### 将模型导出为静态图模型

请确保位于PaddleSeg目录下，执行以下脚本：

```shell
export CUDA_VISIBLE_DEVICES=0 # 设置1张可用的卡
# windows下请执行以下命令
# set CUDA_VISIBLE_DEVICES=0
python ../../export.py \
--config configs/fcn_hrnetw18_small_v1_humanseg_192x192_mini_supervisely.yml \
--model_path saved_model/fcn_hrnetw18_small_v1_humanseg_192x192_mini_supervisely/best_model/model.pdparams \
--save_dir export_model/fcn_hrnetw18_small_v1_humanseg_192x192_mini_supervisely_with_softmax \
--without_argmax --with_softmax
```

### 导出脚本参数解释

|参数名|用途|是否必选项|默认值|
|-|-|-|-|
|config|配置文件|是|-|
|save_dir|模型和visualdl日志文件的保存根路径|否|output|
|model_path|预训练模型参数的路径|否|配置文件中指定值|
|with_softmax|在网络末端添加softmax算子。由于PaddleSeg组网默认返回logits，如果想要部署模型获取概率值，可以置为True|否|False|
|without_argmax|是否不在网络末端添加argmax算子。由于PaddleSeg组网默认返回logits，为部署模型可以直接获取预测结果，我们默认在网络末端添加argmax算子|否|False|

### 结果文件

```shell
output
  ├── deploy.yaml            # 部署相关的配置文件
  ├── model.pdiparams        # 静态图模型参数
  ├── model.pdiparams.info   # 参数额外信息，一般无需关注
  └── model.pdmodel          # 静态图模型文件
```

## Web端部署

![image](https://user-images.githubusercontent.com/10822846/118273079-127bf480-b4f6-11eb-84c0-8a0bbc7c7433.png)

## 移动端部署

![image](../../images/human.png)

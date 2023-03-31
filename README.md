<<<<<<< HEAD
# PingPang_Detection_Car
我国现在倡导全民体育，全民健身，智能化体育。乒乓球虽然只是比赛当中的一个小球，但却是我们中国的国球。除了专业的乒乓球运动员，业余爱好者也是常常会在空闲地时间与二三个志同道合的球友去兵乓球馆内进行对练。只是每次长时间的训练结束后，地板上都能看到四处散落的兵乓球，给场地工作人员带来困扰。因此，就需要可自动拾取乒乓球的机器人来节省一些不必要的人力，满足人们的需求。同时，如果能有一种捡球机器人应用于乒乓球赛场上，且能迅速找到球的位置并能够实现自动捡球的功能，将可以帮助选手节省短暂的休息时间，保证比赛的连续性与流畅性。所以，专门针对乒乓球进行自动拾取的捡球机器人便由此应运而生。
=======
### 项目原地址：【AI达人创造营第三期】乒乓球拾捡小车：https://aistudio.baidu.com/aistudio/projectdetail/4589024?sUid=2131908&shared=1&ts=1672822329812

# 一、项目背景

我国现在倡导全民体育，全民健身，智能化体育。乒乓球虽然只是比赛当中的一个小球，但却是我们中国的国球。除了专业的乒乓球运动员，业余爱好者也是常常会在空闲地时间与二三个志同道合的球友去兵乓球馆内进行对练。只是每次长时间的训练结束后，地板上都能看到四处散落的兵乓球，给场地工作人员带来困扰。因此，就需要可自动拾取乒乓球的机器人来节省一些不必要的人力，满足人们的需求。同时，如果能有一种捡球机器人应用于乒乓球赛场上，且能迅速找到球的位置并能够实现自动捡球的功能，将可以帮助选手节省短暂的休息时间，保证比赛的连续性与流畅性。所以，专门针对乒乓球进行自动拾取的捡球机器人便由此应运而生。

# 二、模型准备

## （一）、数据集处理

### 1.解压数据集


```python
# 解压缩数据到指定目录
! tar -xf /home/aistudio/data/data136961/tabletennis_voc.tar -C ~/work/
```

### 2.按比例划分数据集


```python
%cd /home/aistudio
import random
import os
#生成train.txt和val.txt
random.seed(2020)
xml_dir  = '/home/aistudio/work/tabletennis_voc/voc/annotations'#标签文件地址
img_dir = '/home/aistudio/work/tabletennis_voc/voc/images'#图像文件地址
path_list = list()
for img in os.listdir(img_dir):
    img_path = os.path.join(img_dir,img)
    xml_path = os.path.join(xml_dir,img.replace('jpg', 'xml'))
    path_list.append((img_path, xml_path))
random.shuffle(path_list)
ratio = 0.7
train_f = open('/home/aistudio/work/tabletennis_voc/voc/train.txt','w') #生成训练文件
val_f = open('/home/aistudio/work/tabletennis_voc/voc/val.txt' ,'w')#生成验证文件

for i ,content in enumerate(path_list):
    img, xml = content
    text = img + ' ' + xml + '\n'
    if i < len(path_list) * ratio:
        train_f.write(text)
    else:
        val_f.write(text)
train_f.close()
val_f.close()

#生成标签文档
label = ['ball']#设置你想检测的类别
with open('/home/aistudio/work/tabletennis_voc/voc/label_list.txt', 'w') as f:
    for text in label:
        f.write(text+'\n')
```

    /home/aistudio


### 3.数据集查看
源数据格式为VOC格式，存储格式如下：
![](https://ai-studio-static-online.cdn.bcebos.com/a5e2be8dfa2642639366135d410e72406c24f5dc1f3a4ab7a94a7eea44c7a555)


## （二）、环境准备

### 1.PP-PicoDet介绍

![](https://ai-studio-static-online.cdn.bcebos.com/c5867cee2383418e918f84f579b8e8a2a4e1752a01f84803b6663df683178fe6)


PaddleDetection中提出了全新的轻量级系列模型`PP-PicoDet`，在移动端具有卓越的性能，成为全新SOTA轻量级模型。详细的技术细节可以参考我们的[arXiv技术报告](https://arxiv.org/abs/2111.00902)。

PP-PicoDet模型有如下特点：

- 🌟 更高的mAP: 第一个在1M参数量之内`mAP(0.5:0.95)`超越**30+**(输入416像素时)。
- 🚀 更快的预测速度: 网络预测在ARM CPU下可达150FPS。
- 😊 部署友好: 支持PaddleLite/MNN/NCNN/OpenVINO等预测库，支持转出ONNX，提供了C++/Python/Android的demo。
- 😍 先进的算法: 我们在现有SOTA算法中进行了创新, 包括：ESNet, CSP-PAN, SimOTA等等。目前

### 2.数据格式
目前PP-PicoDet支持 **VOC** 和 **COCO** 两种格式，可根据需要选择。

![](https://ai-studio-static-online.cdn.bcebos.com/ba2f037c289f454086aa66484cb539736c19535de15f457289b9313e3a59fcd3)

### 3.基线


| 模型     | 输入尺寸 | mAP<sup>val<br>0.5:0.95 | mAP<sup>val<br>0.5 | 参数量<br><sup>(M) | FLOPS<br><sup>(G) | 预测时延<sup><small>[NCNN](#latency)</small><sup><br><sup>(ms) | 预测时延<sup><small>[Lite](#latency)</small><sup><br><sup>(ms) |  下载  | 配置文件 |
| :-------- | :--------: | :---------------------: | :----------------: | :----------------: | :---------------: | :-----------------------------: | :-----------------------------: | :----------------------------------------: | :--------------------------------------- |
| PicoDet-S |  320*320   |          27.1           |        41.4        |        0.99        |       0.73        |              8.13               |            **6.65**             | [model](https://paddledet.bj.bcebos.com/models/picodet_s_320_coco.pdparams) &#124; [log](https://paddledet.bj.bcebos.com/logs/train_picodet_s_320_coco.log) | [config](https://github.com/PaddlePaddle/PaddleDetection/tree/release/2.3/configs/picodet/picodet_s_320_coco.yml) |
| PicoDet-S |  416*416   |          30.7           |        45.8        |        0.99        |       1.24        |              12.37              |            **9.82**             | [model](https://paddledet.bj.bcebos.com/models/picodet_s_416_coco.pdparams) &#124; [log](https://paddledet.bj.bcebos.com/logs/train_picodet_s_416_coco.log) | [config](https://github.com/PaddlePaddle/PaddleDetection/tree/release/2.3/configs/picodet/picodet_s_416_coco.yml) |
| PicoDet-M |  320*320   |          30.9           |        45.7        |        2.15        |       1.48        |[              ](http://)11.27              |            **9.61**             | [model](https://paddledet.bj.bcebos.com/models/picodet_m_320_coco.pdparams) &#124; [log](https://paddledet.bj.bcebos.com/logs/train_picodet_m_320_coco.log) | [config](https://github.com/PaddlePaddle/PaddleDetection/tree/release/2.3/configs/picodet/picodet_m_320_coco.yml) |
| PicoDet-M |  416*416   |          34.8           |        50.5        |        2.15        |       2.50        |              17.39              |            **15.88**            | [model](https://paddledet.bj.bcebos.com/models/picodet_m_416_coco.pdparams) &#124; [log](https://paddledet.bj.bcebos.com/logs/train_picodet_m_416_coco.log) | [config](https://github.com/PaddlePaddle/PaddleDetection/tree/release/2.3/configs/picodet/picodet_m_416_coco.yml) |
| PicoDet-L |  320*320   |          32.9           |        48.2        |        3.30        |       2.23        |              15.26              |            **13.42**            | [model](https://paddledet.bj.bcebos.com/models/picodet_l_320_coco.pdparams) &#124; [log](https://paddledet.bj.bcebos.com/logs/train_picodet_l_320_coco.log) | [config](https://github.com/PaddlePaddle/PaddleDetection/tree/release/2.3/configs/picodet/picodet_l_320_coco.yml) |
| PicoDet-L |  416*416   |          36.6           |        52.5        |        3.30        |       3.76        |              23.36              |            **21.85**            | [model](https://paddledet.bj.bcebos.com/models/picodet_l_416_coco.pdparams) &#124; [log](https://paddledet.bj.bcebos.com/logs/train_picodet_l_416_coco.log) | [config](https://github.com/PaddlePaddle/PaddleDetection/tree/release/2.3/configs/picodet/picodet_l_416_coco.yml) |
| PicoDet-L |  640*640   |          40.9           |        57.6        |        3.30        |       8.91        |              54.11              |            **50.55**            | [model](https://paddledet.bj.bcebos.com/models/picodet_l_640_coco.pdparams) &#124; [log](https://paddledet.bj.bcebos.com/logs/train_picodet_l_640_coco.log) | [config](https://github.com/PaddlePaddle/PaddleDetection/tree/release/2.3/configs/picodet/picodet_l_640_coco.yml) |

 **更多的配置**

| 模型     | 输入尺寸 | mAP<sup>val<br>0.5:0.95 | mAP<sup>val<br>0.5 | 参数量<br><sup>(M) | FLOPS<br><sup>(G) | 预测时延<sup><small>[NCNN](#latency)</small><sup><br><sup>(ms) | 预测时延<sup><small>[Lite](#latency)</small><sup><br><sup>(ms) |  下载  | 配置文件 |
| :--------------------------- | :--------: | :---------------------: | :----------------: | :----------------: | :---------------: | :-----------------------------: | :-----------------------------: | :----------------------------------------: | :--------------------------------------- |
| PicoDet-Shufflenetv2 1x      |  416*416   |          30.0           |        44.6        |        1.17        |       1.53        |              15.06              |            **10.63**            |      [model](https://paddledet.bj.bcebos.com/models/picodet_shufflenetv2_1x_416_coco.pdparams) &#124; [log](https://paddledet.bj.bcebos.com/logs/train_picodet_shufflenetv2_1x_416_coco.log)      | [config](https://github.com/PaddlePaddle/PaddleDetection/tree/release/2.3/configs/picodet/more_config/picodet_shufflenetv2_1x_416_coco.yml)      |
| PicoDet-MobileNetv3-large 1x |  416*416   |          35.6           |        52.0        |        3.55        |       2.80        |              20.71              |            **17.88**            | [model](https://paddledet.bj.bcebos.com/models/picodet_mobilenetv3_large_1x_416_coco.pdparams) &#124; [log](https://paddledet.bj.bcebos.com/logs/train_picodet_mobilenetv3_large_1x_416_coco.log) | [config](https://github.com/PaddlePaddle/PaddleDetection/tree/release/2.3/configs/picodet/more_config/picodet_mobilenetv3_large_1x_416_coco.yml) |
| PicoDet-LCNet 1.5x           |  416*416   |          36.3           |        52.2        |        3.10        |       3.85        |              21.29              |            **20.8**             |           [model](https://paddledet.bj.bcebos.com/models/picodet_lcnet_1_5x_416_coco.pdparams) &#124; [log](https://paddledet.bj.bcebos.com/logs/train_picodet_lcnet_1_5x_416_coco.log)           | [config](https://github.com/PaddlePaddle/PaddleDetection/tree/release/2.3/configs/picodet/more_config/picodet_lcnet_1_5x_416_coco.yml)           |
| PicoDet-LCNet 1.5x           |  640*640   |          40.6           |        57.4        |        3.10        |       -        |              -              |            -             |           [model](https://paddledet.bj.bcebos.com/models/picodet_lcnet_1_5x_640_coco.pdparams) &#124; [log](https://paddledet.bj.bcebos.com/logs/train_picodet_lcnet_1_5x_640_coco.log)           | [config](https://github.com/PaddlePaddle/PaddleDetection/tree/release/2.3/configs/picodet/more_config/picodet_lcnet_1_5x_640_coco.yml)           |
| PicoDet-R18           |  640*640   |          40.7           |        57.2        |        11.10        |       -        |              -              |            -             |           [model](https://paddledet.bj.bcebos.com/models/picodet_r18_640_coco.pdparams) &#124; [log](https://paddledet.bj.bcebos.com/logs/train_picodet_r18_640_coco.log)           | [config](https://github.com/PaddlePaddle/PaddleDetection/tree/release/2.3/configs/picodet/more_config/picodet_r18_640_coco.yml)           |

<details open>
<summary><b>注意事项:</b></summary>

- <a name="latency">时延测试：</a> 我们所有的模型都在`骁龙865(4xA77+4xA55)` 上测试(4线程，FP16预测)。上面表格中标有`NCNN`的是使用[NCNN](https://github.com/Tencent/ncnn)库测试，标有`Lite`的是使用[Paddle Lite](https://github.com/PaddlePaddle/Paddle-Lite)进行测试。 测试的benchmark脚本来自: [MobileDetBenchmark](https://github.com/JiweiMaster/MobileDetBenchmark)。
- PicoDet在COCO train2017上训练，并且在COCO val2017上进行验证。
- PicoDet使用4卡GPU训练(PicoDet-L-640使用8卡训练)，并且所有的模型都是通过发布的默认配置训练得到。

</details>

  **其他模型的基线**

| 模型     | 输入尺寸 | mAP<sup>val<br>0.5:0.95 | mAP<sup>val<br>0.5 | 参数量<br><sup>(M) | FLOPS<br><sup>(G) | 预测时延<sup><small>[NCNN](#latency)</small><sup><br><sup>(ms) |
| :-------- | :--------: | :---------------------: | :----------------: | :----------------: | :---------------: | :-----------------------------: |
| YOLOv3-Tiny |  416*416   |          16.6           |        33.1      |        8.86        |       5.62        |             25.42               |
| YOLOv4-Tiny |  416*416   |          21.7           |        40.2        |        6.06           |       6.96           |             23.69               |
| PP-YOLO-Tiny |  320*320       |          20.6         |        -              |   1.08             |    0.58             |    6.75                           |  
| PP-YOLO-Tiny |  416*416   |          22.7          |    -               |    1.08               |    1.02             |    10.48                          |  
| Nanodet-M |  320*320      |          20.6            |    -               |    0.95               |    0.72             |    8.71                           |  
| Nanodet-M |  416*416   |          23.5             |    -               |    0.95               |    1.2              |  13.35                          |
| Nanodet-M 1.5x |  416*416   |          26.8        |    -                  | 2.08               |    2.42             |    15.83                          |
| YOLOX-Nano     |  416*416   |          25.8          |    -               |    0.91               |    1.08             |    19.23                          |
| YOLOX-Tiny     |  416*416   |          32.8          |    -               |    5.06               |    6.45             |    32.77                          |
| YOLOv5n |  640*640       |          28.4             |    46.0            |    1.9                |    4.5              |    40.35                          |
| YOLOv5s |  640*640       |          37.2             |    56.0            |    7.2                |    16.5             |    78.05                          |
 

### 4.安装

环境要求
* PaddlePaddle >= 2.1.2
* Python >= 3.5
* PaddleSlim >= 2.1.1
* PaddleLite >= 2.10**


```python
# 克隆PaddleDetection仓库
# 如果已经克隆，则不需要重复运行，可把第3行直接注释，从第6行开始运行
!git clone https://gitee.com/PaddlePaddle/PaddleDetection.git

# 安装其他依赖
%cd PaddleDetection
!pip install -r requirements.txt

# 编译安装paddledet
!python setup.py install
%cd ~
```


```python
# 安装其他依赖
%cd PaddleYOLO
!pip install -r requirements.txt

# 编译安装paddledet
!python setup.py install
%cd ~
```

本次训练我们采用COCO数据集格式，因此我们需要将源数据集转换为COCO格式的数据集，存储格式如下：![](https://ai-studio-static-online.cdn.bcebos.com/4b11b87b4f114a0591d31c3c9b8724cb70687c17ede64ce6b84159af2f4f46df)



```python
#VOC格式数据集转换为COCO格式 使用Dection的脚本转换
#训练文件
%cd ~/PaddleDetection/
! python tools/x2coco.py \
        --dataset_type voc \
        --voc_anno_dir /home/aistudio/work/tabletennis_voc/voc/annotations/ \
        --voc_anno_list /home/aistudio/work/tabletennis_voc/voc/train.txt \
        --voc_label_list /home/aistudio/work/tabletennis_voc/voc/label_list.txt \
        --voc_out_name voc_train.json
```


```python
#测试文件
%cd ~/PaddleDetection/
! python tools/x2coco.py \
        --dataset_type voc \
        --voc_anno_dir /home/aistudio/work/tabletennis_voc/voc/annotations/ \
        --voc_anno_list /home/aistudio/work/tabletennis_voc/voc/train.txt \
        --voc_label_list /home/aistudio/work/tabletennis_voc/voc/label_list.txt \
        --voc_out_name voc_val.json
```


```python
#将转换的json文件移到指定文件夹 我们进行coco格式数据集构建
! pwd
! mkdir /home/aistudio/work/tabletennis_coco
! mv voc_train.json ~/work/tabletennis_coco/annotations
! mv voc_val.json ~/work/tabletennis_coco/annotations
```


```python
#复制图片过来
! cp -r /home/aistudio/work/tabletennis_voc/voc/images/* /home/aistudio/work/tabletennis_coco/images/
```

## **（三）、执行训练**

### 1.模型选择
因为要部署在移动端，且保证速度快和精度高，因此我们选择PaddleDetection提出的全新轻量级系列模型PP-PicoDet，模型有如下特点：

* 更高的mAP: 第一个在1M参数量之内mAP(0.5:0.95)超越30+(输入416像素时)。
* 更快的预测速度: 网络预测在ARM CPU下可达150FPS。
* 部署友好: 支持PaddleLite/MNN/NCNN/OpenVINO等预测库，支持转出ONNX，提供了C++/Python/Android的demo。
* 先进的算法: 我们在现有SOTA算法中进行了创新, 包括：ESNet, CSP-PAN, SimOTA等等。

在此选择PP-PicoDet的VOC数据集训练配置

### 2.配置修改
（1）首先修改PaddleDetection/configs/datasets/coco_detection.yml文件
将里面的数据集配置改为本项目所需的数据集 修改如下：

```
metric: COCO
num_classes: 1

TrainDataset:
  !COCODataSet
    image_dir: images
    anno_path: annotations/voc_train.json
    dataset_dir: /home/aistudio/work/tabletennis_coco
    data_fields: ['image', 'gt_bbox', 'gt_class', 'is_crowd']

EvalDataset:
  !COCODataSet
    image_dir: images
    anno_path: annotations/voc_val.json
    dataset_dir: /home/aistudio/work/tabletennis_coco

TestDataset:
  !ImageFolder
    anno_path: annotations/voc_val.json # also support txt (like VOC's label_list.txt)
    dataset_dir: /home/aistudio/work/tabletennis_coco # if set, anno_path will be 'dataset_dir/anno_path'

```
数据集包含的类别数：num_classes 1 (只识别乒乓球)

包含训练集、验证集、测试集的图片路径image_dir、标注json文件路径anno_path、数据集路径dataset_dir 
路径改为：构建的乒乓球coco数据集路径

（2）然后修改 PaddleDetection/configs/picodet/picodet_m_320_coco_lcnet.yml
修改以下参数(或者不变，使用默认参数)

预训练模型：pretrain_weights

训练超参数：epoch、batch_size、base_lr

[详细配置文件改动和说明](https://github.com/PaddlePaddle/PaddleDetection/blob/release/2.3/docs/tutorials/GETTING_STARTED_cn.md#3-%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E6%94%B9%E5%8A%A8%E5%92%8C%E8%AF%B4%E6%98%8E)。

### 3.模型训练
PaddleDetection提供了单卡/多卡训练模型，满足用户多种训练需求，具体代码如下：


```python
-r output/yolov3_mobilenet_v1_ssld_270e_coco/169#恢复训练
```


```python
PaddleYOLO/configs/yolov5/yolov5_s_300e_coco.yml
```


```python
# 单卡GPU上训练
%cd ~
%cd PaddleYOLO
!export CUDA_VISIBLE_DEVICES=0 #windows和Mac下不需要执行该命令
!python tools/train.py -c configs/yolov5/yolov5_s_300e_coco.yml --eval

# 多卡GPU上训练
# !export CUDA_VISIBLE_DEVICES=0,1,2,3
# !python -m paddle.distributed.launch --gpus 0,1,2,3 tools/train.py \
#             -c configs/picodet/picodet_m_320_coco_lcnet.yml# 模型训练
```

## （四）、模型评估


```python
%cd ~
%cd PaddleYOLO
!python -u tools/eval.py -c  configs/yolov5/yolov5_s_300e_coco.yml  -o weights=output/yolov5_s_300e_coco/best_model.pdparams
```

## （五）、模型预测


```python
#模型预测
%cd ~
%cd Paddleyolo
# 更换"--infer_img"里的图片路径以预测不同的图片
!python tools/infer.py -c configs/yolov5/yolov5_s_300e_coco.yml \
                    --infer_img=/home/aistudio/work/tabletennis_coco/images/29e49881a1b7c05446801a999af99a13.jpg \
                    --output_dir=infer_output/ \
                    --draw_threshold=0.5 \
                    -o weights=/home/aistudio/PaddleDetection/output/yolov5_s_300e_coco/best_model.pdparams \
                    --use_vdl=Ture
```

### 结果展示

 ![](https://ai-studio-static-online.cdn.bcebos.com/545fb173291a42eb9339bc1d80309d65a4884b4c5b6c4c46b955c12118917406)
 

### 导出模型

需要将模型进行导成部署需要的模型格式，执行下面命令：


```python
!export CUDA_VISIBLE_DEVICES=0
%cd ~/PaddleYOLO/
!python tools/export_model.py \
      -c configs/yolov5/yolov5_s_300e_coco.yml \
      -o weights=output/yolov5_s_300e_coco/best_model.pdparams \
      --output_dir=inference_model
```


```python
!tree ./inference_model/yolov5_s_300e_coco
```

    ./inference_model/yolov3_mobilenet_v1_ssld_270e_coco
    ├── infer_cfg.yml
    ├── model.pdiparams
    ├── model.pdiparams.info
    └── model.pdmodel
    
    0 directories, 4 files


预测模型会导出到inference_model/目录下，包括**model.pdmodel、model.pdiparams、model.pdiparams.info和infer_cfg.yml**四个文件，分别表示模型的**网络结构、模型权重、模型权重名称和模型的配置文件**（包括数据预处理参数等）的流程配置文件。

### 静态图预测
在终端输入以下命令进行预测，详细教程请参考Python端预测部署：


```python
!export CUDA_VISIBLE_DEVICES=0
'''
    --model_dir: 上述导出的模型路径
    --image_file：需要测试的图片
    --image_dir：也可以指定要测试的文件夹路径
    --device：运行时的设备，可选择CPU/GPU/XPU，默认为CPU
    --output_dir：可视化结果保存的根目录，默认为output/
'''
!python deploy/python/infer.py \
        --model_dir=./inference_model/yolov5_s_300e_coco \
        --image_file=/home/aistudio/work/tabletennis_coco/images/086aff34ef4859c455ce8c7e2e75a1ba.jpg \
        --device=GPU
```

####  注释，下面代码不必运行！！！！！！！
####  导出模型的第二种写法，前面已经导出，不必再次运行！！！


```python
# 模型导出
!python -u PaddleDetection/tools/export_model.py -c PaddleYOLO/configs/yolov5/yolov5_s_300e_coco.yml --output_dir=./339 \
                              -o weights=PaddleDetection/output/yolov5_s_300e_coco/219.pdparams

```

推理速耗时 14.60ms ，速度相当不错。

## (六)、总结

乒乓球拾小车的模型训练部分就到此为止了，Picodet模型还是很不错的，模型文件不仅小推理速度也很不错，后面将在Jetson Nano板子上进行部署，并制作其他部分。

鸣谢周国源团队提供的原始数据集、李成宇团队的帮助！！！非常感谢

# 三、Jetson Nano部署

## （1）硬件

* 1.Jetson Nano开发板
* 2.扩展板；
* 3.杜邦线若干；
* 4.显示屏；
* 5.网线；
* 6.720P罗技摄像头
* 。。。

## （2）软件

* Ubuntu18.04系统；
* Jetson Nano官方开发套件包（4.6.1）；
* Python3.6；
* PaddlePaddle2.2.3
* 。。。

## （3）Jetson Nano配置

### 1.参考资料
http://t.csdn.cn/ewYkA

### 2.USB摄像头的调试

首先，直接将摄像头的USB接头与JetsonNano连接。
> 在终端输入ls /dev/video*，如果出现/dev/video0则说明摄像头连接成功。用下面这段程序测试一下摄像头是否正常，USB摄像头初始化改为cap = cv2.VideoCapture(0)。


```python
import cv2
import numpy as np

def video_demo():
    print("init")
    capture = cv2.VideoCapture(0)#0为USB摄像头，这里的编号根据JetsonNano返回结果填写，不一定为0，上面又如何调出摄像头的方法。
    while(True):
        ret, frame = capture.read()#摄像头读取,ret为是否成功打开摄像头,true,false。 frame为视频的每一帧图像
        frame = cv2.flip(frame, 1)#摄像头是和人对立的，将图像左右调换回来正常显示。
        print("OK")
        cv2.imshow("video", frame)
        c = cv2.waitKey(50)
        if c == 27:
            break
    capture.release()

video_demo()
cv2.destroyAllWindows()
```

### 3.安装Paddle Inference

首先拉取官方最新版本的Paddle-Inference-demo并解压：


```python
#拉取Paddle-Inference-Demo
!git clone https://github.com/PaddlePaddle/Paddle-Inference-Demo.git
#解压
!unzip -oq /home/aistudio/Paddle-Inference-Demo-master.zip
```

在拉取Paddle Inference的过程中，可能出现网络问题拉取失败，可以去Gitee（码云）找一个仓库，把上述的地址换掉就行。

### 4.安装PaddlePaddle环境

首先查看Jetpack的版本：
> cat /etc/nv_tegra_release

然后根据Jetpack和Python的版本在官网上下载对应的PaddlePaddle

![](https://ai-studio-static-online.cdn.bcebos.com/da1329c703034831a0f537f46ca5c5103e2be4a63bcb4bb6ac56fd694a9945ae)

将.whl文件下载后通过远程文件传输软件winscp上传到Jetson nano中，进入.whl所在的文件夹并输入以下命令安装：pip3 install paddlepaddle_gpu-2.1.1-cp36-cp36m-linux_aarch64.whl，最后测试一下PaddlePaddle是否安装成功：



```python
# 打开python3测试
import paddle
paddle.fluid.install_check.run_check()
```

输出结果参考：
![](https://ai-studio-static-online.cdn.bcebos.com/bdcd4b58d1454925b02e0adb6a38a22881a881a02b6649c9accc2fd5f0828969)


### 5.部署模型

将已经在AI Studio中导出的模型下载下来

>存储模型结构的 inference.pdmodel

>存储模型参数的 inference.pdiparams

![](https://ai-studio-static-online.cdn.bcebos.com/d83f98444a9341799d6ed340536186e641c7f31f7cf44f73977a14e1e0ecff4a)

将这两个模型文件传入Jetson nano即可


### 6.预测运行

新建一个.txt文件，用来存放标签“ball”。

#### 在Jetson Nano上运行以下代码:

#### 1. 导入资源库


```python
import cv2
import numpy as np
from paddle.inference import Config
from paddle.inference import PrecisionType
from paddle.inference import create_predictor
import yaml
import time
import threading
```

#### 2. 图像预处理


```python
def resize(img, target_size):
    """resize to target size"""
    if not isinstance(img, np.ndarray):
        raise TypeError('image type is not numpy.')
    im_shape = img.shape
    im_size_min = np.min(im_shape[0:2])
    im_size_max = np.max(im_shape[0:2])
    im_scale_x = float(target_size) / float(im_shape[1])
    im_scale_y = float(target_size) / float(im_shape[0])
    img = cv2.resize(img, None, None, fx=im_scale_x, fy=im_scale_y)
    return img

def normalize(img, mean, std):
    img = img / 255.0
    mean = np.array(mean)[np.newaxis, np.newaxis, :]
    std = np.array(std)[np.newaxis, np.newaxis, :]
    img -= mean
    img /= std
    return img

def preprocess(img, img_size):
    mean = [0.485, 0.456, 0.406]
    std = [0.229, 0.224, 0.225]
    img = resize(img, img_size)
    img = img[:, :, ::-1].astype('float32')  # bgr -> rgb
    img = normalize(img, mean, std)
    img = img.transpose((2, 0, 1))  # hwc -> chw
    return img[np.newaxis, :]
```

#### 3. 模型配置和预测


```python
def predict_config(model_file, params_file):
    '''
    函数功能：初始化预测模型predictor
    函数输入：模型结构文件，模型参数文件
    函数输出：预测器predictor
    '''
    # 根据预测部署的实际情况，设置Config
    config = Config()
    # 读取模型文件
    config.set_prog_file(model_file)
    config.set_params_file(params_file)
    # Config默认是使用CPU预测，若要使用GPU预测，需要手动开启，设置运行的GPU卡号和分配的初始显存。
    config.enable_use_gpu(500, 0)
    # 可以设置开启IR优化、开启内存优化。
    config.switch_ir_optim()
    config.enable_memory_optim()
    config.enable_tensorrt_engine(workspace_size=1 << 30, precision_mode=PrecisionType.Half,max_batch_size=1, min_subgraph_size=5, use_static=False, use_calib_mode=False)
    predictor = create_predictor(config)
    return predictor

def predict(predictor, img):
    
    '''
    函数功能：初始化预测模型predictor
    函数输入：模型结构文件，模型参数文件
    函数输出：预测器predictor
    '''
    input_names = predictor.get_input_names()
    for i, name in enumerate(input_names):
        input_tensor = predictor.get_input_handle(name)
        input_tensor.reshape(img[i].shape)
        input_tensor.copy_from_cpu(img[i].copy())
    # 执行Predictor
    predictor.run()
    # 获取输出
    results = []
    # 获取输出
    output_names = predictor.get_output_names()
    for i, name in enumerate(output_names):
        output_tensor = predictor.get_output_handle(name)
        output_data = output_tensor.copy_to_cpu()
        results.append(output_data)
    return results
```

#### 4. 后处理


```python
def draw_bbox_image(frame, result, label, threshold=0.5):
    
    for res in result:
        cat_id, score, bbox = res[0], res[1], res[2:]
        if score < threshold:
    	    continue
        for i in bbox:
            int(i)
        xmin, ymin, xmax, ymax = bbox
        cv2.rectangle(frame, (xmin, ymin), (xmax, ymax), (255,0,255), 2)
        print('category id is {}, bbox is {}'.format(cat_id, bbox))
        try:
            label_id = label[int(cat_id)]
            # #cv2.putText(图像, 文字, (x, y), 字体, 大小, (b, g, r), 宽度)
            cv2.putText(frame, label_id, (int(xmin), int(ymin-2)), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255,0,0), 2)
            cv2.putText(frame, str(round(score,2)), (int(xmin), int(ymin+8)), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0,255,0), 2)
        except KeyError:
            pass

def show_price(frame, result, label, price, threshold=0.5):
    w, h, _ = img.shape
    for res in result:
        cat_id, score, bbox = res[0], res[1], res[2:]
        if score < threshold:
            continue
        label_sum.append(label[int(cat_id)])
        price_sum += int(price[int(cat_id)])
    cv2.putText(frame, "price:"+str(price_sum), (int(w-4), 30), cv2.FONT_HERSHEY_SIMPLEX, 1, (0,0,255), 2)
```

#### 5. 定义摄像头类

在实际运行过程中，摄像头帧率较高，但nano的处理速度跟不上，会出现卡帧的情况，这时可以通过多线程的技巧改善这一问题。


```python
class Camera:
    def __init__(self, src=0):
        self.src = src
        self.stream = cv2.VideoCapture(src)
        self.stopped = False
        self.thread =  threading.Thread(target=self.update, args=())
        for _ in range(10): #warm up the camera
            (self.grabbed, self.frame) = self.stream.read()
        
    def start(self):
       self.thread.start()

    def update(self):
        while True:
            if self.stopped:
                return
            (self.grabbed, self.frame) = self.stream.read()

    def read(self):
        return self.grabbed, self.frame

    def stop(self):
        self.stopped = True
    
    def release(self):
        self.stream.release()
```

#### 6. 主函数


```python
# 从.txt文件中读取label
f = open("./label_list.txt")
label_list = f.read().splitlines()
label = [i.split(":")[0] for i in label_list]
price = [i.split(":")[1] for i in label_list]
print(label)
print(price)
f.close()

# 配置模型参数
model_file = "./model/yolo/model.pdmodel"#这里根据具体的模型更改就行，路径可以自行设定，也可以参考我的
params_file = "./model/yolo/model.pdiparams"
# 初始化预测模型
predictor = predict_config(model_file, params_file)

cap = Camera(0)
cap.start()

# 图像尺寸相关参数初始化
ret, img = cap.read()
if ret==False:
    while True:
        print("error")
im_size = 608 
scale_factor = np.array([im_size * 1. / img.shape[0], im_size * 1. / img.shape[1]]).reshape((1, 2)).astype(np.float32)
im_shape = np.array([im_size, im_size]).reshape((1, 2)).astype(np.float32)

while True:
    ret, frame = cap.read()
    # print(frame)
    # 预处理
    data = preprocess(frame, im_size)

    time_start = time.time()
    # 预测
    result = predict(predictor, [im_shape, data, scale_factor])
    # print(result)
    print('Time Cost：{}'.format(time.time()-time_start) , "s")

    draw_bbox_image(frame, result[0], label, threshold=0.5)
    show_price(frame, result[0], label, price, threshold=0.5)

    cv2.imshow("frame", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

print("break")
cap.stop()
cap.release()
cv2.destroyAllWindows()
```

##### 到了这里，乒乓球识别的模型已经成功运行。

![](https://ai-studio-static-online.cdn.bcebos.com/dd9f461b7cd34e7b885683ae4277085d79226f604fdd4ac8bc94f2a093a0a2a9)


# 四、舵机控制

## 1.机械臂及四个轮子控制


```python
#!/usr/bin/python
# -*- coding: utf-8 -*-

import time
import math
import smbus
import RPi.GPIO as GPIO

Dir = [
    'forward',
    'backward',
]


class serov(object):  # 定义一个类people
  class Struct(object):
    def __init__(self, max_angle, min_angle, num):
      self.num = num
      self.max_angle = max_angle
      self.min_angle = min_angle

  def make_struct(self, max_angle, min_angle, num):
    return self.Struct(max_angle, min_angle, num)


myserov = serov()
myserov_1 = myserov.make_struct(1000, 2300, 12)
myserov_2 = myserov.make_struct(500, 700, 13)
myserov_3 = myserov.make_struct(2300, 2300, 14)
myserov_4 = myserov.make_struct(800, 1500, 15)

class PCA9685:
    # Registers/etc.
    __SUBADR1 = 0x02
    __SUBADR2 = 0x03
    __SUBADR3 = 0x04
    __MODE1 = 0x00
    __PRESCALE = 0xFE
    __LED0_ON_L = 0x06
    __LED0_ON_H = 0x07
    __LED0_OFF_L = 0x08
    __LED0_OFF_H = 0x09
    __ALLLED_ON_L = 0xFA
    __ALLLED_ON_H = 0xFB
    __ALLLED_OFF_L = 0xFC
    __ALLLED_OFF_H = 0xFD

    def __init__(self, address=0x40, debug=False):
        self.bus = smbus.SMBus(1)
        self.address = address
        self.debug = debug
        if (self.debug):
            print("Reseting PCA9685")
        self.write(self.__MODE1, 0x00)

    def write(self, reg, value):
        "Writes an 8-bit value to the specified register/address"
        self.bus.write_byte_data(self.address, reg, value)
        if (self.debug):
            print("I2C: Write 0x%02X to register 0x%02X" % (value, reg))

    def read(self, reg):
        "Read an unsigned byte from the I2C device"
        result = self.bus.read_byte_data(self.address, reg)
        if (self.debug):
            print("I2C: Device 0x%02X returned 0x%02X from reg 0x%02X" % (self.address, result & 0xFF, reg))
        return result

    def setPWMFreq(self, freq):
        "Sets the PWM frequency"
        prescaleval = 25000000.0  # 25MHz
        prescaleval /= 4096.0  # 12-bit
        prescaleval /= float(freq)
        prescaleval -= 1.0
        if (self.debug):
            print("Setting PWM frequency to %d Hz" % freq)
            print("Estimated pre-scale: %d" % prescaleval)
        prescale = math.floor(prescaleval + 0.5)
        if (self.debug):
            print("Final pre-scale: %d" % prescale)

        oldmode = self.read(self.__MODE1);
        newmode = (oldmode & 0x7F) | 0x10  # sleep
        self.write(self.__MODE1, newmode)  # go to sleep
        self.write(self.__PRESCALE, int(math.floor(prescale)))
        self.write(self.__MODE1, oldmode)
        time.sleep(0.005)
        self.write(self.__MODE1, oldmode | 0x80)

    def setPWM(self, channel, on, off):
        "Sets a single PWM channel"
        self.write(self.__LED0_ON_L + 4 * channel, on & 0xFF)
        self.write(self.__LED0_ON_H + 4 * channel, on >> 8)
        self.write(self.__LED0_OFF_L + 4 * channel, off & 0xFF)
        self.write(self.__LED0_OFF_H + 4 * channel, off >> 8)
        if (self.debug):
            print("channel: %d  LED_ON: %d LED_OFF: %d" % (channel, on, off))

    def setServoPulse(self, channel, pulse):
        "Sets the Servo Pulse,The PWM frequency must be 50HZ"
        pulse = pulse * 4096 / 20000  # PWM frequency is 50HZ,the period is 20000us
        self.setPWM(channel, 0, int(pulse))

    def setDutycycle(self, channel, pulse):
        self.setPWM(channel, 0, int(pulse * (4096 / 100)))

    def setLevel(self, channel, value):
        if (value == 1):
          self.setPWM(channel, 0, 4095)
        else:
          self.setPWM(channel, 0, 0)
  



# 控制机器人库
class LOBOROBOT():
    def __init__(self):
        self.PWMA = 0
        self.AIN1 = 2
        self.AIN2 = 1

        self.PWMB = 5
        self.BIN1 = 3
        self.BIN2 = 4

        self.PWMC = 6
        self.CIN2 = 7
        self.CIN1 = 8

        self.PWMD = 11
        self.DIN1 = 25
        self.DIN2 = 24
        self.pwm = PCA9685(0x40, debug=False)
        self.pwm.setPWMFreq(50)
        GPIO.setwarnings(False)
        GPIO.setmode(GPIO.BCM)
        GPIO.setup(self.DIN1,GPIO.OUT)
        GPIO.setup(self.DIN2,GPIO.OUT)

    def MotorRun(self, motor, index, speed):
        if speed > 100:
            return
        if(motor == 0):
            self.pwm.setDutycycle(self.PWMA, speed)
            if(index == Dir[0]):
                self.pwm.setLevel(self.AIN1, 0)
                self.pwm.setLevel(self.AIN2, 1)
            else:
                self.pwm.setLevel(self.AIN1, 1)
                self.pwm.setLevel(self.AIN2, 0)
        elif(motor == 1):
            self.pwm.setDutycycle(self.PWMB, speed)
            if(index == Dir[0]):
                self.pwm.setLevel(self.BIN1, 1)
                self.pwm.setLevel(self.BIN2, 0)
            else:
                self.pwm.setLevel(self.BIN1, 0)
                self.pwm.setLevel(self.BIN2, 1)
        elif(motor == 2):
            self.pwm.setDutycycle(self.PWMC,speed)
            if(index == Dir[0]):
                self.pwm.setLevel(self.CIN1,1)
                self.pwm.setLevel(self.CIN2,0)
            else:
                self.pwm.setLevel(self.CIN1,0)
                self.pwm.setLevel(self.CIN2,1)
        elif(motor == 3):
            self.pwm.setDutycycle(self.PWMD,speed)
            if (index == Dir[0]):
                GPIO.output(self.DIN1,0)
                GPIO.output(self.DIN2,1)
            else:
                GPIO.output(self.DIN1,1)
                GPIO.output(self.DIN2,0)

    def MotorStop(self, motor):
        if (motor == 0):
            self.pwm.setDutycycle(self.PWMA, 0)
        elif(motor == 1):
            self.pwm.setDutycycle(self.PWMB, 0)
        elif(motor == 2):
            self.pwm.setDutycycle(self.PWMC, 0)
        elif(motor == 3):
            self.pwm.setDutycycle(self.PWMD, 0)
    # 前进
    def t_up(self,speed,t_time):
        self.MotorRun(0,'backward',speed)
        self.MotorRun(1,'forward',speed)
        self.MotorRun(2,'forward',speed)
        self.MotorRun(3,'backward',speed)
        time.sleep(t_time)
    #后退
    def t_down(self,speed,t_time):

        self.MotorRun(0,'forward',speed)
        self.MotorRun(1,'backward',speed)
        self.MotorRun(2,'backward',speed)
        self.MotorRun(3,'forward',speed)
        time.sleep(t_time)

    # 左移
    def moveLeft(self,speed,t_time):
        self.MotorRun(0,'backward',speed)
        self.MotorRun(1,'forward',speed)
        self.MotorRun(2,'backward',speed)
        self.MotorRun(3,'forward',speed)
        time.sleep(t_time)

    #右移
    def moveRight(self,speed,t_time):
        
        self.MotorRun(0,'forward',speed)
        self.MotorRun(1,'backward',speed)
        self.MotorRun(2,'forward',speed)
        self.MotorRun(3,'backward',speed)
        time.sleep(t_time)

    # 左转
    def turnLeft(self,speed,t_time):
        self.MotorRun(1,'backward',speed)
        self.MotorRun(2,'backward',speed)
        self.MotorRun(3,'backward',speed)
        self.MotorRun(0,'backward',speed)
        time.sleep(t_time)
    
    # 右转
    def turnRight(self,speed,t_time):
        self.MotorRun(0,'forward',speed)
        self.MotorRun(1,'forward',speed)
        self.MotorRun(2,'forward',speed)
        self.MotorRun(3,'forward',speed)
        time.sleep(t_time)
    
    # 停止
    def t_stop(self,t_time):
        self.MotorStop(0)
        self.MotorStop(1)
        self.MotorStop(2)
        self.MotorStop(3)
        time.sleep(t_time)

        # 辅助功能，使设置舵机脉冲宽度更简单。

    def magnitude_move_mechanical_arm(self,num, status):

        if num == 1:
            self.move_serov_aim = myserov_1
        elif num == 2:
            self.move_serov_aim = myserov_2
        elif num == 3:
            self.move_serov_aim = myserov_3
        elif num == 4:
            self.move_serov_aim = myserov_4

        if (status == "up"):
            self.end_angle = self.move_serov_aim.min_angle
            self.start_angle = self.move_serov_aim.max_angle
            movestep = 10
        else:
            self.end_angle = self.move_serov_aim.max_angle
            self.start_angle = self.move_serov_aim.min_angle
            movestep = -10
        for i in range(self.start_angle, self.end_angle, movestep):
            self.pwm.setServoPulse(self.move_serov_aim.num, i)
            time.sleep(0.02)

    def move_machine_arm(self):


        self.pwm.setServoPulse(14, 2300)
        self.magnitude_move_mechanical_arm(4, "up")
        self.magnitude_move_mechanical_arm(2, "up")
        self.magnitude_move_mechanical_arm(1, "up")
        self.magnitude_move_mechanical_arm(4, "down")

        self.magnitude_move_mechanical_arm(1, "down")

        self.magnitude_move_mechanical_arm(2, "down")
        self.magnitude_move_mechanical_arm(4, "up")

    # 设置舵机角度函数
```

## 2.调用控制小车运动和乒乓球识别模型的处理程序

用于将小车和模型识别的结果关联起来，控制小车运动。


```python
import math

import cv2
import numpy as np
from paddle.inference import Config
from paddle.inference import PrecisionType
from paddle.inference import create_predictor
import yaml
import time
import threading

from LOBOROBOT import LOBOROBOT


def resize(img, target_size):
    """resize to target size"""
    if not isinstance(img, np.ndarray):
        raise TypeError('image type is not numpy.')
    im_shape = img.shape
    im_size_min = np.min(im_shape[0:2])
    im_size_max = np.max(im_shape[0:2])
    im_scale_x = float(target_size) / float(im_shape[1])
    im_scale_y = float(target_size) / float(im_shape[0])
    img = cv2.resize(img, None, None, fx=im_scale_x, fy=im_scale_y)
    return img

def normalize(img, mean, std):
    img = img / 255.0
    mean = np.array(mean)[np.newaxis, np.newaxis, :]
    std = np.array(std)[np.newaxis, np.newaxis, :]
    img -= mean
    img /= std
    return img

def preprocess(img, img_size):
    mean = [0.485, 0.456, 0.406]
    std = [0.229, 0.224, 0.225]
    img = resize(img, img_size)
    img = img[:, :, ::-1].astype('float32')  # bgr -> rgb
    img = normalize(img, mean, std)
    img = img.transpose((2, 0, 1))  # hwc -> chw
    return img[np.newaxis, :]


def predict_config(model_file, params_file):
    '''
    函数功能：初始化预测模型predictor
    函数输入：模型结构文件，模型参数文件
    函数输出：预测器predictor
    '''
    # 根据预测部署的实际情况，设置Config
    config = Config()
    # 读取模型文件
    config.set_prog_file(model_file)
    config.set_params_file(params_file)
    # Config默认是使用CPU预测，若要使用GPU预测，需要手动开启，设置运行的GPU卡号和分配的初始显存。
    config.enable_use_gpu(500, 0)
    # 可以设置开启IR优化、开启内存优化。
    config.switch_ir_optim()
    config.enable_memory_optim()
    config.enable_tensorrt_engine(workspace_size=1 << 30, precision_mode=PrecisionType.Half,max_batch_size=1, min_subgraph_size=5, use_static=False, use_calib_mode=False)
    predictor = create_predictor(config)
    return predictor

def predict(predictor, img):
    
    '''
    函数功能：初始化预测模型predictor
    函数输入：模型结构文件，模型参数文件
    函数输出：预测器predictor
    '''
    input_names = predictor.get_input_names()
    for i, name in enumerate(input_names):
        input_tensor = predictor.get_input_handle(name)
        input_tensor.reshape(img[i].shape)
        input_tensor.copy_from_cpu(img[i].copy())
    # 执行Predictor
    predictor.run()
    # 获取输出
    results = []
    # 获取输出
    output_names = predictor.get_output_names()
    for i, name in enumerate(output_names):
        output_tensor = predictor.get_output_handle(name)
        output_data = output_tensor.copy_to_cpu()
        results.append(output_data)
    return results


def draw_bbox_image(frame, result, text, threshold=0.5):
    
    for res in result:
        cat_id, score, bbox = res[0], res[1], res[2:]
        if score < threshold:
    	    continue
        for i in bbox:
            int(i)
        xmin, ymin, xmax, ymax = bbox
        cv2.rectangle(frame, (int(xmin), int(ymin)), (int(xmax), int(ymax)), (255,0,255), 2)
        print('category id is {}, bbox is {}'.format(cat_id, bbox))
        try:
            # #cv2.putText(图像, 文字, (x, y), 字体, 大小, (b, g, r), 宽度)
            cv2.putText(frame, text, (int(xmin), int(ymin-2)), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255,0,0), 2)
            cv2.putText(frame, str(round(score,2)), (int(xmin), int(ymin+8)), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0,255,0), 2)
        except KeyError:
            pass



class Camera:
    def __init__(self, src=0):
        self.src = src
        self.stream = cv2.VideoCapture(src)
        self.stopped = False
        self.thread =  threading.Thread(target=self.update, args=())
        for _ in range(10): #warm up the camera
            (self.grabbed, self.frame) = self.stream.read()
        
    def start(self):
       self.thread.start()

    def update(self):
        while True:
            if self.stopped:
                return
            (self.grabbed, self.frame) = self.stream.read()

    def read(self):
        return self.grabbed, self.frame

    def stop(self):
        self.stopped = True
    
    def release(self):
        self.stream.release()
def Distance(px):
    F=1150
    W=4.0
    return W*F/px

#计算机视觉识别任务,阻塞式的识别体系
def task(num,tscore,cap):
    res1=[]
    while num>0:
        ret, frame = cap.read()
        data = preprocess(frame, im_size)
        result = predict(predictor, [im_shape, data, scale_factor])
        if len(result[0]) == 0:
            num-=1
            continue
        res=result[0][0]
        cat_id, score, bbox = res[0], res[1], res[2:]
        if score>=tscore:
            xmin, ymin, xmax, ymax = bbox
            # 计算乒乓球的像素直径
            pingpong_px = math.sqrt((xmax - xmin) ** 2 + (ymax - ymin) ** 2)
            distance = Distance(pingpong_px)
            draw_bbox_image(frame, result[0], str(int(distance)) + "cm", tscore)
            num=0
            res1=res
        cv2.imshow("frame", frame)
        cv2.waitKey(1)
        num-=1
    return np.array(res1)

# 配置模型参数
model_file = "./model/yolo/model.pdmodel"
params_file = "./model/yolo/model.pdiparams"
# 初始化预测模型
predictor = predict_config(model_file, params_file)

cap = Camera(0)
cap.start()

# 图像尺寸相关参数初始化
ret, img = cap.read()
frame=img
if ret==False:
    while True:
        print("error")
im_size = 608

scale_factor = np.array([im_size * 1. / img.shape[0], im_size * 1. / img.shape[1]]).reshape((1, 2)).astype(np.float32)
im_shape = np.array([im_size, im_size]).reshape((1, 2)).astype(np.float32)

cbot=LOBOROBOT()
#前进阈值
UD_flag=0
while True:
    #陷入阻塞
    res=task(20,0.3,cap)
    if len(res)==0:
        print("当前视野没有乒乓球，小车开始左移")
        cbot.turnLeft(50, 1)
        continue
    cat_id, score, bbox = res[0], res[1], res[2:]
    xmin, ymin, xmax, ymax = bbox
    # 计算乒乓球的像素直径
    pingpong_px = math.sqrt((xmax - xmin) ** 2 + (ymax - ymin) ** 2)
    distance=Distance(pingpong_px)

    #计算乒乓球中心坐标
    x0 = (xmin + xmax) / 2
    w0 = frame.shape[1] / 2
    r = w0 - x0
    # 左移

    print("轨迹检测正常，继续前进")
    # ----------------------
    # 当距离在某一个范围时进行得操作
    if 23 < distance:
        print("小车前进")
        cbot.t_up(30,int((distance-20)/10))
        # ----------------------
        continue
    if distance < 17 :
        print("小车后退")
        # ----------------------
        cbot.t_down(30, int((distance-20)/10))
        continue

    #
    print("下爪")

print("break")
cap.stop()
cap.release()
cv2.destroyAllWindows()
```

#### 至此，一辆可以抓取乒乓球的小车出炉了。

![](https://ai-studio-static-online.cdn.bcebos.com/7afab2c078cb40b79a79925b05c1ee849ecbf6351f3e4e5a808c2c195729f138)


# 五、总结

> 第一次使用Jetson Nano很多东西不太熟悉，多亏了一位大佬的使用教程化解了我的困扰。由于该项目还涉及到了舵机的控制，但是Jetson Nano的引脚不适合接舵机，必须采用扩展板。
另外，Jetson Nano时不时就死机，导致项目很多时间都在排查问题，至今还是没有解决。后面考虑使用更高级的板子试试，可能是板载过高，Jetson Nano的自我保护机制造成的。

# 六、个人介绍

> 湖北经济学院 计算机科学与技术 在读大二

> 大一参加过计算机设计大赛智慧导盲赛道获省二

> 目前正在深入学习AI，上手不到一年

## 团队成员介绍

> 小云 湖北经济学院 电子商务 在读大一

> 精通Python，综合能力强

> 项目的核心关键代码由他提供

# 特别鸣谢
周国源团队、李成宇团队、唐老师
>>>>>>> 3bdb02c (2023-4-1)

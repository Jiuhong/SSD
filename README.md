# High quality, fast, modular reference implementation of SSD in PyTorch 1.0


This repository implements [SSD (Single Shot MultiBox Detector)](https://arxiv.org/abs/1512.02325). The implementation is heavily influenced by the projects [ssd.pytorch](https://github.com/amdegroot/ssd.pytorch), [pytorch-ssd](https://github.com/qfgaohao/pytorch-ssd) and [maskrcnn-benchmark](https://github.com/facebookresearch/maskrcnn-benchmark). This repository aims to be the code base for researches based on SSD.

<div align="center">
  <img src="figures/004545.jpg" width="500px" />
  <p>Example SSD output (ssd300_voc0712).</p>
</div>

| Losses        | Learning rate | Metrics |
| :-----------: |:-------------:| :------:|
| ![losses](figures/losses.png) | ![lr](figures/lr.png) | ![metric](figures/metrics.png) |

## Highlights

- PyTorch 1.0
- GPU/CPU NMS
- Multi-GPU training and inference
- Modular
- Visualization(Support Tensorboard)
- CPU support for inference
- Evaluating during training
- Metrics Visualization

## Installation
### Requirements

1. Python3
1. PyTorch 1.0
1. yacs
1. GCC >= 4.9
1. OpenCV

### Step-by-step installation

```bash
# First, make sure that your conda is setup properly with the right environment
# for that, check that `which conda`, `which pip` and `which python` points to the
# right path. From a clean conda env, this is what you need to do.
# But if you don't use conda, it's OK. Just pip install necessary packages.

conda create --name SSD
source activate SSD

# follow PyTorch installation in https://pytorch.org/get-started/locally/
conda install pytorch torchvision -c pytorch

pip install yacs tqdm
conda install opencv

# Optional packages
# If you want visualize loss curve. Default is enabled. Disable by using --use_tensorboard 0 when training.
pip install tensorboardX

# If you train coco dataset, must install cocoapi.
cd ~/github
git clone https://github.com/cocodataset/cocoapi.git
cd cocoapi/PythonAPI
python setup.py build_ext install

# Finally, download the pre-trained vgg weights.
wget https://s3.amazonaws.com/amdegroot-models/vgg16_reducedfc.pth
```

### Build

NMS build is not necessary, as we provide a python-like nms, but is 2x slower than build-version.
```bash
# For faster inference you need to build nms, this is needed when evaluating. Only training doesn't need this.
cd ext
python build.py build_ext develop
```

## Train

### Setting Up Datasets
#### Pascal VOC

For Pascal VOC dataset, make the folder structure like this:
```
VOC_ROOT
|__ VOC2007
    |_ JPEGImages
    |_ Annotations
    |_ ImageSets
    |_ SegmentationClass
|__ VOC2012
    |_ JPEGImages
    |_ Annotations
    |_ ImageSets
    |_ SegmentationClass
|__ ...
```
Where `VOC_ROOT` default is `datasets` folder in current project, you can create symlinks to `datasets` or `export VOC_ROOT="/path/to/voc_root"`.

#### COCO

For COCO dataset, make the folder structure like this:
```
COCO_ROOT
|__ annotations
    |_ instances_valminusminival2014.json
    |_ instances_minival2014.json
    |_ instances_train2014.json
    |_ instances_val2014.json
    |_ ...
|__ train2014
    |_ <im-1-name>.jpg
    |_ ...
    |_ <im-N-name>.jpg
|__ val2014
    |_ <im-1-name>.jpg
    |_ ...
    |_ <im-N-name>.jpg
|__ ...
```
Where `COCO_ROOT` default is `datasets` folder in current project, you can create symlinks to `datasets` or `export COCO_ROOT="/path/to/coco_root"`.

### Single GPU training

```bash
# for example, train SSD300:
python train.py --config-file configs/vgg_ssd300_voc0712.yaml
```
### Multi-GPU training

```bash
# for example, train SSD300 with 4 GPUs:
export NGPUS=4
python -m torch.distributed.launch --nproc_per_node=$NGPUS train.py --config-file configs/vgg_ssd300_voc0712.yaml
```
The configuration files that I provide assume that we are running on single GPU. When changing number of GPUs, hyper-parameter (lr, max_iter, ...) will also changed according to this paper: [Accurate, Large Minibatch SGD: Training ImageNet in 1 Hour](https://arxiv.org/abs/1706.02677).
The pre-trained vgg weights can be downloaded here: https://s3.amazonaws.com/amdegroot-models/vgg16_reducedfc.pth.

## Evaluate

### Single GPU evaluating

```bash
# for example, evaluate SSD300:
python test.py --config-file configs/vgg_ssd300_voc0712.yaml
```

### Multi-GPU evaluating

```bash
# for example, evaluate SSD300 with 4 GPUs:
export NGPUS=4
python -m torch.distributed.launch --nproc_per_node=$NGPUS test.py --config-file configs/vgg_ssd300_voc0712.yaml
```

## Demo

Predicting image in a folder is simple:
```bash
python demo.py --config-file configs/ssd300_voc0712.yaml --images_dir demo
```
Then the predicted images with boxes, scores and label names will saved to `demo/result` folder.

Currently, I provide weights trained as follows:

|         |    Weights   |
| :-----: | :----------: |
| SSD300* | [ssd300_voc0712_mAP77.83.pth(100 MB)](https://github.com/lufficc/SSD/releases/download/v1.0.1/ssd300_voc0712_mAP77.83.pth) |
| SSD512* | [ssd512_voc0712_mAP80.25.pth(104 MB)](https://github.com/lufficc/SSD/releases/download/v1.0.1/ssd512_voc0712_mAP80.25.pth) |

## Performance
### Origin Paper:

|         | VOC2007 test | coco test-dev2015 |
| :-----: | :----------: |   :----------:    |
|  Train  |     07+12    |    trainval35k    |
| SSD300* |     77.2     |      25.1         |
| SSD512* |     79.8     |      28.8         |

### Our Implementation:

|         | VOC2007 test |          COCO 2014 minival               |
| :-----: | :----------: |   :----------------------------------:   |
|  Train  |     07+12    |          trainval35k                     |
| SSD300* |     77.8     |          25.5                            |
| SSD512* |     80.2     |          -                               |

## Troubleshooting
If you have issues running or compiling this code, we have compiled a list of common issues in [TROUBLESHOOTING.md](TROUBLESHOOTING.md). If your issue is not present there, please feel free to open a new issue.
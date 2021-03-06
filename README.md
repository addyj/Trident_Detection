# Trident Detection: Plane, Object and Depth
## Introduction

A deep neural architecture, which combines PlaneR-CNN, YOLOv3 and Midas architecture to detects arbitrary number of planes, depth and bounding boxes for a image.

For train log [Colab Train](https://colab.research.google.com/drive/1dzrYak3wLTHlpLf_I1viiH7EtTpwSj2E?usp=sharing).

Repositories referred and merged [PlaneRCNN](https://github.com/NVlabs/planercnn), [YoloV3](https://github.com/theschoolofai/YoloV3) and [MiDas](https://github.com/intel-isl/MiDaS).

The code is implemented using PyTorch.

## Getting Started 
Clone repository: 
```
git clone https://github.com/Trident_Detection.git
```

Build RoI Align module
```
cd Trident_Detection/roialign/
python setup.py install
cd ..
```

For **nms** we use torchvision inbuilt ops and for **roialign** refer [RoIAlign Repo](https://github.com/longcw/RoIAlign.pytorch)

Make **checkpoint/** folder for plane-rcnn and **weights/** folder for yolov3 pretrained weigths. For midas the pretrained weights are directly loaded url.

Once you have a saved pretrained weight file for this model you can comment out the load code and load the pretrained wts file. [automation for this is not available for now.]

Load custom data into **data/** folder. Refer YOLOv3 dataset structure construction. Refer the custom data set for reference [Trident Data Full](https://drive.google.com/file/d/1Oia4b6dNvxs9TbFCs8_60VS2iWxcHgeW/view?usp=sharing) and [Trident Data](https://drive.google.com/file/d/1orpEeyaq9LExJ_MyH7XCAQyxTcTxzTpJ/view?usp=sharing) . Also follow the extension according to the custom dataset.

Segmentation image for training were created using the plane masks inferred from the original NVlabs plane repo. [Removed Scannet Dataset dependency altogether]

Dataset structure:
```
data
  --customdata
    --images/
      --img001.jpg
      --img002.jpg
      --...
    --labels/
      --img001.txt
      --img002.txt
      --...
    --depth/
      --img001.npz
      --img002.npz
      --...
    --segmentation_masks/
      --img001.npz
      --img002.npz
      --...
    --planes/
      --img001.npy
      --img002.npy
      --...
    custom.data #data file
    custom.names #your class names
    train.txt #train paths
    test.txt #test paths
    train.shapes #train shapes
    test.shapes #test shapes
    camera.txt #camera intrinsics
```

## Model

### MiDAS

Took  MiDAS ResNext101 as the backbone network for feature extractor acting as Encoder, and the stages of  ResNext101 are skipped to Midas Decoder, YoloV3 and PlaneRCNN.
MiDAS both encoder and Decoder layers were frozen.

### YoloV3

For YoloV3 model integration, the original configuration file was bisected from layer 75 and added to Midas encoder for bounding box prediction. [New YOLO v3 Config File](https://github.com/addyj/Trident_Detection/blob/master/cfg/yolov3-custom.cfg)

As MiDAS Encoder's - ResNext101 and  YoloV3's - DarkNet are different, added additional layers were added between Encoder and Yolo predictor to match the dimensions.

### PlaneRCNN

Same as Yolo the Midas Encoder outs from ResNext101 were feeded to PlaneRCNN Feature Pyramid Network (FPN) and no intermidiate layers were added as the dimension matched to the PlaneRCNN Resnet101. 
Note: Should add some intermidiate layers for proper merging.

## Training
Due to memory restrictions cant go more than batch size of 5. 

MiDAS model is freezed, but for training loss calculation RMSE Loss and L1Loss combination is used.

The loss function used for YoloV3 is the same as what is used in vanilla YoloV3.

The loss function used for PlaneRCNN is the same as what is used in vanilla PlaneRCNN. The refinement part of the training was removed, as it was just reaffirming theplane prediction.
PlaneRCNN  cannot be trained with a batch size of more than 1 so refactored the code to run on batch in loop and calculate the loss accordingly.

The total loss was taken as  sum of all the three individual losses.

### Training script

```bash
python custom_train.py --cfg cfg/yolov3-custom.cfg --dataset=custom --data data/customdata/custom.data --batchSize=4 --numEpochs 3
```
options:
Refer [options.py](https://github.com/addyj/Trident_Detection/blob/master/options.py)

TODO:

1. Validation Code Integration
2. Result Plotting and Saving.
3. Add Intermediate layers for connecting encoder n decoder
4. Add back the refinement PRCNN model. 
5. Refactoring to run on different sizes.
4. More Training and better Loss function identification.
5. Unfreeze, load, and train more.


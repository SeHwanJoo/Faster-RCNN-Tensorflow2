# Faster-RCNN

## Dataset
Faster-RCNN model was trained with VOC 2007 object detection data

## Preprocessing
Resize imagse size (1000, 600)

Apply random flip horizontally.

Padded for each batch size due to the number of other bboxes.

## Anchor
    "image_size": (1000, 600)
    "feature_map_shape": (62, 37)
    "anchor_ratios": [1., 2., 1./2.]
    "anchor_scales": [128, 256, 512]
    "anchor_count": 9 (ratios.size * scales.size)
    "total_anchor": 62 * 37 * 9 (feature_map_width * feature_map_height * anchor_count)

## input
    "iamge": (batchsize, image_width, image_height, rgb)
    "gt_boxes": (batchsize, max number of the other bboxes each batch size, 4)
    "gt_labels": (batchsize, 4(box_x, box_y, box_width, box_height))
    "bbox_deltas": (batchsize, total_anchor, 4(box_x, box_y, box_width, box_height))
    "bbox_labels": (batchsize, feature_map_width, feature_map_height, anchor_count)
        
## Region proposal Networks (RPN)
### backbone
baseModel vgg16

    Model: "vgg16"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    input_1 (InputLayer)         [(None, 1000, 600, 3)]    0         
    _________________________________________________________________
    block1_conv1 (Conv2D)        (None, 1000, 600, 64)     1792      
    _________________________________________________________________
    block1_conv2 (Conv2D)        (None, 1000, 600, 64)     36928     
    _________________________________________________________________
    block1_pool (MaxPooling2D)   (None, 500, 300, 64)      0         
    _________________________________________________________________
    block2_conv1 (Conv2D)        (None, 500, 300, 128)     73856     
    _________________________________________________________________
    block2_conv2 (Conv2D)        (None, 500, 300, 128)     147584    
    _________________________________________________________________
    block2_pool (MaxPooling2D)   (None, 250, 150, 128)     0         
    _________________________________________________________________
    block3_conv1 (Conv2D)        (None, 250, 150, 256)     295168    
    _________________________________________________________________
    block3_conv2 (Conv2D)        (None, 250, 150, 256)     590080    
    _________________________________________________________________
    block3_conv3 (Conv2D)        (None, 250, 150, 256)     590080    
    _________________________________________________________________
    block3_pool (MaxPooling2D)   (None, 125, 75, 256)      0         
    _________________________________________________________________
    block4_conv1 (Conv2D)        (None, 125, 75, 512)      1180160   
    _________________________________________________________________
    block4_conv2 (Conv2D)        (None, 125, 75, 512)      2359808   
    _________________________________________________________________
    block4_conv3 (Conv2D)        (None, 125, 75, 512)      2359808   
    _________________________________________________________________
    block4_pool (MaxPooling2D)   (None, 62, 37, 512)       0         
    _________________________________________________________________
    block5_conv1 (Conv2D)        (None, 62, 37, 512)       2359808   
    _________________________________________________________________
    block5_conv2 (Conv2D)        (None, 62, 37, 512)       2359808   
    _________________________________________________________________
    block5_conv3 (Conv2D)        (None, 62, 37, 512)       2359808   
    _________________________________________________________________

### RPN conv
apply backbone feature map to rpn_conv
This feature map fed in two sibling fully connected layers (rpn_reg, rpn_cls).

    __________________________________________________________________________________________________
    Layer (type)                    Output Shape         Param #   
    ==================================================================================================
    rpn_conv (Conv2D)               (None, 62, 37, 512)  2359808     block5_conv3[0][0]               
    __________________________________________________________________________________________________
    rpn_reg (Conv2D)                (None, 62, 37, 36)   18468       rpn_conv[0][0]                   
    __________________________________________________________________________________________________
    rpn_cls (Conv2D)                (None, 62, 37, 9)    4617        rpn_conv[0][0]                   
    ==================================================================================================
    

## Fast RCNN (Detector)
### backbone
RPN with ROI(Region Of Interest pooling) layer
### ROI pooling
Region Of Interest pooling

    __________________________________________________________________________________________________
    Layer (type)                    Output Shape               Param #   
    ==================================================================================================
    roi_bboxes (RoIBBox)            (None, 2000, 4)            0           rpn_reg[0][0]                    
                                                                           rpn_cls[0][0]                    
    __________________________________________________________________________________________________
    roi_pooling (RoIPooling)        (None, None, 7, 7, 512)    0           block5_conv3[0][0]               
                                                                           roi_bboxes[0][0]                 
    __________________________________________________________________________________________________

### Loss function



## Hyper parameter
    pre_nms_topn = 6000
    train_nms_topn = 2000
    test_nms_topn = 300
    nms_iou_threshold = 0.7
    pooling_size = (7, 7)

## Accuracy
|Model|Validation Accuracy
|:------:|:---:|
## Graph
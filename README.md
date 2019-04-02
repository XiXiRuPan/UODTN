# Codes for UODTN

We implement our experiments with PyTorch 1.0.

## Step 1 Generate graph for AwA
```
cd GCN/materials/AWA2 
```

Download: http://nlp.stanford.edu/data/glove.6B.zip  
Unzip it, find and put glove.6B.300d.txt here.

### Construct graph:
```
python make_graph_animal_step1.py  
python make_graph_animal_step2.py  
```
Result in animals_graph_all.json, the graph for AwA2 which contains 255 nodes.  

### Extract clasifiers from Resnet-50 pretrained on ImageNet for categories that shared by the construted graph and ImageNet-1K.
Download: https://download.pytorch.org/models/resnet50-19c8e357.pth  
Rename and put it as materials/resnet50-raw.pth  
```
python extract_127+24_from_1K.py  
```

Result in 151_cls_from_1K.  

## Step 2 Prepare data

We upload the source domain of I2AwA :  
https://drive.google.com/file/d/1GdDZ1SvEqGin_zeCAGaJn0821vC_PJmc/view?usp=sharing  
Or  
https://pan.baidu.com/s/122-cvnjhYb0mB1zf4nShdA with password: ibee  

For the target domain, one can download from http://cvml.ist.ac.at/AwA2/. The link is as follow:

http://cvml.ist.ac.at/AwA2/AwA2-data.zip  

`Note that we do not have the rights of the shared images. Please do not use our dataset for commercial purpose.`

We recommend that I2AwA can be used for traditional domain adaptation.

## Step 3 Train classifier on source domain

We find that directly finetune a classifier on source domain and then transfer 40 known classifier to unkown category via GCN resulting very poor results. The reason may be in that only 40 known classifier to train GCN is not sufficient. Therefore, we utilize 127+24 classifier pretrained on ImageNet-1K to train GCN. Specifically, 24 indicates that the classifiers are in source domain while the other 127 classifiers are not in target domain but are included in the constructed graph for target domain. We need to train the missing 16 classisifers for source domain. However, the classifiers pretrained on ImageNet-1K is discpreant from source domain. Thus, we fix the 24 classifiers and train 16 classifiers on source domain. Then we use these 127+24+16 classifiers to train GCN and obtain the initail classifiers for unknown categories.  

```
cd UODTN  
python train_40_cls_with_help_of_1K.py  
```

Result in 151+16_cls_from_1K_ft and base_net_pretrained_on_I2AwA2_source_onl. 151+16_cls_from_1K_ft includes the original 127+24 classifiers and the additional 16 classifiers finetune from the source domain. base_net_pretrained_on_I2AwA2_source_onl is a trained feature extractor for I2AwA.  

## Step 4 Train UODTN

Train a GCN:  

```
cd GCN  
python train_gcn_basic_awa_ezhuo_2019.py  
python extract_50_cls_from_all_graph_for_awa.py  
```

Result in awa_50_cls_basic, which is 50 classifiers for AwA2.  

### Prepare matching:

```
cd data/Matching  
python extract_feature.py  
python match_5_split.py  
```

 
### Train UODTN:
 
```
python baseline_align_ezhuo_2019.py  
```


Acknowledgements: Our codes are mainly based on https://github.com/cyvius96/adgpm and https://github.com/thuml/Xlearn/tree/master/pytorch

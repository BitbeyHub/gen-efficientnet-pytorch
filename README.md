# (Generic) EfficientNets for PyTorch

A 'generic' implementation of EfficientNet, MixNet, MobileNetV3, etc. that covers most of the compute/parameter efficient architectures derived from the MobileNet V1/V2 block sequence, including those found via automated neural architecture search. 

All models are implemented by GenEfficientNet or MobileNetV3 classes, with string based architecture definitions to configure the block layouts (idea from [here](https://github.com/tensorflow/tpu/blob/master/models/official/mnasnet/mnasnet_models.py))

## What's New

### Jan 22, 2020
 * Update weights for EfficientNet B0, B2, B3 and MixNet-XL with latest RandAugment trained weights. Trained with (https://github.com/rwightman/pytorch-image-models)
 * Fix torchscript compatibility for PyTorch 1.4, add torchscript support for MixedConv2d using ModuleDict
 * Test models, torchscript, onnx export with PyTorch 1.4 -- no issues

### Nov 22, 2019
 * New top-1 high! Ported official TF EfficientNet AdvProp (https://arxiv.org/abs/1911.09665) weights and B8 model spec. Created a new set of `ap` models since they use a different
 preprocessing (Inception mean/std) from the original EfficientNet base/AA/RA weights.
 
### Nov 15, 2019
 * Ported official TF MobileNet-V3 float32 large/small/minimalistic weights
 * Modifications to MobileNet-V3 model and components to support some additional config needed for differences between TF MobileNet-V3 and mine

### Oct 30, 2019
 * Many of the models will now work with torch.jit.script, MixNet being the biggest exception
 * Improved interface for enabling torchscript or ONNX export compatible modes (via config)
 * Add JIT optimized mem-efficient Swish/Mish autograd.fn in addition to memory-efficient autgrad.fn
 * Activation factory to select best version of activation by name or override one globally
 * Add pretrained checkpoint load helper that handles input conv and classifier changes
 
### Oct 27, 2019
 * Add CondConv EfficientNet variants ported from https://github.com/tensorflow/tpu/tree/master/models/official/efficientnet/condconv
 * Add RandAug weights for TF EfficientNet B5 and B7 from https://github.com/tensorflow/tpu/tree/master/models/official/efficientnet
 * Bring over MixNet-XL model and depth scaling algo from my pytorch-image-models code base
 * Switch activations and global pooling to modules
 * Add memory-efficient Swish/Mish impl
 * Add as_sequential() method to all models and allow as an argument in entrypoint fns
 * Move MobileNetV3 into own file since it has a different head
 * Remove ChamNet, MobileNet V2/V1 since they will likely never be used here

## Models

Implemented models include:
  * EfficientNet AdvProp (B0-B8) (https://arxiv.org/abs/1911.09665)
  * EfficientNet (B0-B7) (https://arxiv.org/abs/1905.11946) -- validated, compat with TF weights
  * EfficientNet-EdgeTPU (S, M, L) (https://ai.googleblog.com/2019/08/efficientnet-edgetpu-creating.html) --validated w/ TF weights
  * EfficientNet-CondConv (https://arxiv.org/abs/1904.04971)
  * MixNet (https://arxiv.org/abs/1907.09595) -- validated, compat with TF weights
  * MNASNet B1, A1 (Squeeze-Excite), and Small (https://arxiv.org/abs/1807.11626)
  * MobileNet-V3 (https://arxiv.org/abs/1905.02244) -- native PyTorch model trained better than paper spec, ported TF weights
  * FBNet-C (https://arxiv.org/abs/1812.03443)
  * Single-Path NAS (https://arxiv.org/abs/1904.02877) -- pixel1 variant
    
I originally implemented and trained some these models with code [here](https://github.com/rwightman/pytorch-image-models), this repository contains just the GenEfficientNet models, validation, and associated ONNX/Caffe2 export code. 

## Pretrained

I've managed to train several of the models to accuracies close to or above the originating papers and official impl. My training code is here: https://github.com/rwightman/pytorch-image-models


|Model | Prec@1 (Err) | Prec@5 (Err) | Param#(M) | MAdds(M) | Image Scaling | Resolution | Crop |
|---|---|---|---|---|---|---|---|
| efficientnet_b3 | 81.866 (18.134) | 95.836 (4.164) | 12.23 | TBD | bicubic | 320 | 1.0 |
| efficientnet_b3 | 81.508 (18.492) | 95.672 (4.328) | 12.23 | TBD | bicubic | 300 | 0.903 |
| mixnet_xl | 81.074 (18.926) | 95.282 (4.718) | 11.90 | TBD | bicubic | 256 | 1.0 |
| efficientnet_b2 | 80.612 (19.388) | 95.318 (4.682) | 9.1 | TBD | bicubic | 288 | 1.0 |
| mixnet_xl | 80.476 (19.524) | 94.936 (5.064) | 11.90 | TBD | bicubic | 224 | 0.875 |
| efficientnet_b2 | 80.288 (19.712) | 95.166 (4.834) | 9.1 | 1003 | bicubic | 260 | 0.890 |
| mixnet_l | 78.976 (21.024 | 94.184 (5.816) | 7.33 | TBD | bicubic | 224 | 0.875 |
| efficientnet_b1 | 78.692 (21.308) | 94.086 (5.914) | 7.8 | 694 | bicubic | 240 | 0.882 |
| efficientnet_b0 | 77.698 (22.302) | 93.532 (6.468) | 5.3 | 390 | bicubic | 224 | 0.875 |
| mixnet_m | 77.256 (22.744) | 93.418 (6.582) | 5.01 | 353 | bicubic | 224 | 0.875 |
| mixnet_s | 75.988 (24.012) | 92.794 (7.206) | 4.13 | TBD | bicubic | 224 | 0.875 |
| mobilenetv3_rw | 75.634 (24.366) | 92.708 (7.292) | 5.5 | 219 | bicubic | 224 | 0.875 |
| mnasnet_a1 | 75.448 (24.552) | 92.604 (7.396) | 3.9 | 312 | bicubic | 224 | 0.875 |
| fbnetc_100 | 75.124 (24.876) | 92.386 (7.614) | 5.6 | 385 | bilinear | 224 | 0.875 |
| mnasnet_b1 | 74.658 (25.342) | 92.114 (7.886) | 4.4 | 315 | bicubic | 224 | 0.875 |
| spnasnet_100 | 74.084 (25.916)  | 91.818 (8.182) | 4.4 | TBV | bilinear | 224 | 0.875 |


More pretrained models to come...


## Ported Weights

The weights ported from Tensorflow checkpoints for the EfficientNet models do pretty much match accuracy in Tensorflow once a SAME convolution padding equivalent is added, and the same crop factors, image scaling, etc (see table) are used via cmd line args.

**IMPORTANT:** 
* Tensorflow ported weights for EfficientNet AdvProp (AP), EfficientNet EdgeTPU, EfficientNet-CondConv, and MobileNet-V3 models use Inception style (0.5, 0.5, 0.5) for mean and std.
* Enabling the Tensorflow preprocessing pipeline with `--tf-preprocessing` at validation time will improve scores by 0.1-0.5%, very close to original TF impl.

To run validation for tf_efficientnet_b5:
`python validate.py /path/to/imagenet/validation/ --model tf_efficientnet_b5 -b 64 --img-size 456 --crop-pct 0.934 --interpolation bicubic`

To run validation w/ TF preprocessing for tf_efficientnet_b5:
`python validate.py /path/to/imagenet/validation/ --model tf_efficientnet_b5 -b 64 --img-size 456 --tf-preprocessing`

To run validation for a model with Inception preprocessing, ie EfficientNet-B8 AdvProp:
`python validate.py /path/to/imagenet/validation/ --model tf_efficientnet_b8_ap -b 48 --num-gpu 2 --img-size 672 --crop-pct 0.954 --mean 0.5 --std 0.5`

|Model | Prec@1 (Err) | Prec@5 (Err) | Param # | Image Scaling  | Image Size | Crop | 
|---|---|---|---|---|---|---|
| tf_efficientnet_b8_ap *tfp | 85.436 (14.564) | 97.272 (2.728) | 87.4 | bicubic | 672 | N/A |
| tf_efficientnet_b8_ap      | 85.368 (14.632) | 97.294 (2.706) | 87.4 | bicubic | 672 | 0.954 |
| tf_efficientnet_b7_ap *tfp | 85.154 (14.846) | 97.244 (2.756) | 66.35 | bicubic | 600 | N/A |
| tf_efficientnet_b7_ap      | 85.118 (14.882) | 97.252 (2.748) | 66.35 | bicubic | 600 | 0.949 |
| tf_efficientnet_b7 *tfp | 84.940 (15.060) | 97.214 (2.786) | 66.35 | bicubic | 600 | N/A |
| tf_efficientnet_b7      | 84.932 (15.068) | 97.208 (2.792) | 66.35 | bicubic | 600 | 0.949 |
| tf_efficientnet_b6_ap      | 84.786 (15.214) | 97.138 (2.862) | 43.04 | bicubic | 528 | 0.942 |
| tf_efficientnet_b6_ap *tfp | 84.760 (15.240) | 97.124 (2.876) | 43.04 | bicubic | 528 | N/A |
| tf_efficientnet_b5_ap *tfp | 84.276 (15.724) | 96.932 (3.068) | 30.39 | bicubic | 456 | N/A |
| tf_efficientnet_b5_ap      | 84.254 (15.746) | 96.976 (3.024) | 30.39 | bicubic | 456 | 0.934 |
| tf_efficientnet_b6 *tfp  | 84.140 (15.860) | 96.852 (3.148) | 43.04 | bicubic | 528 | N/A |
| tf_efficientnet_b6       | 84.110 (15.890) | 96.886 (3.114) | 43.04 | bicubic | 528 | 0.942 |
| tf_efficientnet_b5 *tfp  | 83.822 (16.178) | 96.756 (3.244) | 30.39 | bicubic | 456 | N/A |
| tf_efficientnet_b5       | 83.812 (16.188) | 96.748 (3.252) | 30.39 | bicubic | 456 | 0.934 |
| tf_efficientnet_b4_ap *tfp | 83.278 (16.722) | 96.376 (3.624) | 19.34 | bicubic | 380 | N/A |
| tf_efficientnet_b4_ap      | 83.248 (16.752) | 96.388 (3.612) | 19.34 | bicubic | 380 | 0.922 |
| tf_efficientnet_b4       | 83.022 (16.978) | 96.300 (3.700) | 19.34 | bicubic | 380 | 0.922 |
| tf_efficientnet_b4 *tfp  | 82.948 (17.052) | 96.308 (3.692) | 19.34 | bicubic | 380 | N/A |
| tf_efficientnet_b3_ap *tfp | 81.882 (18.118) | 95.662 (4.338) | 12.23 | bicubic | 300 | N/A |
| tf_efficientnet_b3_ap      | 81.828 (18.172) | 95.624 (4.376) | 12.23 | bicubic | 300 | 0.903 |
| tf_efficientnet_b3       | 81.636 (18.364) | 95.718 (4.282) | 12.23 | bicubic | 300 | 0.903 |
| tf_efficientnet_b3 *tfp  | 81.576 (18.424) | 95.662 (4.338) | 12.23 | bicubic | 300 | N/A |
| tf_efficientnet_el       | 80.534 (19.466) | 95.190 (4.810) | 10.59 | bicubic | 300 | 0.903 |
| tf_efficientnet_el *tfp  | 80.476 (19.524) | 95.200 (4.800) | 10.59 | bicubic | 300 | N/A |
| tf_efficientnet_b2_ap *tfp | 80.420 (19.580) | 95.040 (4.960) | 9.11 | bicubic | 260 | N/A |
| tf_efficientnet_b2_ap    | 80.306 (19.694) | 95.028 (4.972) | 9.11 | bicubic | 260 | 0.890 |
| tf_efficientnet_b2 *tfp  | 80.188 (19.812) | 94.974 (5.026) | 9.11 | bicubic | 260 | N/A |
| tf_efficientnet_b2       | 80.086 (19.914) | 94.908 (5.092) | 9.11 | bicubic | 260 | 0.890 |
| tf_efficientnet_b1_ap *tfp | 79.532 (20.468) | 94.378 (5.622) | 7.79 | bicubic | 240  N/A |
| tf_efficientnet_cc_b1_8e *tfp | 79.464 (20.536)| 94.492 (5.508) | 39.7 | bicubic | 240 | 0.88 |
| tf_efficientnet_cc_b1_8e | 79.298 (20.702) | 94.364 (5.636) | 39.7 | bicubic | 240 | 0.88 |
| tf_efficientnet_b1_ap    | 79.278 (20.722) | 94.308 (5.692) | 7.79 | bicubic | 240 | 0.88 |
| tf_efficientnet_b1 *tfp  | 79.172 (20.828) | 94.450 (5.550) | 7.79 | bicubic | 240  N/A |
| tf_efficientnet_em *tfp  | 78.958 (21.042) | 94.458 (5.542) | 6.90 | bicubic | 240 | N/A |
| tf_mixnet_l *tfp         | 78.846 (21.154) | 94.212 (5.788) | 7.33 | bilinear | 224 | N/A |
| tf_efficientnet_b1       | 78.826 (21.174) | 94.198 (5.802) | 7.79 | bicubic | 240 | 0.88 |
| tf_mixnet_l              | 78.770 (21.230) | 94.004 (5.996) | 7.33 | bicubic | 224 | 0.875 |
| tf_efficientnet_em       | 78.742 (21.258) | 94.332 (5.668) | 6.90 | bicubic | 240 | 0.875 |
| tf_efficientnet_cc_b0_8e *tfp | 78.314 (21.686) | 93.790 (6.210) | 24.0 | bicubic | 224 | 0.875 |
| tf_efficientnet_cc_b0_8e | 77.908 (22.092) | 93.656 (6.344) | 24.0 | bicubic | 224 | 0.875 |
| tf_efficientnet_cc_b0_4e *tfp | 77.746 (22.254) | 93.552 (6.448) | 13.3 | bicubic | 224 | 0.875 |
| tf_efficientnet_cc_b0_4e | 77.304 (22.696) | 93.332 (6.668) | 13.3 | bicubic | 224 | 0.875 |
| tf_efficientnet_es *tfp  | 77.616 (22.384) | 93.750 (6.250) | 5.44 | bicubic | 224 | N/A |
| tf_efficientnet_b0_ap *tfp | 77.514 (22.486) | 93.576 (6.424) | 5.29  | bicubic | 224 | N/A |
| tf_efficientnet_es       | 77.264 (22.736) | 93.600 (6.400) | 5.44 | bicubic | 224 | N/A |
| tf_efficientnet_b0 *tfp  | 77.258 (22.742) | 93.478 (6.522) | 5.29  | bicubic | 224 | N/A |
| tf_efficientnet_b0_ap    | 77.084 (22.916) | 93.254 (6.746) | 5.29  | bicubic | 224 | 0.875 |
| tf_mixnet_m *tfp         | 77.072 (22.928) | 93.368 (6.632) | 5.01 | bilinear | 224 | N/A |
| tf_mixnet_m              | 76.950 (23.050) | 93.156 (6.844) | 5.01 | bicubic | 224 | 0.875 |
| tf_efficientnet_b0       | 76.848 (23.152) | 93.228 (6.772) | 5.29  | bicubic | 224 | 0.875 |
| tf_mixnet_s *tfp         | 75.800 (24.200) | 92.788 (7.212) | 4.13 | bilinear | 224 | N/A |
| tf_mobilenetv3_large_100 *tfp | 75.768 (24.232) | 92.710 (7.290) | 5.48 | bilinear | 224 | N/A |
| tf_mixnet_s              | 75.648 (24.352) | 92.636 (7.364) | 4.13 | bicubic | 224 | 0.875 |
| tf_mobilenetv3_large_100 | 75.516 (24.484) | 92.600 (7.400) | 5.48 | bilinear | 224 | 0.875 |
| tf_mobilenetv3_large_075 *tfp | 73.730 (26.270) | 91.616 (8.384) | 3.99 | bilinear | 224 |N/A |
| tf_mobilenetv3_large_075 | 73.442 (26.558) | 91.352 (8.648) | 3.99 | bilinear | 224 | 0.875 |
| tf_mobilenetv3_large_minimal_100 *tfp | 72.678 (27.322) | 90.860 (9.140) | 3.92 | bilinear | 224 | N/A |
| tf_mobilenetv3_large_minimal_100 | 72.244 (27.756) | 90.636 (9.364) | 3.92 | bilinear | 224 | 0.875 |
| tf_mobilenetv3_small_100 *tfp | 67.918 (32.082) | 87.958 (12.042 | 2.54 | bilinear | 224 | N/A |
| tf_mobilenetv3_small_100 | 67.918 (32.082) | 87.662 (12.338) | 2.54 | bilinear | 224 | 0.875 |
| tf_mobilenetv3_small_075 *tfp | 66.142 (33.858) | 86.498 (13.502) | 2.04 | bilinear | 224 | N/A |
| tf_mobilenetv3_small_075 | 65.718 (34.282) | 86.136 (13.864) | 2.04 | bilinear | 224 | 0.875 |
| tf_mobilenetv3_small_minimal_100 *tfp | 63.378 (36.622) | 84.802 (15.198) | 2.04 | bilinear | 224 | N/A |
| tf_mobilenetv3_small_minimal_100 | 62.898 (37.102) | 84.230 (15.770) | 2.04 | bilinear | 224 | 0.875 |


*tfp models validated with `tf-preprocessing` pipeline

Google tf and tflite weights ported from official Tensorflow repositories
* https://github.com/tensorflow/tpu/tree/master/models/official/mnasnet
* https://github.com/tensorflow/tpu/tree/master/models/official/efficientnet
* https://github.com/tensorflow/models/tree/master/research/slim/nets/mobilenet

## Usage

### Environment

All development and testing has been done in Conda Python 3 environments on Linux x86-64 systems, specifically Python 3.6.x and 3.7.x.

Users have reported that a Python 3 Anaconda install in Windows works. I have not verified this myself.

PyTorch versions 1.2. 1.3.1, 1.4 have been tested with this code.

I've tried to keep the dependencies minimal, the setup is as per the PyTorch default install instructions for Conda:
```
conda create -n torch-env
conda activate torch-env
conda install -c pytorch pytorch torchvision cudatoolkit=10
```

### PyTorch Hub

Models can be accessed via the PyTorch Hub API

```
>>> torch.hub.list('rwightman/gen-efficientnet-pytorch')
['efficientnet_b0', ...]
>>> model = torch.hub.load('rwightman/gen-efficientnet-pytorch', 'efficientnet_b0', pretrained=True)
>>> model.eval()
>>> output = model(torch.randn(1,3,224,224))
```

### Pip
This package can be installed via pip.

Install (after conda env/install):
```
pip install geffnet
```

Eval use:
```
>>> import geffnet
>>> m = geffnet.create_model('mobilenetv3_rw', pretrained=True)
>>> m.eval()
```

Train use:
```
>>> import geffnet
>>> # models can also be created by using the entrypoint directly
>>> m = geffnet.efficientnet_b2(pretrained=True, drop_rate=0.25, drop_connect_rate=0.2)
>>> m.train()
```

Create in a nn.Sequential container, for fast.ai, etc:
```
>>> import geffnet
>>> m = geffnet.mixnet_l(pretrained=True, drop_rate=0.25, drop_connect_rate=0.2, as_sequential=True)
```

### Exporting

Scripts to export models to ONNX and then to Caffe2 are included, along with a Caffe2 script to verify.

As an example, to export the MobileNet-V3 pretrained model and then run an Imagenet validation:
```
python onnx_export.py --model tf_mobilenetv3_large_100 ./mobilenetv3_100.onnx
python onnx_optimize.py ./mobilenetv3_100.onnx --output ./mobilenetv3_100-opt.onnx
python onnx_to_caffe.py ./mobilenetv3_100-opt.onnx --c2-prefix mobilenetv3
python caffe2_validate.py /imagenet/validation/ --c2-init ./mobilenetv3.init.pb --c2-predict ./mobilenetv3.predict.pb --interpolation bicubic
```
**NOTE** the TF ported weights with the 'SAME' conv padding activated cannot be exported to ONNX unless `_EXPORTABLE` flag in `config.py` is set to True. Use `config.set_exportable(True)` as in the updated `onnx_export.py` example script.


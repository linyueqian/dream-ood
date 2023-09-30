# DREAM-OOD

This is the source code accompanying the paper [***Dream the Impossible: Outlier Imagination with Diffusion Models***](https://arxiv.org/pdf/2309.13415) by Xuefeng Du, Yiyou Sun, Xiaojin Zhu, and Yixuan Li


The codebase is heavily based on [Stable Diffusion](https://github.com/CompVis/stable-diffusion).

## TODO
Upload all the checkpoints and embeddings/images.

## Ads 

Check out our latent-based outlier synthesis papers in ICLR'22 [VOS](https://github.com/deeplearning-wisc/vos) and ICLR'23 [NPOS](https://github.com/deeplearning-wisc/npos) if you are interested!

## Requirements
A suitable [conda](https://conda.io/) environment named `dreamood` can be created
and activated with:

```
conda env create -f environment.yaml
conda activate dreamood
```
Please also install [Xformers](https://github.com/facebookresearch/xformers).

## Dataset Preparation

**ImageNet-100**

* Download the full ImageNet dataset from the official website [here](https://www.image-net.org/).

* Preprocess the dataset to get ImageNet-100 by running:

```
python scripts/generate_in100.py --source_folder xxx --target_folder xxx
```
where "--source_folder" is the address of the full ImageNet dataset and "--target_folder" specifies the address of the dataset you want to store.

**CIFAR-100**

* The dataloader will download it automatically when first running the programs.

**OOD datasets**


* The OOD datasets with ImageNet-100 as in-distribution are 4 OOD datasets from iNaturalist, SUN, Places, and Textures, which contain the de-duplicated concepts overlapped with ImageNet.
* The OOD datasets with CIFAR-100 as in-distribution are 5 OOD datasets, i.e., SVHN, PLACES365, LSUN, ISUN, TEXTURES.
* Please refer to Part 1 and 2 of the codebase [here](https://github.com/deeplearning-wisc/knn-ood). 

**Datasets for evaluating model generalization**

Please download [IMAGENET-A](https://github.com/hendrycks/natural-adv-examples) and [IMAGENET-V2](https://github.com/modestyachts/ImageNetV2) and process the dataset by running (you need change the address of the datasets on your own):
```
python scripts/process_imagenetv2_and_a.py
```



## Training


**1. Learning the text-conditioned latent space**

Please execute the following in the command shell on ImageNet-100:
```
python scripts/pretrain_in100.py
```
Please execute the following in the command shell on CIFAR-100:
```
python scripts/pretrain_cifar100.py
```
After training, it will generate ID feature embeddings for outlier/inlier embedding sampling.

* Pretrained models for [ImageNet-100]() and [Cifar-100]().

**2. Generate the inlier/outlier embeddings**

Please execute the following in the command shell on ImageNet-100:
```
python scripts/get_embed_in100.py
```
Please execute the following in the command shell on CIFAR-100:
```
python scripts/get_embed_cifar100.py
```

After this step, you will see the generated inlier/outlier embedding in the root directory.

* Pretrained embeddings: [inliers for ImageNet-100](), [outlier for ImageNet-100]() and [outliers for Cifar-100]().

**3. Synthesizing outliers in the pixel space**

First, please download the Stable Diffusion 1.4 model [here](https://huggingface.co/CompVis/stable-diffusion-v-1-4-original/tree/main).

Please execute the following in the command shell on different datasets:

```
python scripts/dream_ood.py --plms \
--n_iter 50 --n_samples 3 \
--outdir /nobackup-fast/txt2img-samples-in100-demo/ \
--loaded_embedding /nobackup-slow/dataset/my_xfdu/diffusion/outlier_npos_embed.npy\
--ckpt /nobackup-slow/dataset/my_xfdu/diffusion/sd-v1-4.ckpt \
--id_data in100 \
--skip_grid
```
* "--loaded_embedding" means the address of the saved inlier/outlier embeddings obtained by the previous step.
* "--outdir" denotes the address you want to save the generated outlier images.
* "--n_iter"/"--n_samples" control the number of images you generate in each step and the number of steps for generation.
* "--id_data" can be chozen between in100 and cifar100.
* For generating 100K images, you can specify the n_iter and n_samples such as n_iter=2,500 and n_samples=4. Consider generating on multiplt GPUs for speed up.
* Generated images: [outlier image for IN100](), [inlier image for IN100]() and [outlier image for cifar100]().


**4. Training with the generated outliers in the pixel space**

Please execute the following in the command shell for OOD detection on ImageNet-100:
```
python scripts/train_ood_det_in100.py --my_info xxx --load xxx
```
* "--my_info" denotes the name of the folder that contains the generated datasets in the previous step.

Note that in order to save time, we use a [pretrained model]() for initialization, which is trained using the cross-entropy loss.

Please execute the following in the command shell for OOD detection on Cifar-100:
```
python scripts/train_ood_det_cifar100.py --my_info xxx
```
Here the model is trained from scratch.

* Pretrained models: [IN100](), [CIFAR-100]().

**5. Training with the generated inliers in the pixel space**
Please execute the following in the command shell for generalization on ImageNet-100:
```
python scripts/train_gene_in100.py 
```
* Pretrained models: [IN100]().

## Test-time OOD detection
Please execute the following in the command shell with ImageNet-100 as in-distribution:
```
python scripts/test_ood_in100.py --load xxx
```
where "--load" specifies the address of the saved models.  

Please execute the following in the command shell with Cifar-100 as in-distribution:
```
python scripts/test_ood_cifar100.py --load xxx
```
## Test-time ID generalization
Please execute the following in the command shell with ImageNet-100:
```
python scripts/test_ood_in100_robustness.py --load xxx
```




## Citation ##
If you found any part of this code is useful in your research, please consider citing our paper:

```
  @inproceedings{du2023dream,
      title={Dream the Impossible: Outlier Imagination with Diffusion Models}, 
      author={Xuefeng Du and Yiyou Sun and Xiaojin Zhu and Yixuan Li },
      booktitle={Advances in Neural Information Processing Systems},
      year = {2023}
}
```






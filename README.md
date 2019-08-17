# VisualBERT: A Simple and Performant Baseline for Vision and Language

This repository contains code for the paper [VisualBERT: A Simple and Performant Baseline for Vision and Language (arxiv)](https://arxiv.org/abs/1908.03557).

The repository is still under development. Please open up issues if you have any questions or comments. 

In `pytorch_pretrained_bert` is a modified version of an early clone of HuggingFace's Pytorch BERT. The core part of VisualBERT is implemented mainly by modifying `modeling.py`. Two wrapper models are implemented in `models/model.py`, and code for loading different datasets are in `dataloaders`, where AllenNLP's `Field` and `Instance` are extensively used for wrapping data.

I borrowed and modified code from several repositeries, including but not limited to: [R2C](https://github.com/rowanz/r2c), [Pythia](https://github.com/facebookresearch/pythia), [HugginFace BERT](https://github.com/huggingface/pytorch-transformers), [BAN](https://github.com/jnhwkim/ban-vqa), [Bottom-up and Top-down Attention](https://github.com/peteanderson80/bottom-up-attention), [AllenNLP](https://github.com/allenai/allennlp). I would like to expresse my gratitdue for authors of these repositeries.

# Dependencies

## Basic Dependency

Dependencies of this repository are similar to those of R2C. If you don't need to extract image features yourself or run on VCR, below are basic dependencies needed (assuming you are using a fresh conda environment):

```
conda install numpy pyyaml setuptools cmake cffi tqdm pyyaml scipy ipython mkl mkl-include cython typing h5py pandas nltk spacy numpydoc scikit-learn jpeg


#Please check your cuda version using `nvcc --version` and make sure the cuda version matches the cudatoolkit version.
conda install pytorch torchvision cudatoolkit=XXX -c pytorch


# Below is the way to install allennlp recommanded in R2C. But in my experience, directly installing allennlp seems also okay.
pip install -r allennlp-requirements.txt
pip install --no-deps allennlp==0.8.0
python -m spacy download en_core_web_sm
pip install attrdict
pip install pycocotools
pip install commentjson
```

## Dependency for extracting image features

Only install if you want to run on VCR or extract image features on your own.

1. A special version of pytorch vision that has ROIAlign layer:
```
pip install git+git://github.com/pytorch/vision.git@24577864e92b72f7066e1ed16e978e873e19d13d
```
2. [Detectron](https://github.com/facebookresearch/Detectron/)


## Troubleshooting
Below are some problems I have had when installing these dependencies:

1. pyyaml version.

Detectron might break when the pyyaml version is too high (not sure if this has been fixed now).
Solution: install a lower version `pip install pyyaml==3.12`. (https://github.com/facebookresearch/Detectron/issues/840, https://github.com/pypa/pip/issues/5247)

2. Error when importing torchvision or ROIAlign layer.

Most likely version mismatch between cudatoolkit and cuda verison. Please specify the correct cudatoolkit verison in `conda install pytorch torchvision cudatoolkit=XXX -c pytorch`.

3. Segmentation fault when runing ResNet50 detector in VCR.

Most likely GCC version is too low. Need GCC version >= 5.0. 

`conda install -c psi4 gcc-5` seems to solve the problem. (https://github.com/facebookresearch/maskrcnn-benchmark/blob/master/TROUBLESHOOTING.md)


# Running the code

Assume that the folder XX is the parent directory of the code directory. 
```
export PYTHONPATH=$PYTHONPATH:XX/visualbert/
export PYTHONPATH=$PYTHONPATH:XX/

cd XX/visualbert/models/

CUDA_VISIBLE_DEVICES=XXXX python train.py -folder [where you want to save the logs/models] -config XX/visualbert/configs/YYYY
```
In `visualbert/configs` are configs for different models on different datasets. Please change the data path (`data_root`) and model path(`restore_bin`, which is the path the model that you want to initialize from) in the config to match your local setting.

## NLVR2

### Prepare Data

Download our pre-computed features to a folder X_NLVR ([Train](https://drive.google.com/file/d/1iK9CDfxZ4ejKRWOIItLhD8-sgw78ld7w/view?usp=sharing), [Val](https://drive.google.com/file/d/13rFujBIBr6PLnG5A5i8WJJVT52RPYH9j/view?usp=sharing), [Test-Public](https://drive.google.com/file/d/1RTXZCK_kbFkqOeBnZ5wOAyDSzlmuaKRx/view?usp=sharing)). The image features are from a model from Detectron (e2e_mask_rcnn_R-101-FPN_2x, model_id: 35861858). For downloading from Google Drive links in the command line, check out https://github.com/gdrive-org/gdrive.

Then download the three json files from [the NLVR github page](https://github.com/lil-lab/nlvr/tree/master/nlvr2/data) into X_NLVR.

For COCO pre-training, first download COCO caption annotations to a folder X_COCO.

```
cd X_COCO
wget http://images.cocodataset.org/annotations/annotations_trainval2014.zip
unzip annotations_trainval2014.zip
```

Then download COCO image features to X_COCO. [Train](https://drive.google.com/file/d/1F-LSQhpKleV4nmiKMjQvpHMS2gkbK3bY/view?usp=sharing), [Val](https://drive.google.com/file/d/1cZjPob3YqfM46LaWY3-Ky12claxeXbWi/view?usp=sharing).

### COCO Pre-training
The corresponding config is `visualbert/configs/nlvr2/coco-pre-train.json`. [Model checkpoint](https://drive.google.com/file/d/1QvivVfRsRF518OQSQNaN7aFk6eQ43vP_/view?usp=sharing).

### Task-specific Pre-training
The corresponding config is `visualbert/configs/nlvr2/pre-train.json`. [Model checkpoint](https://drive.google.com/file/d/1Z19G_rAuKn0TQ5Cj-KCcavBKSwBhvxGq/view?usp=sharing).

### Fine-tuning
The corresponding config is `visualbert/configs/nlvr2/fine-tune.json`. [Model checkpoint](https://drive.google.com/file/d/1GCV6woBnWY09JhjtLOXyKUhuFiQz9L5U/view?usp=sharing).



## VQA

### Prepare Data
The image features and VQA data are from Pythia. Assuming the data is stored in X_COCO.

```
cd X_COCO
cd data
wget https://dl.fbaipublicfiles.com/pythia/data/vocabulary_vqa.txt
wget https://dl.fbaipublicfiles.com/pythia/data/answers_vqa.txt
wget https://dl.fbaipublicfiles.com/pythia/data/imdb.tar.gz
gunzip imdb.tar.gz 
tar -xf imdb.tar

wget https://dl.fbaipublicfiles.com/pythia/features/detectron_fix_100.tar.gz
gunzip detectron_fix_100.tar.gz
tar -xf detectron_fix_100.tar
rm -f detectron_fix_100.tar
```

### COCO Pre-training

The corresponding config is `visualbert/configs/vqa/coco-pre-train.json`. [Model checkpoint](https://drive.google.com/file/d/1tgYovjB6MZZlqdSAOPzB8bZqnFezWNBO/view?usp=sharing).

### Task-specific Pre-training
The corresponding config is `visualbert/configs/vqa/pre-train.json`. [Model checkpoint](https://drive.google.com/file/d/1kuPr187zWxSJbtCbVW87XzInXltM-i9Y/view?usp=sharing).

### Fine-tuning
The corresponding config is `visualbert/configs/vqa/fine-tune.json`. [Model checkpoint](https://drive.google.com/file/d/19FpfLYo3rwv0eybUvfkDMCoivyL4XLqB/view?usp=sharing).



## VCR
Coming soon!

## Flickr30K
Coming soon!

## Extract image features on your own
Coming soon!

## Evaluation
Coming soon!

# Visualization
Coming soon!

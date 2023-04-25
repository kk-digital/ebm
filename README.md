# Implicit Generation and Generalization in Energy Based Models

Code for [Implicit Generation and Generalization in Energy Based Models](https://arxiv.org/pdf/1903.08689.pdf). Blog post can be found [here](https://openai.com/blog/energy-based-models/) and website with pretrained models can be found [here](https://sites.google.com/view/igebm/home).

## Requirements

To install the prerequisites for the project run 
```
pip install -r requirements.txt
mkdir sandbox_cachedir
```

Download all [pretrained models](https://sites.google.com/view/igebm/home) and unzip into the folder cachedir.

## Download Datasets

For MNIST and CIFAR-10 datasets, the code will directly download the data.

For ImageNet 128x128 dataset, download the TFRecords of the Imagenet dataset by running the following command

```
for i in $(seq -f "%05g" 0 1023)
do
  wget https://[deprecated]/data/imagenet/train-$i-of-01024
done

for i in $(seq -f "%05g" 0 127)
do
  wget https://[deprecated]/data/imagenet/validation-$i-of-00128
done

wget https://[deprecated]/data/imagenet/index.json
```

For Imagenet 32x32 dataset, download the Imagenet 32x32 dataset and unzip by running the following command

```
wget https://[deprecated]/data/imagenet32/Imagenet32_train.zip
wget https://[deprecated]/data/imagenet32/Imagenet32_val.zip
```

For dSprites dataset, download the dataset by running

```
wget https://github.com/deepmind/dsprites-dataset/blob/master/dsprites_ndarray_co1sh3sc6or40x32y32_64x64.npz?raw=true
```

## Training

To train on different datasets:

For CIFAR-10 Unconditional

```
python train.py --exp=cifar10_uncond --dataset=cifar10 --num_steps=60 --batch_size=128 --step_lr=10.0 --proj_norm=0.01 --zero_kl --replay_batch --large_model
```

For CIFAR-10 Conditional

```
python train.py --exp=cifar10_cond --dataset=cifar10 --num_steps=60 --batch_size=128 --step_lr=10.0 --proj_norm=0.01 --zero_kl --replay_batch --cclass
```

For ImageNet 32x32 Conditional

```
python train.py --exp=imagenet_cond --num_steps=60  --wider_model --batch_size=32 step_lr=10.0 --proj_norm=0.01 --replay_batch --cclass --zero_kl --dataset=imagenet --imagenet_path=<imagenet32x32 path>
```

For ImageNet 128x128 Conditional

```
python train.py --exp=imagenet_cond --num_steps=50 --batch_size=16 step_lr=100.0 --replay_batch --swish_act --cclass --zero_kl --dataset=imagenetfull --imagenet_datadir=<full imagenet path>
```

All code supports horovod execution, so model training can be increased substantially by using multiple different workers by running each command.
```
mpiexec -n <worker_num>  <command>
```

## Demo

The imagenet_demo.py file contains code to experiments with EBMs on conditional ImageNet 128x128. To generate a gif on sampling, you can run the command:

```
python imagenet_demo.py --exp=imagenet128_cond --resume_iter=2238000 --swish_act
```

The ebm_sandbox.py file contains several different tasks that can be used to evaluate EBMs, which are defined by different settings of task flag in the file. For example, to visualize cross class mappings in CIFAR-10, you can run:

```
python ebm_sandbox.py --task=crossclass --num_steps=40 --exp=cifar10_cond --resume_iter=74700
```


## Generalization

To test generalization to out of distribution classification for SVHN (with similar commands for other datasets)
```
python ebm_sandbox.py --task=mixenergy --num_steps=40 --exp=cifar10_large_model_uncond --resume_iter=121200 --large_model --svhnmix --cclass=False
```

To test classification on CIFAR-10 using a conditional model under either L2 or Li perturbations
```
python ebm_sandbox.py --task=label --exp=cifar10_wider_model_cond --resume_iter=21600 --lnorm=-1 --pgd=<number of pgd steps> --num_steps=10 --lival=<li bound value> --wider_model
```


## Concept Combination

To train EBMs on conditional dSprites dataset, you can train each model seperately on each conditioned latent in cond_pos, cond_rot, cond_shape, cond_scale, with an example command given below.

```
python train.py --dataset=dsprites --exp=dsprites_cond_pos --zero_kl --num_steps=20 --step_lr=500.0 --swish_act  --cond_pos --replay_batch -cclass
```

Once models are trained, they can be sampled from jointly by running

```
python ebm_combine.py --task=conceptcombine --exp_size=<exp_size> --exp_shape=<exp_shape> --exp_pos=<exp_pos> --exp_rot=<exp_rot> --resume_size=<resume_size> --resume_shape=<resume_shape> --resume_rot=<resume_rot> --resume_pos=<resume_pos>
```




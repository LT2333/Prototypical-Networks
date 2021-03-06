# Prototypical Networks for Few shot Learning in PyTorch
Simple alternative Implementation of Prototypical Networks for Few Shot Learning ([paper](https://arxiv.org/abs/1703.05175), [code](https://github.com/jakesnell/prototypical-networks)) in PyTorch.

## Prototypical Networks

As shown in the reference paper Prototypical Networks are trained to embed samples features in a vectorial space, in particular, at each episode (iteration), a number of samples for a subset of classes are selected and sent through the model, for each subset of class `c` a number of samples' features (`n_support`) are used to guess the prototype (their barycentre coordinates in the vectorial space) for that class, so then the distances between the remaining `n_query` samples and their class barycentre can be minimized.


## T-SNE 

After training, you can compute t-SNE for the original images with highest or lowest loss in the testing phase, do it by:

1. run train.py, this will generate data (an dictionary ordered by loss value) for visualization
2. if you want to check t-SNE for original images with highest loss, change the mode to "top" inside bottom of the code, otherwise, use "low". Then run:
```
python visual.py
```

The procedure of how we implemented this is: We evaluated our best model on the test dataset, which consists of 100 tasks. Then we have recorded the loss for each of them and found out the tasks with highest (or highest 5) loss and lowest loss for comparison.  For tasks with highest loss, we identified their ground-truth class. Here we are testing using 5-way-5shots models, the number of testing classes is 5. So there should be 5 clusters in a 2D t-SNE visualization. After we found these 5 classes, we traced back the path of the ground truth class and displayed 20 images from each class.

![Testing using 5-way-5shots models](https://github.com/WangTianduo/Prototypical-Networks/blob/master/Experiment_results/t-sne.png)

## Omniglot Dataset

Kudos to [@ludc](https://github.com/ludc) for his contribute: https://github.com/pytorch/vision/pull/46.
We will use the official dataset when it will be added to torchvision if it doesn't imply big changes to the code.

### Dataset splits

We implemented the Vynials splitting method as in [[Matching Networks for One Shot Learning](https://papers.nips.cc/paper/6385-matching-networks-for-one-shot-learning)]. That sould be the same method used in the paper (in fact I download the split files from the "offical" [repo](https://github.com/jakesnell/prototypical-networks/tree/master/data/omniglot/splits/vinyals)). We then apply the same rotations there described. In this way we should be able to compare results obtained by running this code with results described in the reference paper.

## Prototypical Batch Sampler

As described in its PyDoc, this class is used to generate the indexes of each batch for a prototypical training algorithm.

In particular, the object is instantiated by passing the list of the labels for the dataset, the sampler infers then the total number of classes and creates a set of indexes for each class ni the dataset. At each episode the sampler selects `n_classes` random classes and returns a number (`n_support` + `n_query`) of samples indexes for each one of the selected classes.

## Prototypical Loss

Compute the loss as in the cited paper, mostly inspired by [this code](https://github.com/jakesnell/prototypical-networks/blob/master/protonets/models/few_shot.py) by one of its authors.

In [`prototypical_loss.py`](src/prototypical_loss.py) both loss function and loss class à la PyTorch are implemented. 

The function takes in input the batch input from the model, samples' ground truths and the number `n_suppport` of samples to be used as support samples. Episode classes get infered from the target list, `n_support` samples get randomly extracted for each class, their class barycentres get computed, as well as the distances of each remaining samples' embedding from each class barycentre and the probability of each sample of belonging to each episode class get finmally computed; then the loss is then computed from the wrong predictions probabilities (for the query samples) as usual in classification problems.

## Training

Please note that the training code is here just for demonstration purposes. 

To train the Protonet on this task, cd into this repo's `src` root folder and execute:

    $ python train.py


The script takes the following command line options:

- `dataset_root`: the root directory where tha dataset is stored, default to `'../dataset'`

- `nepochs`: number of epochs to train for, default to `100`

- `learning_rate`: learning rate for the model, default to `0.001`

- `lr_scheduler_step`: StepLR learning rate scheduler step, default to `20`

- `lr_scheduler_gamma`: StepLR learning rate scheduler gamma, default to `0.5`

- `iterations`: number of episodes per epoch. default to `100`

- `classes_per_it_tr`: number of random classes per episode for training. default to `60`

- `num_support_tr`: number of samples per class to use as support for training. default to `5`

- `num_query_tr`: nnumber of samples per class to use as query for training. default to `5`

- `classes_per_it_val`: number of random classes per episode for validation. default to `5`

- `num_support_val`: number of samples per class to use as support for validation. default to `5`

- `num_query_val`: number of samples per class to use as query for validation. default to `15`

- `manual_seed`: input for the manual seeds initializations, default to `7`

- `cuda`: enables cuda (store `True`)

Running the command without arguments will train the models with the default hyperparamters values (producing results shown above).


## Performances (Omniglot Dataset)

We are trying to reproduce the reference paper performaces, we'll update here our best results. 

| Model | 1-shot (5-way Acc.) | 5-shot (5-way Acc.) | 1 -shot (20-way Acc.) | 5-shot (20-way Acc.)|
| --- | --- | --- | --- | --- |
| Reference Paper | 98.8% | 99.7% | 96.0% | 98.9%|
| This repo | 98.5%** | 99.6%* | 95.1%°| 98.6%°°|
| with Cosine Distance | 81.8% | 85.2% | 59.5% | 68.8% |
| Resnet50 | 97.9% | 99.3% | 92.0% | CUDA out of memory |


\* achieved using default parameters (using `--cuda` option)

\*\* achieved running `python train.py --cuda -nsTr 1 -nsVa 1`

° achieved running `python train.py --cuda -nsTr 1 -nsVa 1 -cVa 20`

°° achieved running `python train.py --cuda -nsTr 5 -nsVa 5 -cVa 20
`

## Performances (MiniImageNet Dataset)

Images from: https://drive.google.com/open?id=0B3Irx3uQNoBMQ1FlNXJsZUdYWEE

We are trying to reproduce the reference paper performaces, we'll update here our best results. 

| Model | 1-shot (5-way Acc.) | 5-shot (5-way Acc.) |
| --- | --- | --- |
| Reference Paper | 49.42% | 68.20%|
| This repo | 42.48%* | 64.7%** |

\* achieved by running `python train_miniImageNet.py --cuda -nsTr 1 -nsVa 1` (30 epochs)

\*\* achieved by running `python train_miniImageNet.py --cuda` (30 epochs)

## Compare the effect of number of training class for a fixed 'way' model
| Accuracy | Model | classes_per_it_tr | classes_per_it_val | 
| --- | --- | --- | --- | 
| 97.3% | 1-shot (5-way Acc.) | 60 | 5 | 
| 95.7% | 1-shot (5-way Acc.) | 25 | 5 | 
| 93.0% | 1-shot (5-way Acc.) | 10 | 5 | 
| 88.6% | 1-shot (5-way Acc.) | 5 | 5 | 

\* All of the models are trained under 10 epochs

## Compare the effect of number of supporting test samples for a fixed 'shot' model

| Accuracy | Model | num_support_tr | num_support_val | 
| --- | --- | --- | --- | 
| 97.3% | 1-shot (5-way Acc.) | 1 | 1 | 
| 99.3% | 1-shot (5-way Acc.) | 1 | 5 | 
| 99.3% | 5-shot (5-way Acc.) | 5 | 5 | 
| 96.4% | 5-shot (5-way Acc.) | 5 | 1 | 

\* All of the models are trained under 10 epochs 




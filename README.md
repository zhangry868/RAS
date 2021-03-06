# Adversarial Learning of a Sampler Based on an Unnormalized Distribution

<!--- 
<img src="figs/under_construction.png" width="200">  Still Under Construction...
-->

The **RAS** (_Referenced-based Adversarial Sampling_) algorithm is proposed to enable adversarial learning applicable to general unnormalized distribution sampling, with demonstrations on constrained domain sampling and soft Q-learning. This repository contains source code to reproduce the results presented in the paper [Adversarial Learning of a Sampler Based on an Unnormalized Distribution](https://arxiv.org/abs/1901.00612) (AISTATS 2019):

```
@inproceedings{Li_RAS_2019_AISTATS,
  title={Adversarial Learning of a Sampler Based on an Unnormalized Distribution},
  author={Chunyuan Li, Ke Bai, Jianqiao Li, Guoyin Wang, Changyou Chen, Lawrence Carin},
  booktitle={AISTATS},
  year={2019}
}
```

## Introduction

### Comparison of **RAS** and **GAN** learning scenarios

Learning a neural sampler **q** to approximate the target distribution **p**, where only the latter's unnormalized form **u** or empirical samples **p'** is available, respectively.

|  | RAS  |   GAN 
|-------------------------|:-------------------------:|:-------------------------:
| **Illustration** | ![](/figs/scheme/ras_scheme.png)  |   ![](/figs/scheme/gan_scheme.png)
| **Method** | We propose the “reference” **p_r** to bridge neural samples **q**  and unnormalized form **u**, making the evaluations of both F_1 and F_2 terms feasible. | Directly matching neural samples **q** to empirical samples **p'**
| **Setup** | Learning from unnormalized form **u**  | Learning from empirical samples **p'**  
| **Generator** |  ![](https://latex.codecogs.com/gif.latex?\log[\frac{u(x)}{q_{\theta}(x)}&space;]=&space;\underbrace{&space;\log&space;\big[\frac{&space;p_{r}(x)&space;}{&space;q_{\theta}&space;(x)&space;}&space;\big]}_{\mathcal{F}_1}&space;&plus;&space;\underbrace{&space;\log&space;\big[\frac{&space;u&space;(x)&space;}{p_{r}(x)&space;}\big]&space;}_{\mathcal{F}_2})  | ![](https://latex.codecogs.com/gif.latex?\log[\frac{p^{\prime}(x)}{q_{\theta}(x)}&space;])
| **Discriminator** | **q** vs **p_r** | **q** vs **p'**
| **Application to reinforcement learning** | Learning to take optimal actions based on Q-functions | [**GAIL**](https://arxiv.org/abs/1606.03476): Learning to take optimal actions based on expert sample trajectories (_a.k.a._ Imitation learning)

### Discussion
1. In many applications (_e.g._ Soft Q-learining), only **u** is known, from which we are inerested in drawing samples efficiently
2. The choice of **p_r** has an effect on learning; It should be carefully chosen.


## Contents
There are three steps to use this codebase to reproduce the results in the paper.

1. [Dependencies](#dependencies)
2. [Experiments](#experiments)

    2.1. [Adversarial Soft Q-learning](#adversarial-soft-q-learning)
    
    2.2. [Constrained Domain Sampling](#constrained-domain-sampling)
    
    2.3. [Entropy Regularization](#entropy-regularization) 

3. [Reproduce paper figure results](#reproduce-paper-figure-results) 

## Dependencies

This code is based on Python 2.7, with the main dependencies being [TensorFlow==1.7.0](https://www.tensorflow.org/). Additional dependencies for running experiments are: `numpy`, `cPickle`, `scipy`, `math`, `gensim`. 

## Adversarial Soft Q-learning

We consider the following environments: `Hopper`, `Half-cheetah`, `Ant`, `Walker`, `Swimmer` and `Humanoid`. All soft q-learning code is at [`sql`](/sql): 

To run:

    python mujoco_all_sql.py --env Hopper

It takes the following options (among others) as arguments:

- `--env` It specifies the _MuJoCo_/_rllab_ environment; default `Hopper`.  
- `--log_dir` Address to save the training log.
- For other arguements, please refer to the github repo [soft-q-learning](https://github.com/haarnoja/softqlearning)

Other related hyper-parameters setting are located in [`sql/examples/mujoco_all_sql.py`](./sql/examples/mujoco_all_sql.py). The default reference distribution is Beta distribution. The reference distribution option supports "beta" (Beta distribution) and "norm" (Gaussian distribution).

| Swimmer (rllab) | Humanoid (rllab) |  Hopper-v1 |  Half-cheetah-v1 |  Ant-v1 |  Walker-v1
|-------------------------|:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:
| ![](/figs/sql/Swimmer-rllab.png) | ![](/figs/sql/Humanoid-rllab.png)  |  ![](/figs/sql/Hopper-v1.png) |  ![](/figs/sql/Half-cheetah-v1.png) |  ![](/figs/sql/Ant-v1.png) |  ![](/figs/sql/Walker-v1.png)
| ![](./figs/sql/gif/swimmer.gif) | ![](./figs/sql/gif/human.gif) | ![](./figs/sql/gif/hopper.gif) | ![](./figs/sql/gif/hc1.gif) | ![](./figs/sql/gif/ant.gif) | ![](./figs/sql/gif/walker1.gif)

_Note: Humanoid has a higher action space dimension, making adversarial learning instable; More future work is needed to make Humanoid run better._

## Constrained Domain Sampling

To show that RAS can draw samples when the support is bounded, we apply it to sample from the distributions with the support [c1,c2]. Please see the code at [`constrained_sampling`](./constrained_sampling). 


| RAS: Beta ref. | RAS: Gaussian ref.  | SVGD | Amortized SVGD  
|-------------------------|:-------------------------:|:-------------------------:|:-------------------------:
| ![](/figs/constrained/cons1_beta.png) | ![](/figs/constrained/cons1_gaussian.png) | ![](/figs/constrained/cons1_svgd_teacher.png) | ![](/figs/constrained/cons1_svgd_student.png)
| ![](/figs/constrained/cons1_beta_2mode.png) | ![](/figs/constrained/cons1_gaussian_2mode.png) | ![](/figs/constrained/cons1_svgd_teacher_2mode.png) | ![](/figs/constrained/cons1_svgd_student_2mode.png)

_Please note that RAS Gaussian ref. recovers [AVB-AC](https://arxiv.org/abs/1701.04722) (Adversarial Variational Bayes with Adaptive Contrast)._

## Entropy Regularization

An entropy term **H(x)** is approximated to stablize adversarial training. As examples, we consider to regularize the following GAN variants: [`GAN`](https://arxiv.org/abs/1406.2661), [`SN-GAN`](https://arxiv.org/abs/1802.05957), [`D2GAN`](https://arxiv.org/abs/1709.03831) and [`Unrolled-GAN`](https://arxiv.org/abs/1611.02163). All entropy-regularization code is at [`entropy`](./entropy): 

To run:

    python run_test.py --model gan_cc
    
It takes the following options (among others) as arguments:

- The `--model` specifies the GAN variant to apply the entropy regularizer. It supports [`gan`,`d2gan`,`ALLgan`,`SNgan`]; default `gan`.  To apply entropy regularizer, change the argument of `--model` as [`gan_cc`,`d2gan_cc`,`ALLgan_cc`,`SNgan_cc`]
  

| Entropy regularizer on 8-GMM toy dataset | SN-GAN  |   SN-GAN + Entropy  
|-------------------------|:-------------------------:|:-------------------------:
| ![](/figs/entropy/Symmetric_KL_Divergence_iclr.png) | ![](/figs/entropy/sn_gan_8gmm.png)  |   ![](/figs/entropy/sn_gan_entropy_8gmm.png)


## Reproduce paper figure results
Jupyter notebooks in [`plots`](./plots) folders are used to reproduce paper figure results.

Note that without modification, we have copyed our extracted results into the notebook, and script will output figures in the paper. If you've run your own training and wish to plot results, you'll have to organize your results in the same format instead.


## Questions?
Please drop us ([Chunyuan](http://chunyuan.li/), [Ke](https://github.com/beckybai), [Jianqiao](https://github.com/jianqiaol) or [Guoyin](https://github.com/guoyinwang)) a line if you have any questions.



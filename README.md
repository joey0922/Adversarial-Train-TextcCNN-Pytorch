# Adversarial-Train-TextCNN-Pytorch
A repository that implements three adversarial training methods, FGSM, "Free" and PGD, which are published in the paper, [*"Fast is better than free: Revisiting adversarial training"*](https://arxiv.org/abs/2001.03994), created by [*Eric Wong*](https://riceric22.github.io/),  [*Leslie Rice*](https://leslierice1.github.io/), and [*Zico Kolter*](http://zicokolter.com/). Then leverage them to train textCNN models, and show how they performance comparing with an original textCNN model trained in an ordinary way.
# Implementation

By introducing perturbations into embeddings of inputs , adversarial training regularizes model's parameters to improve its robustness and generalization. It assumed that the perturbations on inputs wouldn't influence the distribution of outputs. 

In the paper above, it presented three  different adversaries as follows:

## 1. FGSM

[FGSM](https://arxiv.org/abs/1412.6572), short for Fast Gradient Sign Method, is summarized below:

![FGSM](https://github.com/joey0922/Adversarial-Train-TextCNN-Pytorch/tree/main/PNG/FGSM.PNG)

The pseudo code above tells us that for each input x, how FGSM performs adversarial attack:

1. Initialize perturbation <img src="https://latex.codecogs.com/svg.image?\delta" title="https://latex.codecogs.com/svg.image?\delta" /> from uniform distribution between -<img src="https://latex.codecogs.com/svg.image?\epsilon" title="https://latex.codecogs.com/svg.image?\epsilon" /> and <img src="https://latex.codecogs.com/svg.image?\epsilon" title="https://latex.codecogs.com/svg.image?\epsilon" />.

2. Add perturbation to input x then calculate the gradient of <img src="https://latex.codecogs.com/svg.image?\delta" title="https://latex.codecogs.com/svg.image?\delta" />, and update the <img src="https://latex.codecogs.com/svg.image?\delta" title="https://latex.codecogs.com/svg.image?\delta" /> as below:
<p align="center"><img src="https://latex.codecogs.com/svg.image?\delta&space;=&space;\delta&space;&plus;&space;\alpha&space;*&space;sign(\nabla_\delta&space;l(f_\theta(x_i&space;&plus;&space;\delta),&space;y_i))" title="https://latex.codecogs.com/svg.image?\delta = \delta + \alpha * sign(\nabla_\delta l(f_\theta(x_i + \delta), y_i))" /></p>
   

3. If the absolute value of <img src="https://latex.codecogs.com/svg.image?\delta" title="https://latex.codecogs.com/svg.image?\delta" /> is too great, project it back into (-<img src="https://latex.codecogs.com/svg.image?\epsilon" title="https://latex.codecogs.com/svg.image?\epsilon" />, <img src="https://latex.codecogs.com/svg.image?\epsilon" title="https://latex.codecogs.com/svg.image?\epsilon" />):
<p align="center"><img src="https://latex.codecogs.com/svg.image?\delta&space;=&space;max(min(\delta,&space;\epsilon),&space;-\epsilon)" title="https://latex.codecogs.com/svg.image?\delta = max(min(\delta, \epsilon), -\epsilon)" /></p>
   

4. Update  model weights with some optimizer, e.g. SGD:
<p align="center"><img src="https://latex.codecogs.com/svg.image?\theta&space;=&space;\theta&space;-&space;\nabla_\theta&space;l(f_\theta(x_i&space;&plus;&space;\delta),&space;y_i)" title="https://latex.codecogs.com/svg.image?\theta = \theta - \nabla_\theta l(f_\theta(x_i + \delta), y_i)" /></p>

## 2. Free

The pseudo code of "Free" is as followings:

![free](https://github.com/joey0922/Adversarial-Train-TextCNN-Pytorch/tree/main/PNG/free.PNG)

It can be regarded that Free adversarial repeats several FGSM attacks in one batch of x:

1. Initialize <img src="https://latex.codecogs.com/svg.image?\delta" title="https://latex.codecogs.com/svg.image?\delta" /> to 0 before training starts.

2. For each input x, Free adversary perform FGSM adversarial attack N times simultaneously.

3. For every FGSM attack, It compute gradients for perturbation and model weights simultaneously:
<p align="center"><img src="https://latex.codecogs.com/svg.image?\nabla_\delta,&space;\nabla_\theta&space;=&space;\nabla&space;l(f_\theta(x_i&plus;\delta),y_i)" title="https://latex.codecogs.com/svg.image?\nabla_\delta, \nabla_\theta = \nabla l(f_\theta(x_i+\delta),y_i)" /></p>

4. Update <img src="https://latex.codecogs.com/svg.image?\delta" title="https://latex.codecogs.com/svg.image?\delta" />:
<p align="center"><img src="https://latex.codecogs.com/svg.image?\delta&space;=&space;\delta&space;&plus;&space;\epsilon&space;*&space;sign(\nabla_\delta&space;l(f_\theta(x_i&space;&plus;&space;\delta),&space;y_i))" title="https://latex.codecogs.com/svg.image?\delta = \delta + \epsilon * sign(\nabla_\delta l(f_\theta(x_i + \delta), y_i))" /></p>
   

   This formula is similar to FGSM's, the only difference is the coefficient of sign function.

5. The same as FGSM, project <img src="https://latex.codecogs.com/svg.image?\delta" title="https://latex.codecogs.com/svg.image?\delta" /> back into (-<img src="https://latex.codecogs.com/svg.image?\epsilon" title="https://latex.codecogs.com/svg.image?\epsilon" />, <img src="https://latex.codecogs.com/svg.image?\epsilon" title="https://latex.codecogs.com/svg.image?\epsilon" />) if its absolute value too great:
<p align="center"><img src="https://latex.codecogs.com/svg.image?\delta&space;=&space;max(min(\delta,&space;\epsilon),&space;-\epsilon)" title="https://latex.codecogs.com/svg.image?\delta = max(min(\delta, \epsilon), -\epsilon)" /></p>
   

6. Update  model weights with some optimizer, e.g. SGD:
<p align="center"><img src="https://latex.codecogs.com/svg.image?\theta&space;=&space;\theta&space;-&space;\nabla_\theta" title="https://latex.codecogs.com/svg.image?\theta = \theta - \nabla_\theta" /></p>
   

**Note:** Because Free adversary attack N times for each batch, epochs of training could decreased to T/N times.

## 3. PGD

PGD adversarial training updates perturbation <img src="https://latex.codecogs.com/svg.image?\delta" title="https://latex.codecogs.com/svg.image?\delta" /> N times before update model weights:

![PGD](https://github.com/joey0922/Adversarial-Train-TextCNN-Pytorch/tree/main/PNG/PGD.PNG)

The steps are as followings:

1. For each input x, PGD initialize <img src="https://latex.codecogs.com/svg.image?\delta" title="https://latex.codecogs.com/svg.image?\delta" /> to zero at first.

2. Loop N times to update <img src="https://latex.codecogs.com/svg.image?\delta" title="https://latex.codecogs.com/svg.image?\delta" /> in the way below:
<p align="center"><img src="https://latex.codecogs.com/svg.image?\delta&space;=&space;\delta&space;&plus;&space;\alpha&space;*&space;sign(\nabla_\delta&space;l(f_\theta(x_i&space;&plus;&space;\delta),&space;y_i))" title="https://latex.codecogs.com/svg.image?\delta = \delta + \alpha * sign(\nabla_\delta l(f_\theta(x_i + \delta), y_i))" /></p>

3. If it exceeds the scale (-<img src="https://latex.codecogs.com/svg.image?\epsilon" title="https://latex.codecogs.com/svg.image?\epsilon" />, <img src="https://latex.codecogs.com/svg.image?\epsilon" title="https://latex.codecogs.com/svg.image?\epsilon" />), it must be scaled again:
<p align="center"><img src="https://latex.codecogs.com/svg.image?\delta&space;=&space;max(min(\delta,&space;\epsilon),&space;-\epsilon)" title="https://latex.codecogs.com/svg.image?\delta = max(min(\delta, \epsilon), -\epsilon)" /></p>
   

4. After accumulate <img src="https://latex.codecogs.com/svg.image?\delta" title="https://latex.codecogs.com/svg.image?\delta" /> N times, update model weights with some optimizer:
<p align="center"><img src="https://latex.codecogs.com/svg.image?\theta&space;=&space;\theta&space;-&space;\nabla_\theta&space;l(f_\theta(x_i&space;&plus;&space;\delta),&space;y_i)" title="https://latex.codecogs.com/svg.image?\theta = \theta - \nabla_\theta l(f_\theta(x_i + \delta), y_i)" /></p>

To achieve the three adversarial trainings above , this repository includes two different implementations:

1. encapsulate a adversarial training class to add perturbations of inputs when training, such as FGSM, PGD etc.
2. create a textCNN class that is able to add perturbations when a instance calls forward function, such as Free etc.

# Configuration

## Environment
+ python=3.8.8
+ joblib==1.0.1
+ numpy==1.20.1
+ pandas==1.2.4
+ scikit_learn==1.0.2
+ torch==1.11.0
+ tqdm==4.59.0
## Data

The train data is from [this github site](https://github.com/649453932/Chinese-Text-Classification-Pytorch), and it consists of two hundred thousand new headlines from [THUCNews](http://thuctc.thunlp.org/). There are 10 classes, including finance, realty, stocks, education, science, society, politics, sports, game and entertainment, twenty thousand texts with length between 20 and 30 for each. The sheet below indicates how the data set is divided:

|        |  数量  |
| :----: | :----: |
| 训练集 | 180000 |
| 验证集 | 10000  |
| 测试集 | 10000  |
| 类目数 |   10   |

Inputs of model are characters of texts, and the pre-trained character embeddings come from  [sogou news](https://github.com/Embedding/Chinese-Word-Vectors). Click [here](https://pan.baidu.com/s/14k-9jsspp43ZhMxqPmsWMQ) to download them.

# Result

|        | Precision | Recall |  F1  | Accuracy |
| :----: | :-------: | :----: | :--: | :------: |
| normal |   91.54   | 91.54  | 1.83 |  91.54   |
|  FGSM  |   92.05   | 91.98  | 1.84 |  91.97   |
|  Free  |   90.14   | 89.94  | 1.80 |  89.94   |
|  PGD   |   92.10   | 92.03  | 1.84 |  92.03   |

According to the index listed above, it seems that PDG adversarial training has the best performance, but it takes at least twice longer to train models. The performance of model trained by FGSM is close to PSG's, however, it only takes half the time of PGD training.Though it takes twice the time of a model training in the ordinary way.
# Reference

 * [Fast is better than free: Revisiting adversarial training](https://arxiv.org/abs/2001.03994)
 * [fast_adversarial](https://github.com/locuslab/fast_adversarial)
 * [Chinese-Text-Classification-Pytorch](https://github.com/649453932/Chinese-Text-Classification-Pytorch)
 * [知乎专栏：【炼丹技巧】功守道：NLP中的对抗训练 + PyTorch实现](https://zhuanlan.zhihu.com/p/91269728)
 * [知乎专栏：一文搞懂NLP中的对抗训练](https://zhuanlan.zhihu.com/p/103593948)

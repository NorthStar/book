# Linear Regression
Let us begin the tutorial with a classical problem called Linear Regression \[[1](#References)\]. In this chapter, we will train a model from a realistic dataset to predict home prices. Some important concepts in Machine Learning will be covered through this example.

The source code for this tutorial lives on [book/fit_a_line](https://github.com/PaddlePaddle/book/tree/develop/fit_a_line). For instructions on getting started with PaddlePaddle, see [PaddlePaddle installation guide](http://www.paddlepaddle.org/doc_cn/build_and_install/index.html).

## Problem Setup
Suppose we have a dataset of $n$ real estate properties. These real estate properties will be referred to as *homes* in this chapter for clarity.

Each home is associated with $d$ attributes. The attributes describe characteristics such the number of rooms in the home, the number of schools or hospitals in the neighborhood, and the traffic condition nearby.

In our problem setup, the attribute $x_{i,j}$ denotes the $j$th characteristic of the $i$th home. In addition, $y_i$ denotes the price of the $i$th home. Our task is to predict $y_i$ given a set of attributes $\{x_{i,1}, ..., x_{i,d}\}$. We assume that the price of a home is a linear combination of all of its attributes, namely,

$$y_i = \omega_1x_{i,1} + \omega_2x_{i,2} + \ldots + \omega_dx_{i,d} + b,  i=1,\ldots,n$$

where $\vec{\omega}$ and $b$ are the model parameters we want to estimate. Once they are learned, we will be able to predict the price of a home, given the attributes associated with it. We call this model **Linear Regression**. In other words, we want to regress a value against several values linearly. In practice, a linear model is often too simplistic to capture the real relationships between the variables. Yet, because Linear Regression is easy to train and analyze, it has been applied to a large number of real problems. As a result, it is an important topic in many classic Statistical Learning and Machine Learning textbooks \[[2,3,4](#References)\].

## Results Demonstration
We first show the result of our model. The dataset [UCI Housing Data Set](https://archive.ics.uci.edu/ml/datasets/Housing) is used to train a linear model to predict the home prices in Boston. The figure below shows the predictions the model makes for some home prices. The $X$-axis represents the median value of the prices of simlilar homes within a bin, while the $Y$-axis represents the home value our linear model predicts. The dotted line represents points where $X=Y$. When reading the diagram, the more precise the model predicts, the closer the point is to the dotted line.
<p align="center">
	<img src = "image/predictions.png" width=400><br/>
	Figure 1. Predicted Value V.S. Actual Value (波士顿房价预测->Prediction of Boston home prices; 预测价格->Predicted prices; 单位->Units; 实际价格->Actual prices)
</p>

## Model Overview

### Model Definition

In the UCI Housing Data Set, there are 13 home attributes $\{x_{i,j}\}$ that are related to the median home price $y_i$, which we aim to predict. Thus, our model can be written as:

$$\hat{Y} = \omega_1X_{1} + \omega_2X_{2} + \ldots + \omega_{13}X_{13} + b$$

where $\hat{Y}$ is the predicted value used to differentiate from actual value $Y$. The model learns parameters $\omega_1, \ldots, \omega_{13}, b$, where the entries of $\vec{\omega}$ are **weights** and $b$ is **bias**.

Now we need an objective to optimize, so that the learned parameters can make $\hat{Y}$ as close to $Y$ as possible. Let's refer to the concept of [Loss Function (Cost Function)](https://en.wikipedia.org/wiki/Loss_function). A loss function must output a non-negative value, given any pair of the actual value $y_i$ and the predicted value $\hat{y_i}$. This value reflects the magnitutude of the model error.

For Linear Regression, the most common loss function is [Mean Square Error (MSE)](https://en.wikipedia.org/wiki/Mean_squared_error) which has the following form:

$$MSE=\frac{1}{n}\sum_{i=1}^{n}{(\hat{Y_i}-Y_i)}^2$$

That is, for a dataset of size $n$, MSE is the average value of the the prediction sqaure errors.

### Training

After setting up our model, there are several major steps to go through to train it:
1. Initialize the parameters including the weights $\vec{\omega}$ and the bias $b$. For example, we can set their mean values as $0$s, and their standard deviations as $1$s.
2. Feedforward the network output and the loss function.
3. [Backpropagate](https://en.wikipedia.org/wiki/Backpropagation) the errors. The errors will be propagated from the output layer back to the input layer, during which the model parameters will be updated with the corresponding errors.
4. Repeat steps 2~3, until the loss is below a predefined threshold or the maximum number of repeats is reached.

## Data Preparation
Follow the command below to prepare data:
```bash
cd data && python prepare_data.py
```
This line of code will download the dataset from the [UCI Housing Data Set](https://archive.ics.uci.edu/ml/datasets/Housing) and perform some [preprocessing](#Preprocessing). Finally, the dataset is split into a training set and a test set.

The dataset contains 506 lines in total, each line describing the attributes and the median price of a certain type of homes in Boston. The attributes are explained below:

| Attribute Name | Characteristic | Data Type |
| ------| ------ | ------ |
| CRIM | per capita crime rate by town | Continuous|
| ZN | proportion of residential land zoned for lots over 25,000 sq.ft. | Continuous |
| INDUS | proportion of non-retail business acres per town | Continuous |
| CHAS | Charles River dummy variable | Discrete, 1 if tract bounds river; 0 otherwise|
| NOX | nitric oxides concentration (parts per 10 million) | Continuous |
| RM | average number of rooms per dwelling | Continuous |
| AGE | proportion of owner-occupied units built prior to 1940 | Continuous |
| DIS | weighted distances to five Boston employment centres | Continuous |
| RAD | index of accessibility to radial highways | Continuous |
| TAX | full-value property-tax rate per $10,000 | Continuous |
| PTRATIO | pupil-teacher ratio by town | Continuous |
| B | 1000(Bk - 0.63)^2 where Bk is the proportion of blacks by town | Continuous |
| LSTAT | % lower status of the population | Continuous |
| MEDV | Median value of owner-occupied homes in $1000's | Continuous |

The last entry is the median home price.

### Preprocessing
#### Continuous and Discrete Data
We define a feature vector of length 13 for each home, where each entry corresponds to an attribute. Our first observation is that, among the 13 dimensions, there are 12 continuous dimensions and 1 discrete dimension.

Note that although a discrete value is also written as numeric values such as 0, 1, or 2, its meaning differs from a continuous value drastically.  The linear difference between two discrete values has no meaning. For example, suppose $0$, $1$, and $2$ are used to represent colors *Red*, *Green*, and *Blue* respectively. Judging from the numeric representation of these colors, *Red* differs more from *Blue* than it does from *Green*. Yet in actuality, it is not true that extent to which the color *Blue* is different from *Red* is greater than the extent to which *Green* is different from *Red*. Therefore, when handling a discrete feature that has $d$ possible values, we usually convert it to $d$ new features where each feature takes a binary value, $0$ or $1$, indicating whether the original value is absent or present. Alternatively, the discrete features can be mapped onto a continuous multi-dimensional vector through an embedding table. For our problem here, because CHAS itself is a binary discrete value, we do not need to do any preprocessing.

#### Feature Normalization
We also observe a huge difference among the value ranges of the 13 features (Figure 2). For instance, the values of feature *B* fall in $[0.32, 396.90]$, whereas those of feature *NOX* has a range of $[0.3850, 0.8170]$. An effective optimization would require data normalization. The goal of data normalization is to scale te values of each feature into roughly the same range, perhaps $[-0.5, 0.5]$. Here, we adopt a popular normalization technique where we substract the mean value from the feature value and divide the result by the width of the original range.

There are at least three reasons for [Feature Normalization](https://en.wikipedia.org/wiki/Feature_scaling) (Feature Scaling):
- A value range that is too large or too small might cause floating number overflow or underflow during computation.
- Different value ranges might result in varying *importances* of different features to the model (at least in the beginning of the training process). This assumption about the data is often unreasonable, making the optimization difficult, which in turn results in increased training time.
- Many machine learning techniques or models (e.g., *L1/L2 regularization* and *Vector Space Model*) assumes that all the features have roughly zero means and their value ranges are similar.

<p align="center">
	<img src = "image/ranges.png" width=550><br/>
	Figure 2. The value ranges of the features (特征尺度->Feature value range)
</p>

#### Prepare Training and Test Sets
We split the dataset in two, one for adjusting the model parameters, namely, for model training, and the other for model testing. The model error on the former is called the **training error**, and the error on the latter is called the **test error**. Our goal in training a model is to find the statistical dependency between the outputs and the inputs, so that we can predict new outputs given new inputs. As a result, the test error reflects the performance of the model better than the training error does. We consider two things when deciding the ratio of the training set to the test set: 1) More training data will decrease the variance of the parameter estimation, yielding more reliable models; 2) More test data will decrease the variance of the test error, yielding more reliable test errors. One standard split ratio is $8:2$. You can try different split ratios to observe how the two variances change.

Executing the following command to split the dataset and write the training and test set into the `train.list` and `test.list` files, so that later PaddlePaddle can read from them.
```python
python prepare_data.py -r 0.8 #8:2 is the default split ratio
```

When training complex models, we usually have one more split: the validation set. Complex models usually have [Hyperparameters](https://en.wikipedia.org/wiki/Hyperparameter_optimization) that need to be set before the training process begins. These hyperparameters are not part of the model parameters and cannot be trained using the same Loss Function (e.g., the number of layers in the network). Thus we will try several sets of hyperparameters to get several models, and compare these trained models on the validation set to pick the best one, and finally it on the test set. Because our model is relatively simple in this problem, we ignore this validation process for now.

### Provide Data to PaddlePaddle
After the data is prepared, we use a Python Data Provider to provide data for PaddlePaddle. A Data Provider is a Python function which will be called by PaddlePaddle during training. In this example, the Data Provider only needs to read the data and return it to the training process of PaddlePaddle line by line.

```python
from paddle.trainer.PyDataProvider2 import *
import numpy as np
#define data type and dimensionality
@provider(input_types=[dense_vector(13), dense_vector(1)])
def process(settings, input_file):
    data = np.load(input_file.strip())
    for row in data:
	    yield row[:-1].tolist(), row[-1:].tolist()

```

## Model Configuration

### Data Definition
We first call the function `define_py_data_sources2` to let PaddlePaddle read training and test data from the `dataprovider.py` in the above. PaddlePaddle can accept configuration info from the command line, for example, here we pass a variable named `is_predict` to control the model to have different structures during training and test.
```python
from paddle.trainer_config_helpers import *

is_predict = get_config_arg('is_predict', bool, False)

define_py_data_sources2(
    train_list='data/train.list',
    test_list='data/test.list',
    module='dataprovider',
    obj='process')

```

### Algorithm Settings
Next, set the details of the optimization algorithm. Due to the simplicity of the Linear Regression model, we only need to set the `batch_size` which defines how many samples are used every time for updating the parameters.
```python
settings(batch_size=2)
```

### Network
Finally, we use `fc_layer` and `LinearActivation` to represent the Linear Regression model.
```python
#input data of 13 dimensional home information
x = data_layer(name='x', size=13)

y_predict = fc_layer(
    input=x,
    param_attr=ParamAttr(name='w'),
    size=1,
    act=LinearActivation(),
    bias_attr=ParamAttr(name='b'))

if not is_predict: #when training, we use MSE (i.e., regression_cost) as the Loss Function
    y = data_layer(name='y', size=1)
    cost = regression_cost(input=y_predict, label=y)
    outputs(cost) #output MSE to view the loss change
else: #during test, output the prediction value
    outputs(y_predict)
```

## Training Model
The PaddlePaddle command line trainer can be executed in the root directory of the code. Here, we sepcify the configuration file as `trainer_config.py`; in addition, we train $30$ passes and save the result under the directory `output`:
```bash
./train.sh
```

## Usage
Now we can use the trained model to do prediction.
```bash
python predict.py
```
By default, we use the model in `output/pass-00029` for prediction and compare it with the actual home prices. The result is shown in `predictions.png`.

To change the model, test, or data used in the prediction, simply pass in a new model path or data path:
```bash
python predict.py -m output/pass-00020 -t data/housing.test.npy
```

## Summary
This chapter introduces *Linear Regression* and how to train and test this model with PaddlePaddle, using the UCI Housing Data Set. Because a large number of more complex models and techniques are derived from linear regression, it is important to understand its underlying theory and limitation.


## References
1. https://en.wikipedia.org/wiki/Linear_regression
2. Friedman J, Hastie T, Tibshirani R. The elements of statistical learning[M]. Springer, Berlin: Springer series in statistics, 2001.
3. Murphy K P. Machine learning: a probabilistic perspective[M]. MIT press, 2012.
4. Bishop C M. Pattern recognition[J]. Machine Learning, 2006, 128.

<br/>
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" href="http://purl.org/dc/dcmitype/Text" property="dct:title" rel="dct:type">本教程</span> 由 <a xmlns:cc="http://creativecommons.org/ns#" href="http://book.paddlepaddle.org" property="cc:attributionName" rel="cc:attributionURL">PaddlePaddle</a> 创作，采用 <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享 署名-非商业性使用-相同方式共享 4.0 国际 许可协议</a>进行许可。

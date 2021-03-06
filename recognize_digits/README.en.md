# Recognize Digits

The source code for this tutorial is under [book/recognize_digits](https://github.com/PaddlePaddle/book/tree/develop/recognize_digits). First-time readers, please refer to PaddlePaddle [installation instructions](http://www.paddlepaddle.org/doc_cn/build_and_install/index.html).

## Introduction
When we learn a new programming language, the first task is usually to write a program that prints "Hello World." In Machine Learning or Deep Learning, the equivalent task is to train a model to perform handwritten digit recognition with [MNIST](http://yann.lecun.com/exdb/mnist/) dataset. Handwriting recognition is a typical image classification problem. The problem is relatively easy, and MNIST is a complete dataset. As a simple Computer Vision dataset, MNIST contains images of handwritten digits and their corresponding labels (Fig. 1). The input image is a 28x28 matrix, and the label is one of the digits from 0 to 9. Each image is normalized in size and centered.

<p align="center">
<img src="image/mnist_example_image.png" width="400"><br/>
Fig. 1. Examples of MNIST images
</p>

The MNIST dataset is created from the [NIST](https://www.nist.gov/srd/nist-special-database-19) Special Database 3 (SD-3) and the Special Database 1 (SD-1). The SD-3 is labeled by the staff of the U.S. Census Bureau, while SD-1 is labeled by high school students the in U.S. Therefore the SD-3 is cleaner and easier to recognize than the SD-1 dataset. Yann LeCun et al. used half of the samples from each of SD-1 and SD-3 to create the MNIST training set (60,000 samples) and test set (10,000 samples), where training set was labeled by 250 different annotators, and it was guaranteed that there wasn't a complete overlap of annotators of training set and test set.

Yann LeCun, one of the founders of Deep Learning, contributed highly towards handwritten character recognition in early days and proposed CNN (Convolutional Neural Network), which drastically improved recognition capability for handwritten characters. CNNs are now a critical concept in Deep Learning. From Yann LeCun's first proposal of LeNet to those winning models in ImageNet, such as VGGNet, GoogLeNet, ResNet, etc. (Please refer to [Image Classification](https://github.com/PaddlePaddle/book/tree/develop/image_classification) tutorial), CNN achieved a series of impressive results in Image Classification tasks.

Many algorithms are tested on MNIST. In 1998, LeCun experimented with single layer linear classifier, MLP (Multilayer Perceptron) and Multilayer CNN LeNet. These algorithms constantly reduced test error from 12% to 0.7% \[[1](#References)\]. Since then, researchers have worked on many algorithms such as k-NN (K-Nearest Neighbors) \[[2](#References)\], Support Vector Machine (SVM) \[[3](#References)\], Neural Networks \[[4-7](#References)\] and Boosting \[[8](#References)\]. Various preprocessing methods like distortion removal, noise removal, blurring etc. have also been applied to increase recognition accuracy.

In this tutorial, we tackle the task of handwritten character recognition. We start with a simple softmax regression model and guide our readers step-by-step to improve this model's performance on the task of recognition.


## Model Overview

Before introducing classification algorithms and training procedure, we provide some definitions:
- $X$ is the input: Input is a $28\times28$ MNIST image. It is flattened to a $784$ dimensional vector. $X=\left ( x_0, x_1, \dots, x_{783} \right )$.
- $Y$ is the output: Output of the classifier is 1 of the 10 classes (digits from 0 to 9). $Y=\left ( y_0, y_1, \dots, y_9 \right )$. Each dimension $y_i$ represents the probability that the input image belongs to class $i$.
- $L$ is the ground truth label: $L=\left ( l_0, l_1, \dots, l_9 \right )$. It is also 10 dimensional, but only one dimension is 1 and all others are all 0.

### Softmax Regression

In a simple softmax regression model, the input is fed to fully connected layers and a softmax function is applied to get probabilities of multiple output classes\[[9](#References)\].

Input $X$ is multiplied with weights $W$, and bias $b$ is added to generate activations.

$$ y_i = softmax(\sum_j W_{i,j}x_j + b_i) $$

where $ softmax(x_i) = \frac{e^{x_i}}{\sum_j e^{x_j}} $

For an $N$ class classification problem with $N$ output nodes, an $N$ dimensional vector is normalized to $N$ real values in the range [0, 1], each representing the probability of the sample to belong to the class. Here $y_i$ is the prediction probability that an image is digit $i$.

In such a classification problem, we usually use the cross entropy loss function:

$$  crossentropy(label, y) = -\sum_i label_ilog(y_i) $$

Fig. 2 shows a softmax regression network, with weights in black, and bias in red. +1 indicates bias is 1.

<p align="center">
<img src="image/softmax_regression.png" width=400><br/>
Fig. 2. Softmax regression network architecture<br/>
输入层 -> input layer<br/>
权重W -> weights W<br/>
激活前 -> before activation<br/>
激活函数 -> activation function<br/>
输出层 -> output layer<br/>
偏置b -> bias b<br/>
</p>

### Multilayer Perceptron

The Softmax regression model described above uses the simplest two-layer neural network, i.e. it only contains an input layer and an output layer. So its regression ability is limited. To achieve better recognition results, we consider adding several hidden layers \[[10](#References)\] between the input layer and the output layer.

1.  After the first hidden layer, we get $ H_1 = \phi(W_1X + b_1) $, where $\phi$ is the activation function. Some common ones are sigmoid, tanh and ReLU.
2.  After the second hidden layer, we get $ H_2 = \phi(W_2H_1 + b_2) $.
3.  Finally, after output layer, we get $Y=softmax(W_3H_2 + b_3)$, the final classification result vector.

Fig. 3. is Multilayer Perceptron network, with weights in black, and bias in red. +1 indicates bias is 1.

<p align="center">
<img src="image/mlp.png" width=500><br/>
Fig. 3. Multilayer Perceptron network architecture<br/>
输入层X -> input layer X<br/>
隐藏层$H_1$(含激活函数) -> hidden layer $H_1$ (including activation function)<br/>
隐藏层$H_2$(含激活函数) -> hidden layer $H_2$ (including activation function)<br/>
输出层Y -> output layer Y<br/>
</p>

### Convolutional Neural Network

#### Convolutional Layer

<p align="center">
<img src="image/conv_layer.png" width=500><br/>
Fig. 4. Convolutional layer<br/>
输入数据 -> input data<br/>
卷积输出 -> convolution output<br/>
</p>

The Convolutional layer is the core of a Convolutional Neural Network. The parameters in this layer are composed of a set of filters or kernels. In the forward step, each kernel moves horizontally and vertically, we compute a dot product of the kernel and the input at the corresponding positions, to this result we add bias and apply an activation function. The result is a two-dimensional activation map. For example, some kernel may recognize corners, and some may recognize circles. These convolution kernels may respond strongly to the corresponding features.

Fig. 4 is a dynamic graph of a convolutional layer, where depths are not shown for simplicity. Input is $W_1=5, H_1=5, D_1=3$. In fact, this is a common representation for colored images. $W_1$ and  $H_1$ of a colored image correspond to the width and height respectively. $D_1$ corresponds to the 3 color channels for RGB. The parameters of the convolutional layer are $K=2, F=3, S=2, P=1$. $K$ is the number of kernels. Here, $Filter W_0$ and $Filter   W_1$ are two kernels. $F$ is kernel size. $W0$ and $W1$ are both $3\times3$ matrix in all depths. $S$ is the stride. Kernels move leftwards or downwards by 2 units each time. $P$ is padding, an extension of the input. The gray area in the figure shows zero padding with size 1.

#### Pooling Layer

<p align="center">
<img src="image/max_pooling.png" width="400px"><br/>
Fig. 5 Pooling layer<br/>
输入数据 -> input data<br/>
</p>

A Pooling layer performs downsampling. The main functionality of this layer is to reduce computation by reducing the network parameters. It also prevents overfitting to some extent. Usually, a pooling layer is added after a convolutional layer. Pooling layer can be of various types like max pooling, average pooling, etc. Max pooling uses rectangles to segment the input layer into several parts and computes the maximum value in each part as the output (Fig. 5.)

#### LeNet-5 Network 

<p align="center">
<img src="image/cnn.png"><br/>
Fig. 6. LeNet-5 Convolutional Neural Network architecture<br/>
特征图 -> feature map<br/>
卷积层 -> convolutional layer<br/>
降采样层 -> downsampling layer<br/>
全连接层 -> fully connected layer<br/>
输出层(全连接+Softmax激活) -> output layer (fully connected + softmax activation)<br/>
</p>

[LeNet-5](http://yann.lecun.com/exdb/lenet/) is one of the simplest Convolutional Neural Networks. Fig. 6. shows its architecture: A 2-dimensional input image is fed into two sets of convolutional layers and pooling layers, this output is then fed to a fully connected layer and a softmax classifier. The following three properties of convolution enable LeNet-5 to better recognize images than Multilayer fully connected perceptrons:

- 3D properties of neurons: a convolutional layer is organized by width, height and depth. Neurons in each layer are connected to only a small region in the previous layer. This region is called the receptive field.
- Local connection: A CNN utilizes the local space correlation by connecting local neurons. This design guarantees that the learned filter has a strong response to local input features. Stacking many such layers generates a non-linear filter that is more global. This enables the network to first obtain good representation for small parts of input and then combine them to represent a larger region.
- Sharing weights: In a CNN, computation is iterated on shared parameters (weights and bias) to form a feature map. This means all neurons in the same depth of the output respond to the same feature. This allows detecting a feature regardless of its position in the input and enables translation equivariance.

For more details on Convolutional Neural Networks, please refer to [this Stanford open course]( http://cs231n.github.io/convolutional-networks/ ) and [this Image Classification](https://github.com/PaddlePaddle/book/blob/develop/image_classification/README.md) tutorial.

### List of Common Activation Functions  
- Sigmoid activation function: $ f(x) = sigmoid(x) = \frac{1}{1+e^{-x}} $

- Tanh activation function: $ f(x) = tanh(x) = \frac{e^x-e^{-x}}{e^x+e^{-x}} $

  In fact, tanh function is just a rescaled version of the sigmoid function. It is obtained by magnifying the value of the sigmoid function and moving it downwards by 1.

- ReLU activation function: $ f(x) = max(0, x) $

For more information, please refer to [Activation functions on Wikipedia](https://en.wikipedia.org/wiki/Activation_function).

## Data Preparation

### Data Download

Execute the following command to download the [MNIST](http://yann.lecun.com/exdb/mnist/) dataset and unzip. Add paths to the training set and the test set to train.list and test.list respectively for PaddlePaddle to read.

```bash
./data/get_mnist_data.sh
```

`gzip` downloaded data. The following files can be found in `data/raw_data`:

|    File name          |       Description              |
|----------------------|-------------------------|
|train-images-idx3-ubyte|  Training images, 60,000 |
|train-labels-idx1-ubyte|  Training labels, 60,000 |
|t10k-images-idx3-ubyte |  Evaluation images, 10,000 |
|t10k-labels-idx1-ubyte |  Evaluation labels, 10,000 |

Users can randomly generate 10 images with the following script (Refer to Fig. 1.)

```bash
./load_data.py
```

### Provide Data to PaddlePaddle

We use python interface to provide data to system. `mnist_provider.py` shows a complete example for training on MNIST data.

```python
# Define a py data provider
@provider(
    input_types={'pixel': dense_vector(28 * 28),
                 'label': integer_value(10)})
def process(settings, filename):  # settings is not used currently.
		# Open image file
    with open( filename + "-images-idx3-ubyte", "rb") as f:             
		# Read first 4 parameters. magic is data format. n is number of data. rows and cols are number of rows and columns, respectively
        magic, n, rows, cols = struct.upack(">IIII", f.read(16))        
		# With empty string as a unit, read data one by one
        images = np.fromfile(                                           
            f, 'ubyte',
            count=n * rows * cols).reshape(n, rows, cols).astype('float32')
		# Normalize data of [0, 255] to [-1,1]
        images = images / 255.0 * 2.0 - 1.0                             


		# Open label file
    with open( filename + "-labels-idx1-ubyte", "rb") as l:             
		# Read first two parameters
        magic, n = struct.upack(">II", l.read(8))                       
		# With empty string as a unit, read data one by one
        labels = np.fromfile(l, 'ubyte', count=n).astype("int")         

    for i in xrange(n):
        yield {"pixel": images[i, :], 'label': labels[i]}
```


## Model Configurations

### Data Definition

In the model configuration, use `define_py_data_sources2` to define reading of data from `dataprovider`. If this configuration is used for prediction, data definition is not necessary.

```python
 if not is_predict:
     data_dir = './data/'
     define_py_data_sources2(
         train_list=data_dir + 'train.list',
         test_list=data_dir + 'test.list',
         module='mnist_provider',
         obj='process')
```

### Algorithm Configuration

Set training related parameters.

- batch_size: use 128 samples in each training step.
- learning_rate: determines step taken in each iteration, it determines how fast the model converges.
- learning_method: use optimizer `MomentumOptimizer` for training. The parameter 0.9 indicates momentum keeps 0.9 of previous speed.
- regularization: A method to prevent overfitting. Here L2 regularization is used.

```python
settings(
    batch_size=128,
    learning_rate=0.1 / 128.0,
    learning_method=MomentumOptimizer(0.9),
    regularization=L2Regularization(0.0005 * 128))
```

### Model Architecture

#### Overview

First get reference labels from `data_layer`, and get classification results (predictions) from classifier. Here we provide three different classifiers. In training, we compute loss function, which is usually cross entropy for classification problem. In prediction, we can directly output the results (predictions).

``` python
data_size = 1 * 28 * 28
label_size = 10
img = data_layer(name='pixel', size=data_size)

predict = softmax_regression(img) # Softmax Regression
#predict = multilayer_perceptron(img) # Multilayer Perceptron
#predict = convolutional_neural_network(img) #LeNet5 Convolutional Neural Network
 
if not is_predict:
    lbl = data_layer(name="label", size=label_size)
    inputs(img, lbl)
    outputs(classification_cost(input=predict, label=lbl))
else:
    outputs(predict)
```

#### Softmax Regression

One simple fully connected layer with softmax activation function outputs classification result.

```python
def softmax_regression(img):
    predict = fc_layer(input=img, size=10, act=SoftmaxActivation())
    return predict
```

#### MultiLayer Perceptron

The following code implements a Multilayer Perceptron with two fully connected hidden layers and a ReLU activation function. The output layer has a Softmax activation function.

```python
def multilayer_perceptron(img):
    # First fully connected layer with ReLU
    hidden1 = fc_layer(input=img, size=128, act=ReluActivation())
    # Second fully connected layer with ReLU
    hidden2 = fc_layer(input=hidden1, size=64, act=ReluActivation())
    # Output layer as fully connected layer and softmax activation. The size must be 10.
    predict = fc_layer(input=hidden2, size=10, act=SoftmaxActivation())
    return predict
```

#### Convolutional Neural Network LeNet-5

The following is the LeNet-5 network architecture. A 2D input image is first fed into two sets of convolutional layers and pooling layers, this result is then fed to a fully connected layer, and another fully connected layer with a softmax activation.

```python
def convolutional_neural_network(img):
    # First convolutional layer - pooling layer
    conv_pool_1 = simple_img_conv_pool(
        input=img,
        filter_size=5,
        num_filters=20,
        num_channel=1,
        pool_size=2,
        pool_stride=2,
        act=TanhActivation())
    # Second convolutional layer - pooling layer
    conv_pool_2 = simple_img_conv_pool(
        input=conv_pool_1,
        filter_size=5,
        num_filters=50,
        num_channel=20,
        pool_size=2,
        pool_stride=2,
        act=TanhActivation())
    # Fully connected layer
    fc1 = fc_layer(input=conv_pool_2, size=128, act=TanhActivation())
    # Output layer as fully connected layer and softmax activation. The size must be 10.
    predict = fc_layer(input=fc1, size=10, act=SoftmaxActivation())
    return predict
```

## Training Model

### Training Commands and Logs

1.Configure `train.sh` to execute training:

```bash
config=mnist_model.py                   # Select network in mnist_model.py
output=./softmax_mnist_model            
log=softmax_train.log                   

paddle train \
--config=$config \                      # Scripts for network configuration.
--dot_period=10 \                       # After `dot_period` steps, print one `.`
--log_period=100 \						# Print a log every batchs
--test_all_data_in_one_period=1 \		# Whether to use all data in every test
--use_gpu=0 \							# Whether to use GPU
--trainer_count=1 \						# Number of CPU or GPU
--num_passes=100 \						# Passes for training (One pass uses all data.)
--save_dir=$output \					# Path to saved model
2>&1 | tee $log

python -m paddle.utils.plotcurve -i $log > plot.png
```

After configuring parameters, execute `./train.sh`. Training log is as follows.

```
I0117 12:52:29.628617  4538 TrainerInternal.cpp:165]  Batch=100 samples=12800 AvgCost=2.63996 CurrentCost=2.63996 Eval: classification_error_evaluator=0.241172  CurrentEval: classification_error_evaluator=0.241172 
.........
I0117 12:52:29.768741  4538 TrainerInternal.cpp:165]  Batch=200 samples=25600 AvgCost=1.74027 CurrentCost=0.840582 Eval: classification_error_evaluator=0.185234  CurrentEval: classification_error_evaluator=0.129297 
.........
I0117 12:52:29.916970  4538 TrainerInternal.cpp:165]  Batch=300 samples=38400 AvgCost=1.42119 CurrentCost=0.783026 Eval: classification_error_evaluator=0.167786  CurrentEval: classification_error_evaluator=0.132891 
.........
I0117 12:52:30.061213  4538 TrainerInternal.cpp:165]  Batch=400 samples=51200 AvgCost=1.23965 CurrentCost=0.695054 Eval: classification_error_evaluator=0.160039  CurrentEval: classification_error_evaluator=0.136797 
......I0117 12:52:30.223270  4538 TrainerInternal.cpp:181]  Pass=0 Batch=469 samples=60000 AvgCost=1.1628 Eval: classification_error_evaluator=0.156233 
I0117 12:52:30.366894  4538 Tester.cpp:109]  Test samples=10000 cost=0.50777 Eval: classification_error_evaluator=0.0978 
```

2.Use `plot_cost.py` to plot error curve during training.

```bash
python plot_cost.py softmax_train.log            
```

3.Use `evaluate.py ` to select the best trained model.

```bash
python evaluate.py softmax_train.log
```

### Training Results for Softmax Regression

<p align="center">
<img src="image/softmax_train_log.png" width="400px"><br/>
Fig. 7 Softmax regression error curve<br/>
训练集 -> training set<br/>
测试集 -> test set<br/>
平均代价 -> average cost<br/>
训练轮数 -> epoch<br/>
</p>

Evaluation results of the models:

```text
Best pass is 00013, testing Avgcost is 0.484447
The classification accuracy is 90.01%
```

From the evaluation results, the best pass for softmax regression model is pass-00013, where the classification accuracy is 90.01%, and the last pass-00099 has an accuracy of 89.3%. From Fig. 7, we also see that the best accuracy may not appear in the last pass. This is because during training, the model may already arrive at a local optimum, and it just swings around nearby in the following passes, or it gets a lower local optimum.

### Results of Multilayer Perceptron

<p align="center">
<img src="image/mlp_train_log.png" width="400px"><br/>
Fig. 8. Multilayer Perceptron error curve<br/>
训练集 -> training set<br/>
测试集 -> test set<br/>
平均代价 -> average cost<br/>
训练轮数 -> epoch<br/>
</p>

Evaluation results of the models：

```text
Best pass is 00085, testing Avgcost is 0.164746
The classification accuracy is 94.95%
```

From the evaluation results, the final training accuracy is 94.95%. It is significantly better than the softmax regression model. This is because the softmax regression is simple, and it cannot fit complex data. The Multilayer Perceptron with hidden layers has better capacity to fit complex data than the softmax regression.

### Training results for Convolutional Neural Network

<p align="center">
<img src="image/cnn_train_log.png" width="400px"><br/>
Fig. 9. Convolutional Neural Network error curve<br/>
训练集 -> training set<br/>
测试集 -> test set<br/>
平均代价 -> average cost<br/>
训练轮数 -> epoch<br/>
</p>

Results of model evaluation：

```text
Best pass is 00076, testing Avgcost is 0.0244684
The classification accuracy is 99.20%
```

From the evaluation result, the best accuracy of Convolutional Neural Network is 99.20%. So for image classification, a Convolutional Neural Network has better recognition results than a fully connected network. This is related to the local connection and parameter sharing of convolutional layers. In Fig. 9, the Convolutional Neural Network achieves good results in early steps, which indicates that it converges faster.

## Application Model

### Prediction Commands and Results
Script `predict.py` can make prediction for trained models. For example, in softmax regression:

```bash
python predict.py -c mnist_model.py -d data/raw_data/ -m softmax_mnist_model/pass-00047
```

- -c sets model architecture
- -d sets data for prediction
- -m sets model parameters, here the best trained model is used for prediction

Follow the instructions to input image ID for prediction. The classifier can output probabilities for each digit, predictions with the highest probability, and ground truth label.

```
Input image_id [0~9999]: 3
Predicted probability of each digit:
[[  1.00000000e+00   1.60381094e-28   1.60381094e-28   1.60381094e-28
    1.60381094e-28   1.60381094e-28   1.60381094e-28   1.60381094e-28
    1.60381094e-28   1.60381094e-28]]
Predict Number: 0 
Actual Number: 0
```

From the result, this classifier recognizes the digit on the third image as digit 0 with near to 100% probability. This predicted result is consistent with the ground truth label.

## Conclusion
This tutorial describes a few basic Deep Learning models viz. Softmax regression, Multilayer Perceptron Network and Convolutional Neural Network. The subsequent tutorials will derive more sophisticated models from these. So it is crucial to understand these models for future learning. When our model evolved from a simple softmax regression to slightly complex Convolutional Neural Network, the recognition accuracy on the MNIST data set achieved large improvement in accuracy. This is due to the Convolutional layers' local connections and parameter sharing. While learning new models in the future, we encourage the readers to understand the key ideas that lead a new model to improve results of an old one. Moreover, this tutorial introduced the basic flow of PaddlePaddle model design, starting with a dataprovider, model layer construction, to final training and prediction. Readers can leverage the flow used in this MNIST handwritten digit classification example and experiment with different data and network architectures to train models for classification tasks of their choice.

## References

1. LeCun, Yann, Léon Bottou, Yoshua Bengio, and Patrick Haffner. ["Gradient-based learning applied to document recognition."](http://ieeexplore.ieee.org/abstract/document/726791/) Proceedings of the IEEE 86, no. 11 (1998): 2278-2324.
2. Wejéus, Samuel. ["A Neural Network Approach to Arbitrary SymbolRecognition on Modern Smartphones."](http://www.diva-portal.org/smash/record.jsf?pid=diva2%3A753279&dswid=-434) (2014).
3. Decoste, Dennis, and Bernhard Schölkopf. ["Training invariant support vector machines."](http://link.springer.com/article/10.1023/A:1012454411458) Machine learning 46, no. 1-3 (2002): 161-190.
4. Simard, Patrice Y., David Steinkraus, and John C. Platt. ["Best Practices for Convolutional Neural Networks Applied to Visual Document Analysis."](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.160.8494&rep=rep1&type=pdf) In ICDAR, vol. 3, pp. 958-962. 2003.
5. Salakhutdinov, Ruslan, and Geoffrey E. Hinton. ["Learning a Nonlinear Embedding by Preserving Class Neighbourhood Structure."](http://www.jmlr.org/proceedings/papers/v2/salakhutdinov07a/salakhutdinov07a.pdf) In AISTATS, vol. 11. 2007.
6. Cireşan, Dan Claudiu, Ueli Meier, Luca Maria Gambardella, and Jürgen Schmidhuber. ["Deep, big, simple neural nets for handwritten digit recognition."](http://www.mitpressjournals.org/doi/abs/10.1162/NECO_a_00052) Neural computation 22, no. 12 (2010): 3207-3220.
7. Deng, Li, Michael L. Seltzer, Dong Yu, Alex Acero, Abdel-rahman Mohamed, and Geoffrey E. Hinton. ["Binary coding of speech spectrograms using a deep auto-encoder."](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.185.1908&rep=rep1&type=pdf) In Interspeech, pp. 1692-1695. 2010.
8. Kégl, Balázs, and Róbert Busa-Fekete. ["Boosting products of base classifiers."](http://dl.acm.org/citation.cfm?id=1553439) In Proceedings of the 26th Annual International Conference on Machine Learning, pp. 497-504. ACM, 2009.
9. Rosenblatt, Frank. ["The perceptron: A probabilistic model for information storage and organization in the brain."](http://psycnet.apa.org/journals/rev/65/6/386/) Psychological review 65, no. 6 (1958): 386.
10. Bishop, Christopher M. ["Pattern recognition."](http://s3.amazonaws.com/academia.edu.documents/30428242/bg0137.pdf?AWSAccessKeyId=AKIAJ56TQJRTWSMTNPEA&Expires=1484816640&Signature=85Ad6%2Fca8T82pmHzxaSXermovIA%3D&response-content-disposition=inline%3B%20filename%3DPattern_recognition_and_machine_learning.pdf) Machine Learning 128 (2006): 1-58.

<br/>
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" href="http://purl.org/dc/dcmitype/Text" property="dct:title" rel="dct:type">This book</span> is created by <a xmlns:cc="http://creativecommons.org/ns#" href="http://book.paddlepaddle.org" property="cc:attributionName" rel="cc:attributionURL">PaddlePaddle</a>, and uses <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Shared knowledge signature - non commercial use-Sharing 4.0 International Licensing Protocal</a>.

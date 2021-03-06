# Keras Model Compression 

An Implementation of "Distilling the Knowledge in a Neural Network - Geoffery Hinton et. al" https://arxiv.org/abs/1503.02531


#### Usage 

python runme.py 


##### Initial Model Parameters:
- Total params: 1,199,882
- Trainable params: 1,199,882

### Model Evaluation before compression

- Initial Accuracy on test set~= 0.99
- Initial Model Parameters : 1199882
- Memory footprint per Image Feed Forward ~= ', 4.577 Mb 

##### Evaluation of the Model after compression

- Compressed Model parameters:  74188
- Compression Rate :  16.2x
- Accuracy : 0.96

## Implementation of Logit Regression in Keras 

Experiments and thoughts to investigate the affect of number of parameters in Model.

### Convolutional Network Model Architecture and Summary

____________________________________________________________________________________________________
| Layer (type)                      | Output Shape          | Param #     |Connected to                     
|----------------------------------:|----------------------:|------------:|------------------------:|
| convolution2d_1 (Convolution2D) | (None, 26, 26, 32) |   320      |   convolution2d_input_1[0][0] |     
| convolution2d_2 (Convolution2D) | (None, 24, 24, 64) |   18496    |   convolution2d_1[0][0]       |     
|maxpooling2d_1 (MaxPooling2D)    | (None, 12, 12, 64) |   0        |   convolution2d_2[0][0]       |     
|dropout_1 (Dropout)              | (None, 12, 12, 64) |   0        |   maxpooling2d_1[0][0]        |     
|flatten_1 (Flatten)              | (None, 9216)       |   0        |   dropout_1[0][0]             |     
|dense_1 (Dense)                  | (None, 128)        |   1179776  |   flatten_1[0][0]             |     
|dropout_2 (Dropout)              | (None, 128)        |   0        |   dense_1[0][0]               |     
|dense_2 (Dense)                  | (None, 10)         |   1290     |   dropout_2[0][0]             |     
|activation_1 (Activation)        | (None, 10)         |   0        |   dense_2[0][0]               |     
____________________________________________________________________________________________________

- Total params: 1,199,882
- Trainable params: 1,199,882
- Non-trainable params: 0


### Reducing The Model
If we observe the model weights and number of parameters, the most consuming layers are the 2 Dense Layers. and the two convolutuonal layers have only 18,816 parameters. Accounting for only (18816 / 1,199,882) x 100 =  1.56 percent of all the parameters,

So the first Step is to either to replace it with a lighter model. That generalizes well to new examples, and learns to model the heavy dense layer of the  original model. 

Replacing these 2 heavy dense layers with 1 Hidden layer with 6 HiddenLayer Neurons. We can achieve an accuracy of 0.9626. 

The Logits from the last layer. before the Activation layer were used, as mentioned in Geoffery Hinton's paper to avoid encoutering very small activation values after squashing through Softmax function.

The python Notebook in this repository shows other architectures with varying number of hiddenLayer Neurons, and the affect of HiddenNeurons on the accuracy. See plots/ subdirectory for findings 

### Evaluation of the Model after Reduction

- Compressed Model parameters:  74188
- Compression Rate :  16.2x

### Exepriments on how we can trade accuracy with model size. initial model accuracy is 99%. The minimum model size that we can achieve without dropping below 0.95 accuracy. 

Experiment: 
THe number of hidden Layer Neurons in small model was iteratively decreased, this showed a exponential decrease in model size and linear decrease in accuracy. 

By examining the plot of number of parameters(or model size) to accuracy. we can find the tradeoff, in the number of Hidden Layer Neurons needed to achieve atleast 0.95 accuracy. 

I was able to obtain 16.2x times compression, while keeping accuracy at 0.96

![Compression-Rate-Accuracy](/plots/CompressionRate_Accuracy.png){:class="img-responsive"}
![ParameterSize-Accuracy](/plots/parameterSize_Accuracy.png){:class="img-responsive"}


### Experiment Settings and Hyper Parameters for student model. 
Various Dropouts were tested . 0.2 was chosen to regularize a bit, and showed good generalization accuracy. 

lossFunction : Mean Square Error as mentioned by Geoff Hinton in the paper.. Optimized Adadelta optimizer over

Let s(f) be the minimum model size that can achieve accuracy f. Given 0 <= f_1 < f_2 < ... < f_n  < =1,  0 < s(f_1) < s(f_2) < ... < s(f_n) 

No, The above equation representing the relation of model size and minimum accuray does not hold. During my experiments I have found that accuracy has rised/equal even when the model size was reduced several x times .

However, after plotting the accuracy against model size. it is observed that the accuracy decreases as model size is reduced.

### Future Work and Improvements :
Its also worth noting that in MNIST dataset the characters only appear in the center of the image, and convolution weights corresponding to the edges are blank/constant and likely to be highly redundant or noisy. Pruning these weight connections through the network is likely to effectively reduce model size.

Ideas from Song Hans work in deep compression can be taken forward to establish a general framework for compression of deep-learning models in keras. 

See Hans work here: 
https://arxiv.org/abs/1510.00149 "Deep Compression: Compressing Deep Neural Networks with Pruning, Trained Quantization and Huffman Coding"

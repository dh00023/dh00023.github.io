---
title: "07. Application & Tips"
tag: tensorflow
---

Learning rate,data preprocessing, **Overfitting**(가장 큰 문제 중 하나)

## 이론
### Learning rate
#### - overshooting

rate 값으로 굉장히 큰 값을 정한다면 문제점이 있다.
- step이 크게되면 바깥으로 튕겨 나갈 수 있다.
- 학습이 이루어지지 않을 뿐만 아니라 cost함수에 숫자가 아닌 값들이 나올 수 있다.

#### - Small learning rate

rate값을 굉장히 작게 주어져도 문제점이 있다.
- 시간이 너무 오래걸린다.
- 최소점이 아닌데 멈춰버릴 수 있다.

##### <span style="color: #980000">데이터와 환경에 따라 다르기 때문에 learning rate를 설정할 때는 보통 0.01로 시작해 값을 조절해가는 것이 좋다.</span>

### Preprocessing
상황에 따라 X값을 선처리해야하는 경우가 있다.
데이터 값에 큰차이가 있으면 한쪽으로 왜곡된 형태의 값이 나타난다. 이때는 **정규화(normalize)**를 해줘야한다.
![](http://cfile10.uf.tistory.com/image/191D49114BE007B0331656)

### Overfitting
머신러닝에서 발생하는 큰 문제 중 하나이다.
오버피팅은 학습데이터에 너무 딱 맞도록 모델이 구성된 것이다. 학습데이터는 잘 맞게 나오겠지만 실제적인 테스트데이터에서는 잘 맞지않는 결과가 나오는 경우가 있다.

#### 해결책
- trining data를 많이 가지고 있는다.
- features의 수를 줄인다.
- **Regularization** 일반화시킨다.

#### Regularization(일반화)

너무 큰 weight을 사용하면 안된다. weight이 클 경우에는 구불한 형태가된다.
![](http://www.37steps.com/wp-content/uploads/2014/03/MLSS-2012-Fukumizu-Kernel-Methods-for-Statistical-Learning_0501.png)
![](https://upload.wikimedia.org/wikipedia/commons/thumb/0/02/Regularization.svg/1200px-Regularization.svg.png)

### Learning and test data sets
머신러닝 모델이 얼마나 잘 동작하는지 알아볼 것이다.

![](http://2.bp.blogspot.com/-VtSKa4I6ADg/V7GGoJ7W2gI/AAAAAAAAH_Q/QkKR4xHJ7gA0la6nQ_DWFj7wCzgZO0jLwCK4B/s1600/ScreenShot_20160812235353.png)

30%정도 데이터를 구분해 앞의 70%는 training set, 30%는 test set으로 구분한다. training set만을 가지고 훈련을 시킨다. 그리고 난 후에 test set으로 돌려본다.

### Online learning
training set이 백만개가 있다. 근데 이걸 한번에 넣어서 학습시키려면 시간도 오래걸리므로 10만개씩 나눠서 훈련시키는 방법이다. 처음학습된 결과가 남아 있어야한다. 즉, 있는 데이터에 추가로 학습시켜주는 방법이다.

## 실습
#### Training and Test sets
```python
# datasets을 trainin과 test로 나누는 것이 중요한 점이다.
# Training set은 이 모델에 학습시키는데에만 사용한다.
# Test set으로 이 훈련된 모델을 평가한다.

import tensorflow as tf
tf.set_random_seed(777)  # for reproducibility

x_data = [[1, 2, 1], 
          [1, 3, 2],
          [1, 3, 4],
          [1, 5, 5],
          [1, 7, 5],
          [1, 2, 5],
          [1, 6, 6],
          [1, 7, 7]]
y_data = [[0, 0, 1],
          [0, 0, 1],
          [0, 0, 1],
          [0, 1, 0],
          [0, 1, 0],
          [0, 1, 0],
          [1, 0, 0],
          [1, 0, 0]]

# Evaluation our model using this test dataset
x_test = [[2, 1, 1],
          [3, 1, 2],
          [3, 3, 4]]
y_test = [[0, 0, 1],
          [0, 0, 1],
          [0, 0, 1]]

# placeholder의 장점이 여기에서 나타난다. training data와 학습 데이터를 각각 넣을 수 있다.
X = tf.placeholder("float", [None, 3])
Y = tf.placeholder("float", [None, 3])

W = tf.Variable(tf.random_normal([3, 3]))
b = tf.Variable(tf.random_normal([3]))

# tf.nn.softmax computes softmax activations
# softmax = exp(logits) / reduce_sum(exp(logits), dim)
hypothesis = tf.nn.softmax(tf.matmul(X, W) + b)

# Cross entropy cost/loss
cost = tf.reduce_mean(-tf.reduce_sum(Y * tf.log(hypothesis), axis=1))
# Try to change learning_rate to small numbers
optimizer = tf.train.GradientDescentOptimizer(
    learning_rate=0.1).minimize(cost)

# Correct prediction Test model
prediction = tf.arg_max(hypothesis, 1)
is_correct = tf.equal(prediction, tf.arg_max(Y, 1))
accuracy = tf.reduce_mean(tf.cast(is_correct, tf.float32))

# Launch graph
with tf.Session() as sess:
    # Initialize TensorFlow variables
    sess.run(tf.global_variables_initializer())

    for step in range(201):
        cost_val, W_val, _ = sess.run(
            [cost, W, optimizer], feed_dict={X: x_data, Y: y_data})
        print(step, cost_val, W_val)

    # predict
    print("Prediction:", sess.run(prediction, feed_dict={X: x_test}))
    # Calculate the accuracy
    print("Accuracy: ", sess.run(accuracy, feed_dict={X: x_test, Y: y_test}))
# learning rate이 좋지않으면 2가지 문제가 있다.
# 1. 너무 크면 학습이되지않고 발산이되는 경우가 생긴다.(수렴이 되지않음)
# 2. 너무 작으면 학습이 안될 수도 있고, 조금의 굴곡만 있어도 local minimum에 빠질 수 있다.
```

```
0 5.73203 [[ 0.72881663  0.71536207 -1.18015325]
 [-0.57753736 -0.12988332  1.60729778]
 [ 0.48373488 -0.51433605 -2.02127004]]
1 3.318 [[ 0.66219079  0.74796319 -1.14612854]
 [-0.81948912  0.03000021  1.68936598]
 [ 0.23214608 -0.33772916 -1.94628811]]
2 2.0218 [[ 0.64342022  0.74127686 -1.12067163]
 [-0.81161296 -0.00900121  1.72049117]
 [ 0.2086665  -0.35079569 -1.909742  ]]
3 1.97109 [[ 0.62353212  0.74008238 -1.09958899]
 [-0.80967712 -0.01636279  1.72591698]
 [ 0.17870612 -0.33665746 -1.89391983]]
4 1.94468 [[ 0.60733104  0.73664987 -1.07995546]
 [-0.7900784  -0.03363196  1.72358739]
 [ 0.16691469 -0.3340635  -1.88472235]]
5 1.92357 [[ 0.59031737  0.73510242 -1.06139433]
 [-0.77496767 -0.0404885   1.71533322]
 [ 0.150766   -0.32209399 -1.88054323]]

...

198 0.673627 [[-1.14866793  0.28236824  1.13032556]
 [ 0.3735742   0.18841396  0.33788884]
 [-0.35681725 -0.43911386 -1.25593936]]
199 0.672261 [[-1.15377033  0.28146935  1.13632679]
 [ 0.37484586  0.18958236  0.33544877]
 [-0.35609841 -0.43973011 -1.25604188]]
200 0.670909 [[-1.15885413  0.28058422  1.14229572]
 [ 0.37609792  0.19073224  0.33304682]
 [-0.35536593 -0.44033223 -1.2561723 ]]
Prediction: [2 2 2]
Accuracy:  1.0
```

#### Normalize
```python
import tensorflow as tf
import numpy as np
tf.set_random_seed(777)  # for reproducibility

# normalize해주는 함수이다.
def MinMaxScaler(data):
    numerator = data - np.min(data, 0)
    denominator = np.max(data, 0) - np.min(data, 0)
    # noise term prevents the zero division
    return numerator / (denominator + 1e-7)

# Non-normalized
xy = np.array([[828.659973, 833.450012, 908100, 828.349976, 831.659973],
               [823.02002, 828.070007, 1828100, 821.655029, 828.070007],
               [819.929993, 824.400024, 1438100, 818.97998, 824.159973],
               [816, 820.958984, 1008100, 815.48999, 819.23999],
               [819.359985, 823, 1188100, 818.469971, 818.97998],
               [819, 823, 1198100, 816, 820.450012],
               [811.700012, 815.25, 1098100, 809.780029, 813.669983],
               [809.51001, 816.659973, 1398100, 804.539978, 809.559998]])

# very important. It does not work without it.
xy = MinMaxScaler(xy)
print(xy)

x_data = xy[:, 0:-1]
y_data = xy[:, [-1]]

# placeholders for a tensor that will be always fed.
X = tf.placeholder(tf.float32, shape=[None, 4])
Y = tf.placeholder(tf.float32, shape=[None, 1])

W = tf.Variable(tf.random_normal([4, 1]), name='weight')
b = tf.Variable(tf.random_normal([1]), name='bias')

# Hypothesis
hypothesis = tf.matmul(X, W) + b

# Simplified cost/loss function
cost = tf.reduce_mean(tf.square(hypothesis - Y))

# Minimize
# learning_rate이 적음에도 불구하고 잘돌아간다.
optimizer = tf.train.GradientDescentOptimizer(learning_rate=1e-5)
train = optimizer.minimize(cost)

# Launch the graph in a session.
sess = tf.Session()
# Initializes global variables in the graph.
sess.run(tf.global_variables_initializer())

for step in range(101):
    cost_val, hy_val, _ = sess.run(
        [cost, hypothesis, train], feed_dict={X: x_data, Y: y_data})
    print(step, "Cost: ", cost_val, "\nPrediction:\n", hy_val)
```
```
0 Cost:  0.152309
Prediction:
 [[ 1.63461912]
 [ 0.06613699]
 [ 0.35008186]
 [ 0.67072523]
 [ 0.6113075 ]
 [ 0.61464405]
 [ 0.23171967]
 [-0.13728359]]
1 Cost:  0.152309
Prediction:
 [[ 1.63461804]
 [ 0.06613842]
 [ 0.35008252]
 [ 0.67072499]
 [ 0.61130762]
 [ 0.61464423]
 [ 0.23171999]
 [-0.13728246]]
2 Cost:  0.152308
Prediction:
 [[ 1.63461685]
 [ 0.06613985]
 [ 0.35008317]
 [ 0.67072475]
 [ 0.61130774]
 [ 0.61464441]
 [ 0.23172033]
 [-0.13728121]]
3 Cost:  0.152308
Prediction:
 [[ 1.63461566]
 [ 0.06614128]
 [ 0.35008383]
 [ 0.67072463]
 [ 0.6113078 ]
 [ 0.61464453]
 [ 0.23172063]
 [-0.13728008]]

...

99 Cost:  0.152255
Prediction:
 [[ 1.63450408]
 [ 0.06627944]
 [ 0.35014686]
 [ 0.67070597]
 [ 0.61131608]
 [ 0.61466044]
 [ 0.23175152]
 [-0.13716647]]
100 Cost:  0.152254
Prediction:
 [[ 1.63450289]
 [ 0.06628087]
 [ 0.35014752]
 [ 0.67070574]
 [ 0.61131608]
 [ 0.61466062]
 [ 0.23175186]
 [-0.13716528]]
```

#### MNIST Dataset
```python
# 손으로 쓴 글씨을 자동으로 읽기위한 MNIST Dataset
# Data set 28 x 28 x I image
import tensorflow as tf
import random
# import matplotlib.pyplot as plt
tf.set_random_seed(777)  # for reproducibility

# 간단하게 불러와서 사용할 수 있음.
from tensorflow.examples.tutorials.mnist import input_data
# Check out https://www.tensorflow.org/get_started/mnist/beginners for
# more information about the mnist dataset

# one_hot = True이면 Y값을 one_hot으로 처리한다.
mnist = input_data.read_data_sets("MNIST_data/", one_hot=True)

nb_classes = 10

# MNIST data image of shape 28 * 28 = 784
X = tf.placeholder(tf.float32, [None, 784])
# 0 - 9 digits recognition = 10 classes
Y = tf.placeholder(tf.float32, [None, nb_classes])

W = tf.Variable(tf.random_normal([784, nb_classes]))
b = tf.Variable(tf.random_normal([nb_classes]))

# Hypothesis (using softmax)
hypothesis = tf.nn.softmax(tf.matmul(X, W) + b)

cost = tf.reduce_mean(-tf.reduce_sum(Y * tf.log(hypothesis), axis=1))
optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.1).minimize(cost)

# Test model
is_correct = tf.equal(tf.arg_max(hypothesis, 1), tf.arg_max(Y, 1))
# Calculate accuracy
accuracy = tf.reduce_mean(tf.cast(is_correct, tf.float32))

# 전체 training data set을 한번 훈련시킨 것을 Epoch이라한다.
# 한번돌때의 크기를 batch size라한다.
training_epochs = 15
batch_size = 100

with tf.Session() as sess:
    # Initialize TensorFlow variables
    sess.run(tf.global_variables_initializer())
    # Training cycle
    for epoch in range(training_epochs):
        avg_cost = 0
        total_batch = int(mnist.train.num_examples / batch_size)

        for i in range(total_batch):
            batch_xs, batch_ys = mnist.train.next_batch(batch_size)
            c, _ = sess.run([cost, optimizer], feed_dict={
                            X: batch_xs, Y: batch_ys})
            avg_cost += c / total_batch

        print('Epoch:', '%04d' % (epoch + 1),
              'cost =', '{:.9f}'.format(avg_cost))

    print("Learning finished")

    # Test the model using test sets
    print("Accuracy: ", accuracy.eval(session=sess, feed_dict={
          X: mnist.test.images, Y: mnist.test.labels}))

    # Get one and predict
    r = random.randint(0, mnist.test.num_examples - 1)
    print("Label: ", sess.run(tf.argmax(mnist.test.labels[r:r + 1], 1)))
    print("Prediction: ", sess.run(
        tf.argmax(hypothesis, 1), feed_dict={X: mnist.test.images[r:r + 1]}))

    # don't know why this makes Travis Build error.
    # plt.imshow(
    #     mnist.test.images[r:r + 1].reshape(28, 28),
    #     cmap='Greys',
    #     interpolation='nearest')
    # plt.show()
```
```
Epoch: 0001 cost = 2.826302775
Epoch: 0002 cost = 1.061668983
Epoch: 0003 cost = 0.838061326
Epoch: 0004 cost = 0.733232750
Epoch: 0005 cost = 0.669279897
Epoch: 0006 cost = 0.624611847
Epoch: 0007 cost = 0.591160361
Epoch: 0008 cost = 0.563869004
Epoch: 0009 cost = 0.541745184
Epoch: 0010 cost = 0.522673592
Epoch: 0011 cost = 0.506782337
Epoch: 0012 cost = 0.492447654
Epoch: 0013 cost = 0.479955841
Epoch: 0014 cost = 0.468893682
Epoch: 0015 cost = 0.458703491
Learning finished
Accuracy:  0.8951
Label:  [5]
Prediction:  [5]
```
---
title: NN, ReLu, Xavier, Dropout, and Adam
categories:
- deeplearning
tag:
- python
- tensorflow
---

<a href="https://hunkim.github.io/ml/">모두를 위한 머신러닝/딥러닝 강의</a>를 참조했습니다.

## 이론

### ReLU : Better non-linearity

Sigmoid와 같은 함수를 Activation function이라 부른다. 일정 값을 넘으면 activate되기 때문이다.

텐서플로우로 Deep한 Neural Nets을 만드는 것은 간단하다.
```python
W1 = tf.Variable(tf.random_uniform([2,5],-1.0,1.0))
W2 = tf.Variable(tf.random_uniform([5,4],-1.0,1.0))
W3 = tf.Variable(tf.random_uniform([4,1],-1.0,1.0))

b1 = tf.Variable(tf.zeros([5]),name = "Bias1")
b2 = tf.Variable(tf.zeros([4]),name = "Bias2")
b3 = tf.Variable(tf.zeros([1]),name = "Bias3")

L2 = tf.sigmoid(tf.matmul(X,W1)+b1)
L3 = tf.sigmoid(tf.matmul(L2,W2)+b2)
hypothesis = tf.sigmoid(tf.matmul(L3,W3)+b3)
```

Input layer - hidden layer - output layer라고 보통 불린다.
chain rule으로 이렇게 더 깊게 들어갈 수 있다.

#### Vanishing gradient
근데 너무 깊게 하면 hidden layer가 1개나 2개일 때보다 결과가 나쁘게 나올 수 있다.
왜 이런문제가 나타날까??
2단, 3단 정도는 학습이 잘되는데 너무 깊은 layer는 학습이 안된다.
Backpropagation(역전파) 과정에서 항상 sigmoid를 통과하기때문에 0보다 크고 1보다작은 값이 나온다.
근데 그 값들을 계속해서 곱하다보면 결과값은 점점 작아져 0과 가까운 값이 된다.
이러한 문제점을 **Vanishing gradient** 문제라고 한다.

#### ReLU
![](http://cfile4.uf.tistory.com/image/22293C50579F7BBF1331FA)
Vanishing gradient problem을 해결하기 위해서 ReLU함수를 사용하면된다.
**ReLU**는 0보다 작을때는 끄고 0보다 클때는 값을 적용(activate)하는 함수이다.
```python
L1 = tf.nn.relu(tf.matmul(X,W1)+b1)
```

Sigmoid대신 ReLU를 사용하면 학습이 잘되는 것을 확인할 수 있다.
activation function으로 `sigmoid`,`ReLU`말고도 `maxout`,`VLReLU`,`tanh`를 사용한다.

### initialize weights in a smart way
초기 값이 잘못 적용된 것이다.
#### Need to set the initial weight values wisely
- 전체를 0으로 설정하면 안된다.
- RBM(Restircted Blotzmann machine)
	- RBM을 사용하지 않아도 된다.
	- 굉장히 간단한 methods를 써도 괜찮다.(**Xavier** : weights를 너무 작거나 크지만 않게 설정하면된다.)

### NN dropout and model ensemble

#### Overfitting
학습 데이터(training dataset)에서는 정확도가 굉장히 높은데 test dataset에서는 정확도가 낮게나오는 경우를 overfitting이라고 한다. 이 경우에는 실제로 사용하면 좋지않다.

#### 해결책
- training data가 많을 수록 좋다.
- Regularization(정규화)를 해준다.

#### Dropout
모든 노드를 사용하는게 아니라 랜덤하게 neurons을 0으로 설정해서 쉬게하는 방법이다.
![](http://cfile26.uf.tistory.com/image/2264014957A0248C2D18CD)

이때 중요한 점은 학습(training)할때만 dropout을 시키고, 실제로 test할때는 반드시 전체를 불러와야한다.

#### Ensemble
독립적으로 NN집단을 만든다. 각각의 NN을 학습을 시킨후 나중에 합치는 방법이다.
대략 2%의 성능이 45%까지 발전했다. 실제로 사용할때는 Ensemble을 사용하면 좋다.

## 실습
```python
import tensorflow as tf
import random
# import matplotlib.pyplot as plt

from tensorflow.examples.tutorials.mnist import input_data

tf.set_random_seed(777)  # reproducibility

mnist = input_data.read_data_sets("MNIST_data/", one_hot=True)
# Check out https://www.tensorflow.org/get_started/mnist/beginners for
# more information about the mnist dataset

# parameters
learning_rate = 0.001
training_epochs = 15
batch_size = 100

# input place holders
X = tf.placeholder(tf.float32, [None, 784])
Y = tf.placeholder(tf.float32, [None, 10])

# weights & bias for nn layers
W1 = tf.Variable(tf.random_normal([784, 256]))
b1 = tf.Variable(tf.random_normal([256]))
L1 = tf.nn.relu(tf.matmul(X, W1) + b1)

W2 = tf.Variable(tf.random_normal([256, 256]))
b2 = tf.Variable(tf.random_normal([256]))
L2 = tf.nn.relu(tf.matmul(L1, W2) + b2)

W3 = tf.Variable(tf.random_normal([256, 10]))
b3 = tf.Variable(tf.random_normal([10]))
hypothesis = tf.matmul(L2, W3) + b3

# define cost/loss & optimizer
cost = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(
    logits=hypothesis, labels=Y))
optimizer = tf.train.AdamOptimizer(learning_rate=learning_rate).minimize(cost)

# initialize
sess = tf.Session()
sess.run(tf.global_variables_initializer())

# train my model
for epoch in range(training_epochs):
    avg_cost = 0
    total_batch = int(mnist.train.num_examples / batch_size)

    for i in range(total_batch):
        batch_xs, batch_ys = mnist.train.next_batch(batch_size)
        feed_dict = {X: batch_xs, Y: batch_ys}
        c, _ = sess.run([cost, optimizer], feed_dict=feed_dict)
        avg_cost += c / total_batch

    print('Epoch:', '%04d' % (epoch + 1), 'cost =', '{:.9f}'.format(avg_cost))

print('Learning Finished!')

# Test model and check accuracy
correct_prediction = tf.equal(tf.argmax(hypothesis, 1), tf.argmax(Y, 1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
print('Accuracy:', sess.run(accuracy, feed_dict={
      X: mnist.test.images, Y: mnist.test.labels}))

# Get one and predict
r = random.randint(0, mnist.test.num_examples - 1)
print("Label: ", sess.run(tf.argmax(mnist.test.labels[r:r + 1], 1)))
print("Prediction: ", sess.run(
    tf.argmax(hypothesis, 1), feed_dict={X: mnist.test.images[r:r + 1]}))

# plt.imshow(mnist.test.images[r:r + 1].
#           reshape(28, 28), cmap='Greys', interpolation='nearest')
# plt.show()
```
#### Xavier
```python
# weights & bias for nn layers
# http://stackoverflow.com/questions/33640581/how-to-do-xavier-initialization-on-tensorflow
W1 = tf.get_variable("W1", shape=[784, 256],
                     initializer=tf.contrib.layers.xavier_initializer())
b1 = tf.Variable(tf.random_normal([256]))
L1 = tf.nn.relu(tf.matmul(X, W1) + b1)

W2 = tf.get_variable("W2", shape=[256, 256],
                     initializer=tf.contrib.layers.xavier_initializer())
b2 = tf.Variable(tf.random_normal([256]))
L2 = tf.nn.relu(tf.matmul(L1, W2) + b2)

W3 = tf.get_variable("W3", shape=[256, 10],
                     initializer=tf.contrib.layers.xavier_initializer())
b3 = tf.Variable(tf.random_normal([10]))
hypothesis = tf.matmul(L2, W3) + b3
```
이 부분에서 `xavier_initializer()`로 바뀌었다.

#### Deep
더 깊고 넓게 만들었다.
```python
import tensorflow as tf
import random
# import matplotlib.pyplot as plt

from tensorflow.examples.tutorials.mnist import input_data

tf.set_random_seed(777)  # reproducibility

mnist = input_data.read_data_sets("MNIST_data/", one_hot=True)
# Check out https://www.tensorflow.org/get_started/mnist/beginners for
# more information about the mnist dataset

# parameters
learning_rate = 0.001
training_epochs = 15
batch_size = 100

# input place holders
X = tf.placeholder(tf.float32, [None, 784])
Y = tf.placeholder(tf.float32, [None, 10])

# weights & bias for nn layers
# http://stackoverflow.com/questions/33640581/how-to-do-xavier-initialization-on-tensorflow
W1 = tf.get_variable("W1", shape=[784, 512],
                     initializer=tf.contrib.layers.xavier_initializer())
b1 = tf.Variable(tf.random_normal([512]))
L1 = tf.nn.relu(tf.matmul(X, W1) + b1)

W2 = tf.get_variable("W2", shape=[512, 512],
                     initializer=tf.contrib.layers.xavier_initializer())
b2 = tf.Variable(tf.random_normal([512]))
L2 = tf.nn.relu(tf.matmul(L1, W2) + b2)

W3 = tf.get_variable("W3", shape=[512, 512],
                     initializer=tf.contrib.layers.xavier_initializer())
b3 = tf.Variable(tf.random_normal([512]))
L3 = tf.nn.relu(tf.matmul(L2, W3) + b3)

W4 = tf.get_variable("W4", shape=[512, 512],
                     initializer=tf.contrib.layers.xavier_initializer())
b4 = tf.Variable(tf.random_normal([512]))
L4 = tf.nn.relu(tf.matmul(L3, W4) + b4)

W5 = tf.get_variable("W5", shape=[512, 10],
                     initializer=tf.contrib.layers.xavier_initializer())
b5 = tf.Variable(tf.random_normal([10]))
hypothesis = tf.matmul(L4, W5) + b5

# define cost/loss & optimizer
cost = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(
    logits=hypothesis, labels=Y))
optimizer = tf.train.AdamOptimizer(learning_rate=learning_rate).minimize(cost)

# initialize
sess = tf.Session()
sess.run(tf.global_variables_initializer())

# train my model
for epoch in range(training_epochs):
    avg_cost = 0
    total_batch = int(mnist.train.num_examples / batch_size)

    for i in range(total_batch):
        batch_xs, batch_ys = mnist.train.next_batch(batch_size)
        feed_dict = {X: batch_xs, Y: batch_ys}
        c, _ = sess.run([cost, optimizer], feed_dict=feed_dict)
        avg_cost += c / total_batch

    print('Epoch:', '%04d' % (epoch + 1), 'cost =', '{:.9f}'.format(avg_cost))

print('Learning Finished!')

# Test model and check accuracy
correct_prediction = tf.equal(tf.argmax(hypothesis, 1), tf.argmax(Y, 1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
print('Accuracy:', sess.run(accuracy, feed_dict={
      X: mnist.test.images, Y: mnist.test.labels}))

# Get one and predict
r = random.randint(0, mnist.test.num_examples - 1)
print("Label: ", sess.run(tf.argmax(mnist.test.labels[r:r + 1], 1)))
print("Prediction: ", sess.run(
    tf.argmax(hypothesis, 1), feed_dict={X: mnist.test.images[r:r + 1]}))

# plt.imshow(mnist.test.images[r:r + 1].
#           reshape(28, 28), cmap='Greys', interpolation='nearest')
# plt.show()
```
256에서 512로, 깊이도 5단계로 더 늘렸다. 하지만 결과를 보면 정확도가 더 떨어지는 것을 볼 수 있다. 아마도 overfitting때문일 것이다.

#### Dropout
```python
# dropout (keep_prob) rate  0.7 on training, but should be 1 for testing
keep_prob = tf.placeholder(tf.float32)

W1 = tf.get_variable("W1", shape=[784, 512],
                     initializer=tf.contrib.layers.xavier_initializer())
b1 = tf.Variable(tf.random_normal([512]))
L1 = tf.nn.relu(tf.matmul(X, W1) + b1)
L1 = tf.nn.dropout(L1, keep_prob=keep_prob)

W2 = tf.get_variable("W2", shape=[512, 512],
                     initializer=tf.contrib.layers.xavier_initializer())
b2 = tf.Variable(tf.random_normal([512]))
L2 = tf.nn.relu(tf.matmul(L1, W2) + b2)
L2 = tf.nn.dropout(L2, keep_prob=keep_prob)

W3 = tf.get_variable("W3", shape=[512, 512],
                     initializer=tf.contrib.layers.xavier_initializer())
b3 = tf.Variable(tf.random_normal([512]))
L3 = tf.nn.relu(tf.matmul(L2, W3) + b3)
L3 = tf.nn.dropout(L3, keep_prob=keep_prob)

W4 = tf.get_variable("W4", shape=[512, 512],
                     initializer=tf.contrib.layers.xavier_initializer())
b4 = tf.Variable(tf.random_normal([512]))
L4 = tf.nn.relu(tf.matmul(L3, W4) + b4)
L4 = tf.nn.dropout(L4, keep_prob=keep_prob)

W5 = tf.get_variable("W5", shape=[512, 10],
                     initializer=tf.contrib.layers.xavier_initializer())
b5 = tf.Variable(tf.random_normal([10]))
hypothesis = tf.matmul(L4, W5) + b5

...

for epoch in range(training_epochs):
    avg_cost = 0
    total_batch = int(mnist.train.num_examples / batch_size)

    for i in range(total_batch):
        batch_xs, batch_ys = mnist.train.next_batch(batch_size)
        feed_dict = {X: batch_xs, Y: batch_ys, keep_prob: 0.7}
        c, _ = sess.run([cost, optimizer], feed_dict=feed_dict)
        avg_cost += c / total_batch

    print('Epoch:', '%04d' % (epoch + 1), 'cost =', '{:.9f}'.format(avg_cost))

print('Learning Finished!')

# Test model and check accuracy
correct_prediction = tf.equal(tf.argmax(hypothesis, 1), tf.argmax(Y, 1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
print('Accuracy:', sess.run(accuracy, feed_dict={
      X: mnist.test.images, Y: mnist.test.labels, keep_prob: 1}))
```
Layer를 각각 더 생성해 dropout을 만들어준다. 학습(training)할때는 0.5 ~ 0.7정도로 주어진다. Test할때는 무조건 1로 줘야한다.

여러가지의 Optimizer가 있는데 그 중에서 `Adam`이 상당히 좋은 결과를 내고 있는 지표를 확인할 수 있다.
```python
optimizer = tf.train.AdamOptimizer(learning_rate=learning_rate).minimize(cost)
```
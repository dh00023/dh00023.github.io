---
title: "06. Multinomial Classification - Softmax Classification"
tag: tensorflow
---

## 이론

### Logistic regression
![](http://cfile26.uf.tistory.com/image/236A814058BE07222FC2F5)
위의 그래프에서 A와 B를 구분하는 선을 학습하는 것이다.

### Multinomial Classification

여러개의 값을 예측할 때!

![](http://cfile6.uf.tistory.com/image/215DE63B58BE002609B1F5)
여기서 3개의 선으로 구별을 할 수 있다.
1. A or not
2. B or not
3. C or not

![](http://cfile26.uf.tistory.com/image/232FE93358BE093C2DB375)

![](http://cfile28.uf.tistory.com/image/22361D3558BE3360331402)

Vector로 결과 값이 나오게된다.

이때 결과값의 합이 1이 나오면 좋겠다. ==> **Softmax**

#### Softmax
![](http://andersonjo.github.io/assets/posts/TensorFlow-Softmax-Regression/udacity_logistic02.png)
1. 0~1사의 값
2. 전체의 합이 1이된다.

그러면 각각을 확률로 볼 수 있다.

**One-Hot Encoding**은 가장 큰 값을 1, 나며지는 0


#### Cost function
![](http://cfile29.uf.tistory.com/image/2153C64F5797F9C32E9735)

S(y)는 softmax가 예측한 값이고, L(y)는 실제 Y의 값이다.

![](http://cfile23.uf.tistory.com/image/243B804E579806C11E2A8F)
이때, 실제의 값과 예측이 같을 경우에는 cost가 0이나오지만 틀릴경우에는 무한대로 큰 값이 나오는 것을 볼 수 있다.

## 실습

```python
import tensorflow as tf

x_data = [[1,2,1,1],[2,1,3,2],[3,1,3,4],[4,1,5,5],[1,7,5,5],[1,2,5,6],[1,6,6,6],[1,7,7,7]]
# y_data를 one-hot encoding으로 나타냄
# one-hot encoding : tf.arg_max(a,1) 하나만 핫하게 한다.
y_data = [[0,0,1],[0,0,1],[0,0,1],[0,1,0],[0,1,0],[0,1,0],[1,0,0],[1,0,0]]

X = tf.placeholder("float",[None,4])
Y = tf.placeholder("float",[None,3])
nb_classes = 3

W = tf.Variable(tf.random_normal([4,nb_classes]),name="weight")
b = tf.Variable(tf.random_normal([nb_classes]),name="bias")

# softmax
hypothesis=tf.nn.softmax(tf.matmul(X,W)+b)

cost = tf.reduce_mean(-tf.reduce_sum(Y*tf.log(hypothesis),axis=1))
optimizer = tf.train.GradientDescentOptimizer(learning_rate = 0.1).minimize(cost)


with tf.Session() as sess:
	sess.run(tf.global_variables_initializer())

	for step in range(2001):
		sess.run(optimizer,feed_dict={X: x_data,Y: y_data})
		if step % 200 == 0:
			print(step,sess.run(cost,feed_dict={X: x_data, Y: y_data}))
	print('--------------------')
	# Testing & one-hot encoding
	a = sess.run(hypothesis, feed_dict={X: [[1,11,7,9]]})
	print(a,sess.run(tf.arg_max(a,1))) # one-hot encoding으로 가장 큰 값의 위치 출력

	print('--------------------')
	b = sess.run(hypothesis,feed_dict={X: [[1,3,4,3]]})
	print(b,sess.run(tf.arg_max(b,1)))

	print('--------------------')
	c = sess.run(hypothesis,feed_dict={X: [[1,1,0,1]]})
	print(c,sess.run(tf.arg_max(c,1)))

	print('--------------------')
	all = sess.run(hypothesis,feed_dict={X: [[1,11,7,9],[1,3,4,3],[1,1,0,1]]})
	print(all,sess.run(tf.arg_max(all,1)))
```
```
0 10.8421
200 0.556759
400 0.461243
600 0.387494
800 0.318165
1000 0.24816
1200 0.218706
1400 0.199462
1600 0.183214
1800 0.169323
2000 0.157319
--------------------
[[  5.80435386e-03   9.94184315e-01   1.13274800e-05]] [1]
--------------------
[[ 0.84905654  0.13018762  0.02075585]] [0]
--------------------
[[  1.75439148e-08   3.69824149e-04   9.99630094e-01]] [2]
--------------------
[[  5.80435386e-03   9.94184315e-01   1.13274800e-05]
 [  8.49056542e-01   1.30187616e-01   2.07558516e-02]
 [  1.75439165e-08   3.69824149e-04   9.99630094e-01]] [1 0 2]
```

```python
import tensorflow as tf
import numpy as np
tf.set_random_seed(777)  # for reproducibility

# Predicting animal type based on various features
xy = np.loadtxt('data06.csv', delimiter=',', dtype=np.float32)
x_data = xy[:, 0:-1]
y_data = xy[:, [-1]]

print(x_data.shape, y_data.shape)

nb_classes = 7  # 0 ~ 6

X = tf.placeholder(tf.float32, [None, 16])
Y = tf.placeholder(tf.int32, [None, 1])  # 0 ~ 6
# one hot으로 바꿔주는 함수
# 차원이 한개가 추가된다.(rank = N에서 rank N+1)(?,1,7)
Y_one_hot = tf.one_hot(Y, nb_classes)
print("one_hot", Y_one_hot)
# reshape으로 해결한다.(?,7)
Y_one_hot = tf.reshape(Y_one_hot, [-1, nb_classes])
print("reshape", Y_one_hot)

W = tf.Variable(tf.random_normal([16, nb_classes]), name='weight')
b = tf.Variable(tf.random_normal([nb_classes]), name='bias')

# tf.nn.softmax computes softmax activations
# softmax = exp(logits) / reduce_sum(exp(logits), dim)
logits = tf.matmul(X, W) + b
hypothesis = tf.nn.softmax(logits)

# Cross entropy cost/loss
# 여기서 logit이 입력값으로 들어간다는 점을 주의해야한다.
cost_i = tf.nn.softmax_cross_entropy_with_logits(logits=logits,
                                                 labels=Y_one_hot)
# 이 cost는 cost = tf.reduce_mean(-tf.reduce_sum(Y*tf.log(hypothesis),axis=1))과 같은 값이다.
cost = tf.reduce_mean(cost_i)
optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.1).minimize(cost)

prediction = tf.argmax(hypothesis, 1)
correct_prediction = tf.equal(prediction, tf.argmax(Y_one_hot, 1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
# Launch graph
with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())

    for step in range(2000):
        sess.run(optimizer, feed_dict={X: x_data, Y: y_data})
        if step % 100 == 0:
            loss, acc = sess.run([cost, accuracy], feed_dict={
                                 X: x_data, Y: y_data})
            print("Step: {:5}\tLoss: {:.3f}\tAcc: {:.2%}".format(
                step, loss, acc))

    # Let's see if we can predict
    pred = sess.run(prediction, feed_dict={X: x_data})
    # y_data: (N,1) = flatten => (N, ) matches pred.shape
    # flatten을 이용해서 [[1],[0]] => [1,0]로 바꿔준다.
    for p, y in zip(pred, y_data.flatten()):
        print("[{}] Prediction: {} True Y: {}".format(p == int(y), p, int(y)))
```
```
Step:     0	Loss: 5.106	Acc: 37.62%
Step:   100	Loss: 0.800	Acc: 79.21%
Step:   200	Loss: 0.486	Acc: 88.12%
Step:   300	Loss: 0.349	Acc: 90.10%
Step:   400	Loss: 0.272	Acc: 94.06%
Step:   500	Loss: 0.222	Acc: 95.05%
Step:   600	Loss: 0.187	Acc: 97.03%
Step:   700	Loss: 0.161	Acc: 97.03%
Step:   800	Loss: 0.140	Acc: 97.03%
Step:   900	Loss: 0.124	Acc: 97.03%
Step:  1000	Loss: 0.111	Acc: 97.03%
Step:  1100	Loss: 0.101	Acc: 99.01%
Step:  1200	Loss: 0.092	Acc: 100.00%
Step:  1300	Loss: 0.084	Acc: 100.00%
Step:  1400	Loss: 0.078	Acc: 100.00%
Step:  1500	Loss: 0.072	Acc: 100.00%
Step:  1600	Loss: 0.068	Acc: 100.00%
Step:  1700	Loss: 0.064	Acc: 100.00%
Step:  1800	Loss: 0.060	Acc: 100.00%
Step:  1900	Loss: 0.057	Acc: 100.00%
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 3 True Y: 3
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 3 True Y: 3
[True] Prediction: 3 True Y: 3
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 1 True Y: 1
[True] Prediction: 3 True Y: 3
[True] Prediction: 6 True Y: 6
[True] Prediction: 6 True Y: 6
[True] Prediction: 6 True Y: 6
[True] Prediction: 1 True Y: 1
[True] Prediction: 0 True Y: 0
[True] Prediction: 3 True Y: 3
[True] Prediction: 0 True Y: 0
[True] Prediction: 1 True Y: 1
[True] Prediction: 1 True Y: 1
[True] Prediction: 0 True Y: 0
[True] Prediction: 1 True Y: 1
[True] Prediction: 5 True Y: 5
[True] Prediction: 4 True Y: 4
[True] Prediction: 4 True Y: 4
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 5 True Y: 5
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 1 True Y: 1
[True] Prediction: 3 True Y: 3
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 1 True Y: 1
[True] Prediction: 3 True Y: 3
[True] Prediction: 5 True Y: 5
[True] Prediction: 5 True Y: 5
[True] Prediction: 1 True Y: 1
[True] Prediction: 5 True Y: 5
[True] Prediction: 1 True Y: 1
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 6 True Y: 6
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 5 True Y: 5
[True] Prediction: 4 True Y: 4
[True] Prediction: 6 True Y: 6
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 1 True Y: 1
[True] Prediction: 1 True Y: 1
[True] Prediction: 1 True Y: 1
[True] Prediction: 1 True Y: 1
[True] Prediction: 3 True Y: 3
[True] Prediction: 3 True Y: 3
[True] Prediction: 2 True Y: 2
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 1 True Y: 1
[True] Prediction: 6 True Y: 6
[True] Prediction: 3 True Y: 3
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 2 True Y: 2
[True] Prediction: 6 True Y: 6
[True] Prediction: 1 True Y: 1
[True] Prediction: 1 True Y: 1
[True] Prediction: 2 True Y: 2
[True] Prediction: 6 True Y: 6
[True] Prediction: 3 True Y: 3
[True] Prediction: 1 True Y: 1
[True] Prediction: 0 True Y: 0
[True] Prediction: 6 True Y: 6
[True] Prediction: 3 True Y: 3
[True] Prediction: 1 True Y: 1
[True] Prediction: 5 True Y: 5
[True] Prediction: 4 True Y: 4
[True] Prediction: 2 True Y: 2
[True] Prediction: 2 True Y: 2
[True] Prediction: 3 True Y: 3
[True] Prediction: 0 True Y: 0
[True] Prediction: 0 True Y: 0
[True] Prediction: 1 True Y: 1
[True] Prediction: 0 True Y: 0
[True] Prediction: 5 True Y: 5
[True] Prediction: 0 True Y: 0
[True] Prediction: 6 True Y: 6
[True] Prediction: 1 True Y: 1
```

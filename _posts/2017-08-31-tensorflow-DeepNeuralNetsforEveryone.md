---
title: "08. Deep Neural Nets for Everyone"
tag: tensorflow
---

## 이론

### activation function
어떤 일정 값을 넘어가면 1 아니면 0을 반환하는 함수이다.
여러개의 출력을 동시에 낼 수 있다.
![](https://i.stack.imgur.com/iIcbq.gif)

### XOR problem
xor은 두개의 값이 같으면 0, 다르면 1을 출력한다.
Linear하게 어떠한 선을 그어도 100%가 안된다.

### Perceptrons - by Minsky
XOR도 여러개를 합치면 할수있다.(multilayer neural nets)
하지만 문제는 W(가중치)와 b(bias)를 학습 시킬 수 없는 것이다.

### Backpropagation(역전파) - by Hinton

error를 구해서 뒤로 돌아가면서 수정하면 어떨까??
더 복잡한 형태도 접근이 가능해졌다.
![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcT3J_uUPOLa4wAAoc-kWv0_TXrsOu56Pcu7ukaea8dcpu8qcEwW)

### Convlutional Neural Networks
부분부분 역할이 나누어져 있고, 나중에 합치는 방법을 생각해 냈다.
문자를 인식하고 그림을 인식하는데 약 90% 이상의 성능이 보여졌다.

### 문제점
- Backpropagation은 조금의 layer에서는 동작하지만, 층이많을 수록 성능이 떨어지는 문제가 발생했다.
- 훨씬 더 간단한 알고리즘들이 나왔다. (SVM, RandomForest 등등)

### Breakthrough
각각의 W(weight) 초깃값을 잘 준다면 학습할 수 있으며, 깊게 신경망을 구축하면 복잡한 문제도 해결할 수 다.
=> 나중에 Deep Nets, Deep Learning으로 이름 바뀜

## 실습
```python
import tensorflow as tf
import numpy as np
import pprint
tf.set_random_seed(777)  # for reproducibility

pp = pprint.PrettyPrinter(indent=4)
sess = tf.InteractiveSession()

# Simple Array
t = np.array([0., 1., 2., 3., 4., 5., 6.])
pp.pprint(t)
print(t.ndim) # rank 배열의 차원을 알려줌
print(t.shape) # shape 모양은 어떻게 생겼는지
print(t[0], t[1], t[-1])
print(t[2:5], t[4:-1]) # 2:5면 2,3,4 인덱스 부분 출력
print(t[:2], t[3:]) # 처음부터 2번째까지, 3번째부터 끝까지
```
```
array([ 0.,  1.,  2.,  3.,  4.,  5.,  6.])
1
(7,)
0.0 1.0 6.0
[ 2.  3.  4.] [ 4.  5.]
[ 0.  1.] [ 3.  4.  5.  6.]
```
```python
# 2D Array
t = np.array([[1., 2., 3.], [4., 5., 6.], [7., 8., 9.], [10., 11., 12.]])
pp.pprint(t)
print(t.ndim) # rank
print(t.shape) # shape
```
```
array([[  1.,   2.,   3.],
       [  4.,   5.,   6.],
       [  7.,   8.,   9.],
       [ 10.,  11.,  12.]])
2
(4, 3)
```
```python
# Shape, Rank, Axis
t = tf.constant([1,2,3,4])
tf.shape(t).eval()

t = tf.constant([[1,2],
                 [3,4]])
tf.shape(t).eval()

t = tf.constant([[[[1, 2, 3, 4], [5, 6, 7, 8], [9, 10, 11, 12]],[[13, 14, 15, 16], [17, 18, 19, 20], [21, 22, 23, 24]]]])
tf.shape(t).eval()
```
```
[4]
[2 2]
[1 2 3 4]
```
Axis 축은 가장 안쪽에 있는게 가장 큰 값을 가진다. 0부터 시작해서 안쪽으로 들어간다.

```python
# matmul VS multiply
matrix1 = tf.constant([[3., 3.]])
matrix2 = tf.constant([[2.],[2.]])
print("Matrix 1 shape",matrix1.shape)
print("Matrix 2 shape",matrix2.shape)
print(tf.matmul(matrix1, matrix2).eval())
```
```
Matrix 1 shape (1, 2)
Matrix 2 shape (2, 1)
[[ 12.]]
```
```python
# Watch out broadcasting
matrix1 = tf.constant([[3., 3.]])
matrix2 = tf.constant([[2.],[2.]])
(matrix1+matrix2).eval()

matrix1 = tf.constant([[3., 3.]])
matrix2 = tf.constant([[2., 2.]])
(matrix1+matrix2).eval()
```
```
[[ 5.  5.]
 [ 5.  5.]]
[[ 5.  5.]]
```
격이 다른 형태를 맞춰준다. 잘모르면서 사용하면 우리가 생각하지 않는 값이 나오므로 주의해서 사용해야한다.

```python
# Random values for variable initializations
a = tf.random_normal([3]).eval()
b = tf.random_uniform([2]).eval()
c = tf.random_uniform([2, 3]).eval()
```
```
[ 0.9833048   0.46072197 -0.74365181]
[ 0.8153863   0.61427879]
[[ 0.21454096  0.43489015  0.52136803]
 [ 0.1903218   0.05559099  0.32977045]]
```
```python
# Reduce Mean/Sum
tf.reduce_mean([1, 2], axis=0).eval()

x = [[1., 2.],
     [3., 4.]]
tf.reduce_mean(x).eval()
# 열의 평균
tf.reduce_mean(x, axis=0).eval()
# 행의 평균
tf.reduce_mean(x, axis=1).eval()
# 가장 안쪽에 있는 평균
tf.reduce_mean(x, axis=-1).eval()
tf.reduce_sum(x).eval()
tf.reduce_sum(x, axis=0).eval()
tf.reduce_sum(x, axis=-1).eval()
tf.reduce_mean(tf.reduce_sum(x, axis=-1)).eval()
```
```
1
2.5
[ 2.  3.]
[ 1.5  3.5]
[ 1.5  3.5]
10.0
[ 4.  6.]
[ 3.  7.]
5.0
```
```python
# Argmax with axis
# 가장 큰 수의 위치를 출력해준다.
x = [[0, 1, 2],
     [2, 1, 0]]
tf.argmax(x, axis=0).eval()
tf.argmax(x, axis=1).eval()
tf.argmax(x, axis=-1).eval()
```
```
[1 0 0]
[2 0]
[2 0]
```
```python
# Reshape, squeeze, expand_dims
t = np.array([[[0, 1, 2], 
               [3, 4, 5]],
              
              [[6, 7, 8], 
               [9, 10, 11]]])
print(t.shape)
# 가장 안쪽에 있는 값은 잘 변경하지 않고 전체의 shape을 조정한다.
tf.reshape(t, shape=[-1, 3]).eval()
tf.reshape(t, shape=[-1, 1, 3]).eval()
tf.squeeze([[0], [1], [2]]).eval()
# 차원을 추가하고 싶을 때
tf.expand_dims([0, 1, 2], 1).eval()
```
```
(2, 2, 3)
[[ 0  1  2]
 [ 3  4  5]
 [ 6  7  8]
 [ 9 10 11]]
[[[ 0  1  2]]

 [[ 3  4  5]]

 [[ 6  7  8]]

 [[ 9 10 11]]]
[0 1 2]
[[0]
 [1]
 [2]]
```

```python
# One hot
# 가장 큰 자리 하나만 hot하게 1로 바꿔주고 나머지는 0으로 바꿔주는 것이다.
tf.one_hot([[0], [1], [2], [0]], depth=3).eval()
t = tf.one_hot([[0], [1], [2], [0]], depth=3)
tf.reshape(t, shape=[-1, 3]).eval()
```
```
[[[ 1.  0.  0.]]

 [[ 0.  1.  0.]]

 [[ 0.  0.  1.]]

 [[ 1.  0.  0.]]]
[[ 1.  0.  0.]
 [ 0.  1.  0.]
 [ 0.  0.  1.]
 [ 1.  0.  0.]]
```
```python
# casting
tf.cast([1.8, 2.2, 3.3, 4.9], tf.int32).eval()
tf.cast([True, False, 1 == 1, 0 == 1], tf.int32).eval()
```
```
[1 2 3 4]
[1 0 1 0]
```

```python
# stack
x = [1, 4]
y = [2, 5]
z = [3, 6]

# Pack along first dim.
tf.stack([x, y, z]).eval()
tf.stack([x, y, z], axis=1).eval()
```
```
[[1 4]
 [2 5]
 [3 6]]
[[1 2 3]
 [4 5 6]]
```
```python
# Ones like and Zeros like
# 1과 0으로 채워준다.
x = [[0, 1, 2],
     [2, 1, 0]]

tf.ones_like(x).eval()
tf.zeros_like(x).eval()
```
```
[[1 1 1]
 [1 1 1]]
[[0 0 0]
 [0 0 0]]
```
```python
# Zip
for x, y in zip([1, 2, 3], [4, 5, 6]):
    print(x, y)
for x, y, z in zip([1, 2, 3], [4, 5, 6], [7, 8, 9]):
    print(x, y, z)
```
```
1 4
2 5
3 6
1 4 7
2 5 8
3 6 9
```
```python
# Transpose
t = np.array([[[0, 1, 2], [3, 4, 5]], [[6, 7, 8], [9, 10, 11]]])
pp.pprint(t.shape)
pp.pprint(t)
t1 = tf.transpose(t, [1, 0, 2])
pp.pprint(sess.run(t1).shape)
pp.pprint(sess.run(t1))
t = tf.transpose(t1, [1, 0, 2])
pp.pprint(sess.run(t).shape)
pp.pprint(sess.run(t))
t2 = tf.transpose(t, [1, 2, 0])
pp.pprint(sess.run(t2).shape)
pp.pprint(sess.run(t2))
t = tf.transpose(t2, [2, 0, 1])
pp.pprint(sess.run(t).shape)
pp.pprint(sess.run(t))
```
```
(2, 2, 3)
array([[[ 0,  1,  2],
        [ 3,  4,  5]],

       [[ 6,  7,  8],
        [ 9, 10, 11]]])
(2, 2, 3)
array([[[ 0,  1,  2],
        [ 6,  7,  8]],

       [[ 3,  4,  5],
        [ 9, 10, 11]]])
(2, 2, 3)
array([[[ 0,  1,  2],
        [ 3,  4,  5]],

       [[ 6,  7,  8],
        [ 9, 10, 11]]])
(2, 3, 2)
array([[[ 0,  6],
        [ 1,  7],
        [ 2,  8]],

       [[ 3,  9],
        [ 4, 10],
        [ 5, 11]]])
(2, 2, 3)
array([[[ 0,  1,  2],
        [ 3,  4,  5]],

       [[ 6,  7,  8],
        [ 9, 10, 11]]])
```
---
title: "Neural Networks"
tag: deeplearning
---

신경망(Neural networks)은 `torch.nn` 패키지를 사용해 만들 수 있다.
`nn.Module`은 layers(층)과 output을 반환하는 `forward(input)`메소드를 포함하고 있다.

예를 들어, 이미지를 분류하는 네트워크를 보자.
![](http://pytorch.org/tutorials/_images/mnist.png)

간단한 **feed-forward** 네트워크이다. input과 feeds를 받아 여러개의 층을 통과한뒤 output을 출력해낸다.

- 전형적인 신경망의 training 절차
	- 학습가능한 파라미터 또는 가중치가 있는 신경망을 정의한다.
	- input dataset를 반복 훈련한다.
	- 네트워크를 통해 input값을 처리한다.
	- loss(결과가 정확한 값으로 부터 얼마나 멀리있는지)를 계산한다.
	- gradients를 네트워크 파라미터로 전파한다.
	- `weight = weight - learning_rate * gradient`와 같이 간단한 규칙을 사용해 네트워크의 가중치를 갱신한다.

### Define the network(네트워크 정의하기)

```python
import torch
from torch.autograd import Variable
import torch.nn as nn
import torch.nn.functional as F


class Net(nn.Module):

    def __init__(self):
        super(Net, self).__init__()
        # 1 input image channel, 6 output channels, 5x5 square convolution
        # kernel
        self.conv1 = nn.Conv2d(1, 6, 5)
        self.conv2 = nn.Conv2d(6, 16, 5)
        # an affine operation: y = Wx + b
        self.fc1 = nn.Linear(16 * 5 * 5, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

    def forward(self, x):
        # Max pooling over a (2, 2) window
        x = F.max_pool2d(F.relu(self.conv1(x)), (2, 2))
        # If the size is a square you can only specify a single number
        x = F.max_pool2d(F.relu(self.conv2(x)), 2)
        x = x.view(-1, self.num_flat_features(x))
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x

    def num_flat_features(self, x):
        size = x.size()[1:]  # all dimensions except the batch dimension
        num_features = 1
        for s in size:
            num_features *= s
        return num_features


net = Net()
print(net)
```
```
Net (
  (conv1): Conv2d(1, 6, kernel_size=(5, 5), stride=(1, 1))
  (conv2): Conv2d(6, 16, kernel_size=(5, 5), stride=(1, 1))
  (fc1): Linear (400 -> 120)
  (fc2): Linear (120 -> 84)
  (fc3): Linear (84 -> 10)
)
```
직접 `forward` 함수를 정의하면, `backwoard`함수(gradients가 계산되는)는 `autograd`를 사용해 자동으로 정의된다. `forward`함수의 `Tensor`연산을 사용할 수 있다.

모델의 학습할 수 있는 파라미터들은 `net.parameters()`에 의해서 반환된다.

```python
params = list(net.parameters())
print(len(params))
print(params[0].size())  # conv1's .weight
```
```
10
torch.Size([6, 1, 5, 5])
```

`forward`의 input과 output은 `autograd.Variable`이다.

```python
input = Variable(torch.randn(1, 1, 32, 32))
out = net(input)
print(out)
```
```
Variable containing:
 0.0653 -0.1383  0.0357  0.1132 -0.0363  0.1177  0.0418  0.0548 -0.0306  0.0047
[torch.FloatTensor of size 1x10]
```

랜덤 gradients가 있는 모든 파라미터와 역전파 gradient버퍼를 제로화하면

```python
net.zero_grad()
out.backward(torch.randn(1, 10))
```

> `torch.nn`은 오직 mini-batches만 지원한다. 전반적인 `torch.nn`패키지는  단일 샘플이 아닌mini-batch의 샘플만 지원한다.
예를 들어, `nn.Conv2d` 는 `nSamples x nChannels x Height x Width` 4차원 Tensor를 가져온다.
만약 단일 샘플이 있다면, `input.unsquessze(0)을 사용해 가짜 배치차원을 추가하면된다.

---
- `torch.Tensor` : 다차원배열
- `autograd.Variable` : Tensor를 감싸고 적용된 연산을 기록한다. Tensor와 동일한 API를 가지고 있으면서 `backward()`와 같은 추가적인 API도 가지고 있다. 또한 Tensor와 연관된 gradient도 가지고 있다.
- `nn.Module` : 신경망 모듈이다. GPU로 이동, 보내기, 로드 등의 보조작업을 통해서 파라미터 캡슐화를 편리하게 해준다.
- `nn.Parameter` : 변수의 종류이다. 모듈의 속성으로 할당할 때 자동으로 파라미터로 등록된다.
- `autograd.Function` : autograd 연산의 forward와 backward의 정의를 생성한다. 모든 Variable연산은 Variable을 생성하고 그 기록을 인코딩하는 함수에 연결되는 최소 한개의 Fucntion 노드를 생성한다.

### Loss Function
loss function은 (output, target)형태의 입력을 받아 target으로부터 outpu이 얼마나 먼지 추정한 값을 계산한다.
nn 패키지에는 여러가지 다른 loss function이 있다. 가장 간단한 loss function 은 `nn.MSELoss`로 input과 target 사이의 평균제곱오차를 계산하는 것이다.

```python
# ex)
output = net(input)
target = Variable(torch.arange(1, 11))  # a dummy target, for example
criterion = nn.MSELoss()

loss = criterion(output, target)
print(loss)
```
```
Variable containing:
 38.7810
[torch.FloatTensor of size 1]
```

이제, 만약 역방향으로 loss를 따라간다면, `.grad_fn` attribute를 이용해 계산된 그래프를 볼 수 있다.
```
input -> conv2d -> relu -> maxpool2d -> conv2d -> relu -> maxpool2d
      -> view -> linear -> relu -> linear -> relu -> linear
      -> MSELoss
      -> loss
```

`loss.backward()`를 호출할 때, 전체 그래프는 loss와 구별된다. 그리고 그래프의 모든 변수는 gradient에 축적되는 `.grad`를 가지게된다.

```python
print(loss.grad_fn)  # MSELoss
print(loss.grad_fn.next_functions[0][0])  # Linear
print(loss.grad_fn.next_functions[0][0].next_functions[0][0])  # ReLU
```

```
<torch.autograd.function.MSELossBackward object at 0x2b5905cd95e8>
<torch.autograd.function.AddmmBackward object at 0x2b5905cd94f8>
<AccumulateGrad object at 0x2b58fed2f080>
```

### Backprop(역전파)

error(오차)를 역전파하기 위해서는 `loss.backward()`를 사용하면된다. 기존 gradients를 초기화해야하는데 다른 gradients는 기존의 gradients에 축적되어진다.

```python
net.zero_grad()     # zeroes the gradient buffers of all parameters

print('conv1.bias.grad before backward')
print(net.conv1.bias.grad)

loss.backward()

print('conv1.bias.grad after backward')
print(net.conv1.bias.grad)
```
```
conv1.bias.grad before backward
Variable containing:
 0
 0
 0
 0
 0
 0
[torch.FloatTensor of size 6]

conv1.bias.grad after backward
Variable containing:
 0.0005
 0.0213
 0.1481
-0.0372
 0.1560
 0.1718
[torch.FloatTensor of size 6]
```

[FULL Documents](http://pytorch.org/docs/master/nn.html)

### Update the weights

Stochastic Gradient Descent (SGD): `weight = weight - learning_rate * gradient`

```python
learning_rate = 0.01
for f in net.parameters():
    f.data.sub_(f.grad.data * learning_rate)
```

하지만 신경망을 사용할때 `SGD`, `Nesterov-SGD`, `Adam`, `RMSProp` 등 다양한 update rule을 사용하고 싶을 수 있다. 이를 위해서 `torch.optim`을 제공한다.

```python
import torch.optim as optim

# create your optimizer
optimizer = optim.SGD(net.parameters(), lr=0.01)

# in your training loop:
optimizer.zero_grad()   # zero the gradient buffers
output = net(input)
loss = criterion(output, target)
loss.backward()
optimizer.step()    # Does the update
```
---
layout: post
title: Autograd
tags: [pytorch, python]
---

### Autograd: automatic differentiation

`Pytorch`에서 neural networks의 중심은 `autograd` 패키지이다.
- `autograd`는 모든 연산(operations)에 있어서 자동구별화를 제공한다.
- `define-by-run` 프레임워크로 backprop(역전파)가 코드가 어떻게 실행되냐에 따라 정의되며, 모든 반복은 다를 수 있다.


#### Variable

`autograd.Variable`은 패키지의 주요 class이다. `autograd.Variable`는 Tensor를 감싸고, 정의된 모든 연산을 지원한다. 연산이 끝나면, `.backward()`를 호출하고 모든 gradients를 자동으로 계산한다. 해당 변수와 관련된 gradient가 있는 동안 `.data` attribute를 이용해 raw tensor에 접근할 수 있다. 이 변수는 `.grad`에 축적된다.

![](http://pytorch.org/tutorials/_images/Variable.png)

#### Function
`Function`또한 autograd에서 중요한 변수이다. `Variable`과 `Function`은 내부적으로 연결되어있고, 비주기그래프를 생성한다. 이 그래프는 계산의 완전한 기록을 인코딩한다. 각각 변수는 `.grad_fn` attribute를 가지고 있다. 이 attribute는 `Variable`을 생성하는 `Function`을 참조한다.

미분을 계산하고 싶으면 `.backward()`를 `Variable`에서 호출하면된다. `Variable`이 scalar라면 `.backword()`에 어떠한 arguments도 구체화할 필요 없다. 하지만 요소가 더 있다면 tensor 형태인 `grad_output`를 구체화해줘야한다.

#### 시작하기
```python
import torch
from torch.autograd import Variable
```

#### 변수생성
```python
x = Variable(torch.ones(2, 2), requires_grad=True)
print(x)
```
```
# 결과값
Variable containing:
 1  1
 1  1
[torch.FloatTensor of size 2x2]
```

#### 변수연산
```python
y = x + 2
print(y)
```
```
Variable containing:
 3  3
 3  3
[torch.FloatTensor of size 2x2]
```

y는 연산의 결과로 생성되었기 때문에 `grad_fn`를 가지고 있다.
```python
print(y.grad_fn)
```
```
Traceback (most recent call last):
  File "pytorch/variable.py", line 8, in <module>
    print(y.grad_fn)
  File "/Library/Frameworks/Python.framework/Versions/3.5/lib/python3.5/site-packages/torch/autograd/variable.py", line 86, in __getattr__
    raise AttributeError(name)
AttributeError: grad_fn
```
error가 발생하는 것을 확인했다. 지금은 `grad_fn`은 더이상 지원하지 않고, `Variable.creator`를 지원한다.
```python
print(y.creator)
```
```
<torch.autograd._functions.basic_ops.AddConstant object at 0x104366800>
```

```python
z = y * y * 3
out = z.mean()

print(z, out)
```
Variable containing:
 27  27
 27  27
[torch.FloatTensor of size 2x2]
 Variable containing:
 27
[torch.FloatTensor of size 1]
```

#### Gradients

역전파(backprop)인 `out.backward()`는 `out.backward(torch.Tensor([1.0]))`를 하는 것과 동일하다.

```python
out.backward()
print(x.grad)
```
```
Variable containing:
 4.5000  4.5000
 4.5000  4.5000
[torch.FloatTensor of size 2x2]
```

#### autograd 예제
```python
x = torch.randn(3)
x = Variable(x, requires_grad=True)

y = x * 2
while y.data.norm() < 1000:
    y = y * 2

print(y)
```
```
Variable containing:
-132.2659
 457.3291
-950.7308
[torch.FloatTensor of size 3]
```

```python
gradients = torch.FloatTensor([0.1, 1.0, 0.0001])
y.backward(gradients)

print(x.grad)
```
```
Variable containing:
  102.4000
 1024.0000
    0.1024
[torch.FloatTensor of size 3]
```
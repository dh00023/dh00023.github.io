---
title: "Tensor"
tag: pytorch
---

- [Pytorch tutorial](http://pytorch.org/tutorials/beginner/blitz/tensor_tutorial.html#tensors)


### Tensor

 Tensors are similar to numpy’s ndarrays, Tensors can also be used on a GPU to accelerate computing.
Tensor는 numpy의 ndarrays와 비슷하다. Tensor는 GPU를 사용해 계산속도를 높일 수 있다.

>  `numpy.ndarray`
```python
np.ndarray(shape=(2,2), dtype=float, order='F')
array([[ -1.13698227e+002,   4.25087011e-303],
       [  2.88528414e-306,   3.27025015e-309]])         #random
# order = 'F' (column-major), 'C'(row-major)
```
https://docs.scipy.org/doc/numpy/reference/generated/numpy.ndarray.html


```python
# Tensor 시작하기
from __future__ import print_function
import torch
```

```python
# 5X3 행렬 uninitialized
x = torch.Tensor(5, 3)
print(x)
# 5X3 행렬 initialized
x = torch.rand(5, 3)
print(x)
# 행렬크기를 구하기
print(x.size())
# => torch.Size([5, 3])
```

### Operations(연산)

```python
x = torch.rand(5, 3)
y = torch.rand(5, 3)

print(x + y)
print(torch.add(x, y))

result = torch.Tensor(5, 3)
torch.add(x, y, out=result)
print(result)

y.add_(x)
print(y)
```

[Tensor operations](http://pytorch.org/tutorials/beginner/blitz/tensor_tutorial.html#getting-started)(transposing, indexing, slicing, mathematical operations, linear algebra, random numbers)

### Numpy Bridge
torch의 Tensor를 numpy array로 전환해준다. torch Tensor와 numpy array는 메모리 위치를 공유한다.

#### Converting torch Tensor to numpy Array
```python
a = torch.ones(5)
b = a.numpy()
a.add_(1)
```

#### Converting numpy Array to torch Tensor
```python
import numpy as np
a = np.ones(5)
b = torch.from_numpy(a)
np.add(a, 1, out=a)
```

#### CUDA Tensors
CUDA ("Compute Unified Device Architecture", 쿠다)는 그래픽 처리 장치(GPU)에서 수행하는 (병렬 처리) 알고리즘을 C 프로그래밍 언어를 비롯한 산업 표준 언어를 사용하여 작성할 수 있도록 하는 GPGPU 기술이다.
```python
# let us run this cell only if CUDA is available
if torch.cuda.is_available():
    x = x.cuda()
    y = y.cuda()
    x + y
```
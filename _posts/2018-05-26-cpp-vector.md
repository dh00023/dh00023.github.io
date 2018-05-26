---
title: vector
categories:
- STL
tag:
- c/c++
---

vector 컨테이너는 **대표적인 시퀀스 컨테이너로 배열과 비슷**하여 사용이 쉬우며 자주 사용된다. vector는 임의 접근 반복자(Random Access Iterator)를 지원하는 **배열 기반 컨테이너**이다. vector의 가장 큰 특징 중 하나는 원소가 **하나의 메모리 블록에 연속하게 저장된다**는 것이다.그렇다 보니 원소가 추가되거나 삽입될 때 메모리 재할당이 발생할 수 있고 상당한 비용을 지불하게 된다. 그래서 메모리 할당 크기를 알 수 있게 capacity() 함수를 제공하며 한번에 메모리를 할당할 수 있는 reserve() 함수도 제공된다. 원소가 연속하게 저장되므로 [] 연산자 또는 at 으로 읽기에는 빠르지만  insert(), erase(), push_back() 등은 비효율적으로 동작한다.



### 생성자

| 생성자                                       | 설명 |
| -------------------------------------------- | ---- |
| vector v                                     |v는 빈 컨테이너        |
| vector v(n)                                  | v는 기본값으로 초기화된 n개의 원소를 갖는다. |
| vector v(n,x) | v는 x 값으로 초기화된 n개의 원소를 갖는다. |
| vector v(v2)                    | v는 v2 컨테이너의 복사본이다.(복사 생성자 호출) |
| vector v(a,b) | v는  반복자 구간 [b,e)로 초기화된 원소를 갖는다. |

### 멤버함수

| 함수                                                | 설명                                                         |
| --------------------------------------------------- | ------------------------------------------------------------ |
| v.assign(n,x)                                       | v에 x 값으로 n개의 원소를 할당한다.                          |
| v.assign(a,b)                                       | v를 반복자 구간 [a,b)로 할당                                 |
| v.at(i)                                             | v의 i번째 원소를 참조                                        |
| v.front()                                           | v의 첫 번째 원소를 참조한다.                                 |
| v.back()                                            | v의 마지막 원소를 참조                                       |
| v.capacity()                                        | v에 할당된 공간의 크기                                       |
| v.clear()                                           | v의 모든 원소를 제거                                         |
| v.empty()                                           | v가 비었는지 조사                                            |
| p=v.begin()                                         | p는 v의 첫 원소를 가리킨다.                                  |
| p=v.end()                                           | p는 v의 끝을 표식하는 반복자                                 |
| q=v.erase(p)                                        | p가 가리키는 원소를 제거한다. q는 다음 원소를 가리킨다.      |
| q=v.erase(b,e)                                      | 반복자 구간 [b,e)의 모든 원소를 제거한다. q는 다음 원소를 가리킨다. |
| q=v.insert(p,x)                                     | p가 가리키는 위치에 x 값을 삽입한다. q는 삽입한 원소를 가리키는 반복자다. |
| v.insert(p,n,x)                                     | p가 가리키는 위치에 n개의 x 값을 삽입한다.                   |
| v.insert(p,b,e)                                     | p가 가리키는 위치에 반복자 구간 [b,e)의 원소를 삽입한다.     |
| x=v.max_size()                                      | x는 v가 담을 수 있는 최대 원소의 개수(메모리의 크기)         |
| v.pop_back()                                        | v의 마지막 원소를 제거한다.                                  |
| v.push_back()                                       | v의 끝에 x를 추가한다.                                       |
| p=v.rbegin()                                        | p는 v의 역 순차열의 첫 원소를 가리키는 반복자다.             |
| p=v.rend()                                          | p는 v의 역 순차열의 끝을 표식하는 반복자                     |
| v.reserve(n)                                        | n개의 원소를 저장할 공간을 예약한다.                         |
| v.resize(n)                                         | v의 크기를 n으로 변경하고 확장되는 공간의 값을 기본값으로 초기화 한다. |
| v.resize(n,x)                                       | v의 크기를 n으로 변경하고 확장되는 공간의 값을 x 값으로 초기화한다. |
| v.size()                                            | v의 원소 갯수                                                |
| v.swap(v2)                                          | v와 v2를 swap한다.                                           |

### 예제

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main(){

    vector<int> v;
    vector<int> v2(5);    // 0으로 초기화된 5개의 원소
    // 메모리크기 8만큼 할당
    v.reserve(8);

    v.push_back(10);
    v.push_back(20);
    v.push_back(30);
    v.push_back(40);
    v.push_back(50);

    for (vector<int>::size_type i = 0; i < v.size(); ++i)
        cout << v[i] << endl;
    cout << endl;

    // 벡터의 원소 갯수를 10개로 확장, 추가된 공간은 디폴트 0으로 채워진다.
    v.resize(10);                
}
```

여기서 v.size()만큼 for문을 실행하려면 v.size()의 타입과 i 가 같아야 한다. 

- `vector<int>::size_type` : 벡터의 size 의 반환 타입을 가져온다 (unsigned int)

 

### 참조페이지

출처 : http://hyeonstorage.tistory.com/324



#### 
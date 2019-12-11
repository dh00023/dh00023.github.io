---
title: STL Container - Container Adapter
tag:
- c/c++
---

간결함과 명료성을 위해 인터페이스를 제한한 시퀀스나 연관 컨테이너의 변형이다.
반복자를 지원하지 않으므로 STL 알고리즘에서는 사용할 수 없다.

### stack

```cpp
#include <iostream>
#include <stack>

using namespace std;

int main(void){
    stack<int> s;
    s.push(7);
    s.push(3);
    s.push(9);

    while(!s.empty()){
        cout << s.top() << ' ';
        s.pop();
    }
}
```

### queue

- 추가 : push()
- 삭제 : pop()
- 조회 : front() / back()
- 검사 : empty() / size()

```cpp
#include <iostream>
#include <queue>

using namespace std;

int main(void){
    
    queue<int> q;
    q.push(1);
    q.push(2);
    q.push(3);

    while(!q.empty()){
        cout << q.front() << ' ';
        q.pop();
    }
}
```

### priority queue

```cpp
#include <iostream>
#include <queue>

using namespace std;

int main(void){

    int n, x;
    cin >> n;

    priority_queue<int> pq;
    for(int i=0; i<n;i++){
        cin >> x;
        pq.push(x);
    }

    while(!pq.empty()){
        cout << pq.top() << ' ';
        pq.pop();
    }
}
```
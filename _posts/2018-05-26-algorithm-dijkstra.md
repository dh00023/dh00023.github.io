---
title: 다익스트라 알고리즘(Dijkstra's Algorithm)
categories:
- algorithm/ds
tag:
- c/c++
---

다익스트라 알고리즘은 시작점으로 부터 나머지 정점까지 최단거리를 구할 때 사용한다. 주로 네트워크 경로 설계에 많이 적용된다.

- (ex) OSPF(Open Shortest Path First)라는 IP 망에서의 라우팅 프로토콜.

간선에 가중치가 있는 그래프에서 1:N 최단거리를 구하는 알고리즘이다. 다익스트라 알고리즘은 너비우선탐색(BFS)을 기본으로 한다. 이때 **가중치는 음수값을 갖지 않는다.**



## 구현방법

다익스트라 알고리즘은 각각의 정점에 대해서 정점 s에서 정점 v까지의 최단 거리를 d[v]에 저장하면서 탐색한다. 알고리즘 시작 시에 d[s] = 0이고, s가 아닌 다른 모든 정점에 대해서는 d[v] = ∞ 로 놓아 다른 정점에 대해서는 아직 최단 경로를 모른다는 것을 표시한다. 알고리즘이 종료되었을 때 d[v]는 s에서 v까지의 최단 경로의 거리를 나타내게 되고, 만약 경로가 존재하지 않으면 거리는 여전히 무한대로 남는다.

![ë¤ìµì¤í¸ë¼ ìê³ ë¦¬ì¦ì ëí ì´ë¯¸ì§ ê²ìê²°ê³¼](https://lh3.googleusercontent.com/nriXR42Kaq1uwLkXLqiAi5r8naWSA2Ur2Wp_za6DA7boHk0m6f79zovVJsNVL4qua3_Q2jpnExWtJiSSACClvmxbDRCn0vkPbl2nEMsX0kB_bf1kQ3iX)

다익스트라 알고리즘은 다음과 같다.` G[A][B]`는 A와 B 사이의 거리라고 하자.

1. 출발점으로부터의 최단거리를 저장할 배열 d[v]를 만들고, 출발 노드에는 0을, 출발점을 제외한 다른 노드들에는 매우 큰 값 INF(자료형의 최대값)를 채워 넣는다.
2. 현재 노드 A에 출발 노드를 저장한다.
3. A로부터 갈 수 있는 임의의 노드 B에 대해, `d[A]+G[A][B]`와 d[B]의 값을 비교한다.
4. 만약 `d[A]+G[A][B]`가 더 짧다면, d[B]의 값을 이 값으로 갱신시킨다.
5. 모든 이웃 노드 B에 대해 이 작업을 수행한다.
6. A의 상태를 "방문 완료"로 바꾼다. 그러면 이제 더 이상 A는 사용하지 않는다.
7. "미방문" 상태인 모든 노드들 중, 출발점으로부터의 거리가 제일 짧은 짧은 노드 하나를 골라서 그 노드를 A에 저장한다.
	- 모든 노드를 순회해 거리가 제일 짧은 노드를 찾아야한다. 이때 우선순위큐를 활용하면 이 비용을 줄일 수 있다.
8. 도착 노드가 "방문 완료" 상태가 되거나, 혹은 더 이상 미방문 상태의 노드를 선택할 수 없을 때까지, 3~7의 과정을 반복한다.

도착 노드에 저장된 값이 A로부터의 최단거리이다. 만약 도착노드의 값이 INF라면, 중간에 경로가 끊긴 것을 의미한다.

## 구현

### 인접행렬

```c
#include <stdio.h>
#include <limits.h> //INT_MAX
#define V 9
typedef enum { false, true } bool;

// 최단 거리를 찾는 함수
int min_distance(int dist[], bool visited[])
{
    // 자료형의 최대값으로 min값 설정
    int min = INT_MAX, min_index=0;

    for (int v = 0; v < V; v++){
        //visited[v]가 false이면서 dist[v]가 최소값보다 적은 경우
        if (visited[v] == false && dist[v] <= min){
            min = dist[v];
            min_index = v;
        }

    }
    return min_index;
}

void print_path(int parent[], int j){
    if (parent[j] == - 1)
        return;
    
    print_path(parent, parent[j]);
    
    printf("%d ", j);
}

// dist 배열을 출력해주는 함수
void print_distance(int dist[], int n, int parent[])
{
    int src =0;
    printf("Vertex\t시작 정점으로부터 거리\t경로\n");
    for (int i = 0; i < V; i++){
        printf("\n%d -> %d \t\t %d\t\t%d ",
               src, i, dist[i], src);
        print_path(parent, i);
    }
    
    
}

// 다익스트라 함수
void dijkstra(int graph[V][V], int src)
{
    // dist배열은 최단거리를 저장하는 배열이다. 즉, 결과값
    // src에서 정점 i까지 가는 거리
    int dist[V];
    bool visited[V]; //방문했는지 기록하는 배열
    int parent[V]; // 부모 노드를 기억할 배열

    // dist배열은 자료형의 최대값, visited배열은 false 로 초기화
    parent[0] = -1;
    for (int i = 0; i < V; i++){
        dist[i] = INT_MAX;
        visited[i] = false;
    }

    // 시작점(src)에서 자기자신의 거리는 0이다.
    dist[src] = 0;

    // 모든 정점에서 최단거리 찾기
    for (int count = 0; count < V-1; count++){

        int u = min_distance(dist, visited); // 최소거리의 정점을 저장한다. u는 count가 0일때 src와 같다.

        visited[u] = true; //방문했음을 표시한다.

        // 값을 update해준다.
        for (int v = 0; v < V; v++){
            // visited이 false이면서, u->v로의 엣지가 있고
            // 엣지를 더한 값이 현재 distance보다 작은 경우
            if (!visited[v] && graph[u][v] && dist[u] != INT_MAX
                && dist[u]+graph[u][v] < dist[v]){
                parent[v] = u;
                dist[v] = dist[u] + graph[u][v];
            }
        }


    }
    // 시작점으로 부터 각 정점의 최단거리 출력
    print_distance(dist, V,parent);
}

int main()
{
    
    int graph[V][V] = {
        {0, 4, 0, 0, 0, 0, 0, 8, 0},
        {5, 0, 8, 0, 0, 0, 0, 11, 0},
        {0, 8, 0, 7, 0, 4, 0, 0, 2},
        {0, 0, 7, 0, 9, 14, 0, 0, 0},
        {0, 0, 0, 9, 0, 10, 0, 0, 0},
        {0, 0, 4, 14, 10, 0, 2, 0, 0},
        {0, 0, 0, 0, 0, 2, 0, 1, 6},
        {0, 12, 0, 0, 0, 0, 1, 0, 7},
        {0, 0, 2, 0, 0, 0, 6, 7, 0}
    };

    dijkstra(graph, 0);

    return 0;
}
```



## 단점

![](https://i.imgur.com/71eZ0Fo.png)



- 가중치가 음수인 경우 처리할 수 없다.

![](http://chunchu.yonsei.ac.kr/news/photo/201603/21321_9235_5528.JPG)

- 출발점으로부터 거리가 가까우면서 동시에 도착점의 방향과 무관한 점들의 최단 경로를 먼저 찾는 헛수고를 하는 경우가 있다. 출발점이 a이고 도착점이 d일 때, 다익스트라 알고리즘은 도착점과 무관한 방향에 있는 점 e, f, g, h, i 모두의 최단 경로를 찾은 후에서야 도착점 d를 향해 최단 경로 찾기를 수행한다. 
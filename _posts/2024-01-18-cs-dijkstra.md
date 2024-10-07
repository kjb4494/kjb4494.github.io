---
title: 다익스트라(Dijkstra) 알고리즘
author: kjb4494
date: 2024-01-18 20:40:00 +0900
categories: [코딩테스트, 백준]
tags: [코딩테스트, 다익스트라]
---

## 다익스트라 알고리즘이란?

다익스트라(Dijkstra) 알고리즘은 네덜란드의 컴퓨터 과학자 에츠허르 비베르트 디익스트라(Edsger Wybe Dijkstra)가 고안한 알고리즘으로, 그래프에서 두 노드 사이의 최단 경로를 찾는 데 사용됩니다. **가중치가 있는 그래프**에서 작동하며, **가중치는 양수**여야 합니다.

### 기본 동작 원리

1. **초기화**: 시작 노드를 설정하고, 시작 노드에서 각 노드까지의 거리를 무한대로 설정합니다. 시작 노드의 거리는 0으로 설정합니다.
1. **노드 방문**: 아직 방문하지 않은 모든 노드 중에서, 시작 노드로부터 가장 가까운 노드를 선택합니다.
1. **최단 경로 갱신**: 선택된 노드를 통과하여 인접한 노드로 갈 때의 거리를 계산하고, 이 거리가 현재 알려진 최단 경로보다 짧다면, 그 노드까지의 거리를 갱신합니다.
1. **반복**: 모든 노드가 방문될 때까지 2번과 3번 단계를 반복합니다.
1. **결과**: 시작 노드로부터 모든 노드까지의 최단 경로가 결정됩니다.

## 다익스트라 알고리즘 예제와 해석

### [_1. \[백준 1916번\] 최소비용 구하기_](https://www.acmicpc.net/problem/1916)

```python
import heapq
import sys

input = sys.stdin.readline


def dijkstra():
    costs = {node: float('inf') for node in GRAPH}
    costs[START_NODE_INDEX] = 0

    # cost, node_index
    queue = [(0, START_NODE_INDEX)]

    while queue:
        current_cost, current_node_index = heapq.heappop(queue)

        if costs[current_node_index] < current_cost:
            continue

        for next_node_index, cost_to_next_node in GRAPH[current_node_index]:
            cost = current_cost + cost_to_next_node
            if cost < costs[next_node_index]:
                costs[next_node_index] = cost
                heapq.heappush(queue, (cost, next_node_index))

    return costs


if __name__ == '__main__':
    N = int(input())
    M = int(input())
    GRAPH = {i: set() for i in range(1, N + 1)}
    for _ in range(M):
        # 출발 도시 번호, 도착지 도시 번호, 버스 비용
        u, v, w = list(map(int, input().split()))
        GRAPH[u].add((v, w))
    START_NODE_INDEX, DESTINATION_NODE_INDEX = list(map(int, input().split()))
    COSTS = dijkstra()
    print(COSTS[DESTINATION_NODE_INDEX])

```

### [_2. \[백준 1753번\] 최단경로_](https://www.acmicpc.net/problem/1753)

```python
import heapq
import sys

input = sys.stdin.readline


def dijkstra():
    distances = {node: float('inf') for node in GRAPH}
    distances[START_NODE_INDEX] = 0
    queue = [(0, START_NODE_INDEX)]

    while queue:
        current_distance, current_node_index = heapq.heappop(queue)

        if distances[current_node_index] < current_distance:
            continue

        for next_node_index, distance_to_next_node in GRAPH[current_node_index]:
            distance = current_distance + distance_to_next_node
            if distance < distances[next_node_index]:
                distances[next_node_index] = distance
                heapq.heappush(queue, (distance, next_node_index))

    return distances


if __name__ == '__main__':
    V, E = list(map(int, input().split()))
    START_NODE_INDEX = int(input())
    GRAPH = {i: set() for i in range(1, V + 1)}
    for _ in range(E):
        u, v, w = list(map(int, input().split()))
        GRAPH[u].add((v, w))

    DISTANCES = dijkstra()
    for i in range(1, V + 1):
        print(DISTANCES[i] if DISTANCES[i] != float('inf') else "INF")

```

### 문제 해석

두 문제와 해답이 거의 완전하게 동일합니다. 두 문제 모두 **그래프 문제**이며 각 노드의 연결에 **양수의 가중치**가 있으며 **최소값을 구하는 문제**입니다.

다익스트라 알고리즘 코드는 **큐(Queue)를 이용**하여 그래프 문제에서 활용되기 때문에 BFS(너비 우선 탐색)와 유사한 코드를 가졌습니다. 하지만 BFS와는 몇 가지 차이점이 있습니다.

1. **사용하는 큐의 종류**: BFS에서는 FIFO(First-In-First-Out) 큐를 사용하여 노드를 탐색합니다. 즉, 먼저 들어온 노드가 먼저 처리됩니다. 반면, 다익스트라 알고리즘에서는 **우선순위 큐**를 사용하여 가장 낮은 비용을 가진 노드를 먼저 처리합니다.
1. **비용 갱신**: BFS는 각 노드를 단 한 번씩만 방문하며, 그래프의 최단 경로를 노드의 개수로 측정합니다. 반면, 다익스트라 알고리즘은 가중치가 있는 그래프에서 작동하며, 노드를 방문할 때마다 해당 노드까지의 비용을 갱신하고, 이 비용을 우선순위 큐에 넣어 계속해서 **최소 비용을 가진 노드를 찾아**나갑니다.
1. **적용 가능한 그래프**: BFS는 주로 비가중치 그래프에서 사용되며, 다익스트라는 **양수의 가중치가 있는 그래프에서 사용**됩니다.

### 왜 우선순위 큐(heapq)를 사용할까?

우선순위 큐는 각 요소가 우선순위를 가지고 있고, 이 우선순위에 따라 요소가 처리됩니다.

만약 다익스트라 알고리즘에서 일반 큐를 사용할 경우, 큐에서 노드를 하나씩 꺼내 그래프를 탐색할 때, 꺼낸 노드가 최적의 선택이 아닐 수 있으며, 결과적으로 불필요한 노드 탐색이 발생할 수 있습니다. 반면, 우선순위 큐를 사용하면 항상 현재까지의 최소 비용을 가진 노드를 먼저 탐색하게 되어, 더 효율적으로 최단 경로를 찾을 수 있습니다.

일반 큐를 사용하면, 각 노드를 처리하는데 O(N)의 시간이 소요됩니다. 여기서 N은 노드의 수입니다. 그러나 우선순위 큐를 사용하면, 각 노드를 처리하는데 O(log N)의 시간이 소요됩니다. 따라서 전체적인 알고리즘의 시간 복잡도가 크게 감소합니다.

또한, 일반 큐를 사용할 경우, 동일한 노드가 여러 번 큐에 추가될 수 있으며, 이는 불필요한 계산을 유발합니다. 우선순위 큐를 사용하면, 더 낮은 비용으로 동일한 노드에 도달하는 경우에만 큐에 다시 추가되므로, 중복 처리를 줄일 수 있습니다.

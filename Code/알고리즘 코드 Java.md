

코테에서 자주 쓰이는 알고리즘 기본형 With GPT

## DFS

```java
import java.util.*;

public class DFS {
    public static void dfs(int v, boolean[] visited, List<List<Integer>> graph) {
        visited[v] = true;
        System.out.print(v + " ");
        for (int i : graph.get(v)) {
            if (!visited[i]) {
                dfs(i, visited, graph);
            }
        }
    }

    public static void main(String[] args) {
        List<List<Integer>> graph = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            graph.add(new ArrayList<>());
        }
        graph.get(0).add(1);
        graph.get(0).add(2);
        graph.get(1).add(0);
        graph.get(1).add(3);
        graph.get(2).add(0);
        graph.get(2).add(4);
        graph.get(3).add(1);
        graph.get(4).add(2);

        boolean[] visited = new boolean[5];
        dfs(0, visited, graph);
    }
}

```

## BFS

```java
import java.util.*;

public class BFS {
    public static void bfs(int start, List<List<Integer>> graph) {
        boolean[] visited = new boolean[graph.size()];
        Queue<Integer> queue = new LinkedList<>();
        queue.add(start);
        visited[start] = true;

        while (!queue.isEmpty()) {
            int v = queue.poll();
            System.out.print(v + " ");
            for (int i : graph.get(v)) {
                if (!visited[i]) {
                    visited[i] = true;
                    queue.add(i);
                }
            }
        }
    }

    public static void main(String[] args) {
        List<List<Integer>> graph = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            graph.add(new ArrayList<>());
        }
        graph.get(0).add(1);
        graph.get(0).add(2);
        graph.get(1).add(0);
        graph.get(1).add(3);
        graph.get(2).add(0);
        graph.get(2).add(4);
        graph.get(3).add(1);
        graph.get(4).add(2);

        bfs(0, graph);
    }
}

```

## 다익스트라

```java 
import java.util.*;

public class Dijkstra {
    static class Node implements Comparable<Node> {
        int vertex, weight;
        Node(int v, int w) {
            this.vertex = v;
            this.weight = w;
        }

        public int compareTo(Node other) {
            return this.weight - other.weight;
        }
    }

    public static void dijkstra(int start, List<List<Node>> graph) {
        int n = graph.size();
        int[] dist = new int[n];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[start] = 0;

        PriorityQueue<Node> pq = new PriorityQueue<>();
        pq.add(new Node(start, 0));

        while (!pq.isEmpty()) {
            Node node = pq.poll();
            int v = node.vertex;
            int w = node.weight;

            for (Node neighbor : graph.get(v)) {
                int newDist = w + neighbor.weight;
                if (newDist < dist[neighbor.vertex]) {
                    dist[neighbor.vertex] = newDist;
                    pq.add(new Node(neighbor.vertex, newDist));
                }
            }
        }

        System.out.println(Arrays.toString(dist));
    }

    public static void main(String[] args) {
        List<List<Node>> graph = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            graph.add(new ArrayList<>());
        }
        graph.get(0).add(new Node(1, 10));
        graph.get(0).add(new Node(2, 3));
        graph.get(1).add(new Node(2, 1));
        graph.get(1).add(new Node(3, 2));
        graph.get(2).add(new Node(3, 8));
        graph.get(3).add(new Node(4, 7));

        dijkstra(0, graph);
    }
}

```

 ## 플로이드워셜
 
```java
import java.util.*;

public class FloydWarshall {
    public static void floydWarshall(int[][] graph) {
        int n = graph.length;
        int[][] dist = new int[n][n];

        for (int i = 0; i < n; i++) {
            System.arraycopy(graph[i], 0, dist[i], 0, n);
        }

        for (int k = 0; k < n; k++) {
            for (int i = 0; i < n; i++) {
                for (int j = 0; j < n; j++) {
                    if (dist[i][k] + dist[k][j] < dist[i][j]) {
                        dist[i][j] = dist[i][k] + dist[k][j];
                    }
                }
            }
        }

        for (int[] row : dist) {
            System.out.println(Arrays.toString(row));
        }
    }

    public static void main(String[] args) {
        int INF = 99999;
        int[][] graph = {
            {0, 3, INF, 5},
            {2, 0, INF, 4},
            {INF, 1, 0, INF},
            {INF, INF, 2, 0}
        };

        floydWarshall(graph);
    }
}

```

벨만포드
```java
import java.util.*;

public class BellmanFord {
    public static void bellmanFord(int[][] edges, int V, int start) {
        int[] dist = new int[V];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[start] = 0;

        for (int i = 0; i < V - 1; i++) {
            for (int[] edge : edges) {
                int u = edge[0], v = edge[1], w = edge[2];
                if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                }
            }
        }

        for (int[] edge : edges) {
            int u = edge[0], v = edge[1], w = edge[2];
            if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) {
                System.out.println("Negative weight cycle detected");
                return;
            }
        }

        System.out.println(Arrays.toString(dist));
    }

    public static void main(String[] args) {
        int V = 5;
        int[][] edges = {
            {0, 1, -1}, {0, 2, 4},
            {1, 2, 3}, {1, 3, 2}, {1, 4, 2},
            {3, 2, 5}, {3, 1, 1}, {4, 3, -3}
        };

        bellmanFord(edges, V, 0);
    }
}

```
#include <iostream>
#include <vector>
#include <queue>
#include <omp.h>
using namespace std;

class Graph {
    int V;
    vector<vector<int>> adj;

public:
    Graph(int V) : V(V), adj(V) {}

    void addEdge(int u, int v) {
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    // 🔵 Parallel BFS
    void parallelBFS(int start) {
        vector<bool> visited(V, false);
        queue<int> q;

        visited[start] = true;
        q.push(start);

        cout << "\nBFS: ";

        while (!q.empty()) {
            int size = q.size();

            #pragma omp parallel for
            for (int i = 0; i < size; i++) {
                int node = -1;

                // Safe queue access
                #pragma omp critical
                {
                    if (!q.empty()) {
                        node = q.front();
                        q.pop();
                        cout << node << " ";
                    }
                }

                if (node == -1) continue;

                for (int j = 0; j < adj[node].size(); j++) {
                    int nbr = adj[node][j];

                    if (!visited[nbr]) {
                        #pragma omp critical
                        {
                            if (!visited[nbr]) {
                                visited[nbr] = true;
                                q.push(nbr);
                            }
                        }
                    }
                }
            }
        }
        cout << endl;
    }

    // 🔴 DFS Utility
    void dfsUtil(int node, vector<bool> &visited) {
        bool alreadyVisited;

        #pragma omp critical
        {
            alreadyVisited = visited[node];
            if (!visited[node]) {
                visited[node] = true;
                cout << node << " ";
            }
        }

        if (alreadyVisited) return;

        #pragma omp parallel for
        for (int i = 0; i < adj[node].size(); i++) {
            int nbr = adj[node][i];

            if (!visited[nbr]) {
                #pragma omp task
                dfsUtil(nbr, visited);
            }
        }
    }

    // 🔴 Parallel DFS
    void parallelDFS(int start) {
        vector<bool> visited(V, false);

        cout << "\nDFS: ";

        #pragma omp parallel
        {
            #pragma omp single
            {
                dfsUtil(start, visited);
            }
        }

        cout << endl;
    }
};

int main() {
    int V, E, u, v, start;

    cout << "Enter number of vertices: ";
    cin >> V;

    Graph g(V);

    cout << "Enter number of edges: ";
    cin >> E;

    cout << "Enter edges (u v):\n";
    for (int i = 0; i < E; i++) {
        cin >> u >> v;
        g.addEdge(u, v);
    }

    cout << "Enter starting vertex: ";
    cin >> start;

    g.parallelBFS(start);
    g.parallelDFS(start);

    return 0;
}
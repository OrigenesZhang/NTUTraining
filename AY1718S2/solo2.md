# Solo Training 2 Report

Author: Die_Welle
Time: 07 Mar 2018

This report is for the second solo training of AY17/18 Sem 2.

***

### Overview

The contest consistes of two questions and the duration is 1 hour. Both the question are related to search/bruteforce.

### Problem Analysis

##### Problem A Little Queens
&nbsp;
This problem is very similar to the traditional eight queens problem. However, the problem tests our ability to optimize the code by setting a very tight time limit (0.75s).
The most important optimization is we need to judge whether the current position is confilict with the former columns and two diagonals since we place the queens row by row and they won't be in the same row.
We define bool vis[3][N]. Use vis[0][i] to judge whether the current position i will conflict in the column. vis[1][i] and vis[2][i] decide whether the current position will confilict in the two diagonals.

**Code:**
```cpp
#include<iostream>
using namespace std;

const int N = 10 + 5;

int n, k, ans, pos[N];
bool vis[5][N * 2];

void solve(int cur, int lef) {
    if(!lef) { ans++; return; }
    if(cur >= n) return;
    solve(cur + 1, lef);
    for(int i = 0; i < n; i++)
        if(!vis[0][i] && !vis[1][i + cur] && !vis[2][cur - i + n]) {
            //pos[i] = cur;
            vis[0][i] = vis[1][i + cur] = vis[2][cur - i + n] = true;
            solve(cur + 1, lef - 1);
            vis[0][i] = vis[1][i + cur] = vis[2][cur - i + n] = false;
        }
}

int main() {
    cin >> n >> k;
    if(k > n) { cout << 0 << '\n'; return 0; }
    solve(0, k);
    cout << ans << '\n';
    return 0;
}
```

##### Problem B Divide and conquer
&nbsp;
This problem is much harder than the first problem. Since the problem asks us to divide all points into two parts at first, it's easy for us to come up with an algorithm to try to find a subset in all points. However, with time limit only 0.25s, it will definitely result in time limit exceeded.
The right solution is to use bruteforce to enumerate all transformation possibilities, both rotation and translation. Then we will add an edge if after transformation, one vertex goes to another vertex's position. Since every vertex can at most have one "father" and one "son", there will be several chains and circuits in our graph.
Next we prove that the transformation is valid if and only if all chains/circuits consist of even vertices. Suppose all chains/circuits consist of even vertices, then we can simply choose odd indexed vertices to move and others keep still, which is obvious a valid movement. When there exists a chain/circuit consists of odd vertices, then obvious there will always be one vertex left and the movement cannot be valid.
So after we build the graph, we just have to check whether all chains/circuits are made of even nodes.

**Code:**
```cpp
#include<iostream>
#include<cstdio>
#include<cstring>
using namespace std;

struct Node {
    int r, c;
};

const int N = 20 + 5;
char s[N][N];
int n, a[N][N], sz, fr[N * N], to[N * N], ans[N][N], mp[N][N];
Node node[N * N];
bool vis[N * N];

void rotate(int& r, int& c) {
    int t = r;
    r = n - 1 - c;
    c = t;
}

bool inside(int r, int c) { return r >= 0 && r < n && c >= 0 && c < n; }

int main() {
    scanf("%d", &n);
    for(int i = 0; i < n; i++) scanf("%s", s[i]);
    for(int i = 0; i < n; i++)
        for(int j = 0; j < n; j++) {
            a[i][j] = s[i][j] - '0';
            if(a[i][j]) { mp[i][j] = sz; node[sz].r = i; node[sz++].c = j; }
        }
    if(sz % 2 == 1) { printf("NO\n"); return 0; }

    for(int ro = 0; ro < 4; ro++)  // rotation
        for(int addr = -n + 1; addr < n; addr++)  // move x-axis
            for(int addc = -n + 1; addc < n; addc++) {  // move y-axis
                memset(fr, -1, sizeof(fr));
                memset(to, -1, sizeof(to));
                memset(ans ,0, sizeof(ans));
                memset(vis, false, sizeof(vis));

                for(int i = 0; i < sz; i++) {
                    int r = node[i].r, c = node[i].c;
                    for(int j = 0; j <= ro; j++) rotate(r, c);
                    //cout << r << ' ' << c << endl;
                    r += addr; c += addc;
                    if(r == node[i].r && c == node[i].c) goto label;
                    if(inside(r, c) && a[r][c]) {
                        to[i] = mp[r][c];
                        fr[mp[r][c]] = i;
                    }
                }

                for(int i = 0; i < sz; i++)
                    if(fr[i] == -1) {
                        int j = i, cnt = 0;
                        while(j != -1) {
                            vis[j] = true;
                            if(!(cnt & 1)) ans[node[j].r][node[j].c] = 1;
                            j = to[j];
                            cnt++;
                        }
                        if(cnt & 1) goto label;
                    }

                for(int i = 0; i < sz; i++)
                    if(!vis[i]) {
                        int j = i, cnt = 0;
                        while(!vis[j]) {
                            vis[j] = true;
                            if(!(cnt & 1)) ans[node[j].r][node[j].c] = 1;
                            j = to[j];
                            cnt++;
                        }
                        if(cnt & 1) goto label;
                    }

                printf("YES\n");
                for(int i = 0; i < n; i++) {
                    for(int j = 0; j < n; j++) printf("%d", ans[i][j]);
                    printf("\n");
                }
                return 0;
                label:;
            }
    cout << "NO\n";
    return 0;
}
```
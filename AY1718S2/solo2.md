# Solo Training 2 Report

Author: Die_Welle
Time: 07 Mar 2018

This report is for the second solo training of AY17/18 Sem 2.

***

### Overview

The contest consistes of two questions and the duration is 1 hour. Both the question are related to search/bruteforce.

### Problem Analysis

##### Problem A Little Queens
&nbsp
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
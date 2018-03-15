# Training Report

Author: Origenes  
Team: NTU_Origenes  
Time: 12 Mar 2018  

The report is for the fourth solo traning session of AY17/18 sem 2.  
The topic of the training is dynamic programming.  

***

### Problem Analysis

##### Problem A Space Elevator
&nbsp;
Since a height restriction is mentioned in the problem, it is natural to greedily consider the ones with tighter restrictions first. Hence the bricks would be sorted in terms of height restriction.  
After the conversion, the problem falls into the multi-knapsack type.  
Time complexity: O(K\*max(a)\*max(c))  

**Code:**
```cpp
#include <iostream>
#include <algorithm>
using namespace std;
typedef long long ll;
const int maxk=412;
const int maxa=41234;
struct Block{
	int h, c, a;
	bool operator < (const Block& rhs) const{
		return a<rhs.a;
	}
}blks[maxk];
int K;
bool dp[maxk][maxa];
int main(){
	cin>>K;
	for(int i=0; i<K; i++)
		cin>>blks[i].h>>blks[i].a>>blks[i].c;
	sort(blks, blks+K);
	for(int i=0; i<=blks[0].a; i+=blks[0].h)
		dp[0][i]=true;
	// for all the previous stages, if any one is true, the result is true
	for(int i=1; i<K; i++)
		for(int j=0; j<=blks[i].a; j++)
			for(int k=0; k<=min(blks[i].c, j/blks[i].h); k++)
				dp[i][j]|=dp[i-1][j-k*blks[i].h];
	for(int i=blks[K-1].a; i>=0; i--)
		if(dp[K-1][i]){
			cout<<i<<endl;
			return 0;
		}
	cout<<"-1"<<endl;
	return 0;
}
```

##### Problem B Margaritas on the River Walk
&nbsp;
The problem statement indicates that the selecting process shall not be stopped until no more can be selected.  
If the cost of items are sorted in increasing order, then the selection can be done using linear scanning.  
The ith loop denotes the items indexing from 0 to i-1 are chosen, item i is not chosen and subloops start from item i+1. In this sequence, all possible states would be looped only once.  
Time complexity: O(V^2\*D)  

**Code:**
```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;
typedef unsigned long long ull;
const int maxd=1234;
const int maxv=35;
ull dp[maxd], cost[maxv], s, ans;
int N, V, D;
int main(){
	cin>>N;
	for(int cs=1; cs<=N; cs++){
		cin>>V>>D;
		for(int i=0; i<V; i++)
			cin>>cost[i];
		sort(cost, cost+V);
		s=ans=0;
		for(int i=0; i<V; i++){
			memset(dp, 0, sizeof(dp));
			dp[s]=1;
			// items indexing from 0 to i-1 are chosen and item i is not chosen
			for(int j=i+1; j<V; j++)
				for(int k=D; k>=cost[j]+s; k--)
					dp[k]+=dp[k-cost[j]];
			for(int j=D; j>=D-cost[i]+1; j--)
				if(j>=s) ans+=dp[j];
			// Update the prefix sum
			s+=cost[i];
		}
		cout<<cs<<' '<<ans<<endl;
	}
	return 0;
}
```

##### Problem C Cash Machine
&nbsp;
The problem is a typical multi 0-1 knapsack problem but some overwhelming TLE verdicts suggests that binary optimisation is needed here.  
Binary optimisation is to rewrite the weights of the items into the form of summation of the powers of two. It is a constant level optimisation.  
However, I got the my solution passed without any optimisation...  
Time Complexity: O(n\*cash)  

**Code:**
```cpp
#include <iostream>
#include <cstring>
using namespace std;
const int maxc=123456;
const int maxn=15;
int cash, N, n[maxn], d[maxn];
bool dp[maxc];
int main(){
	while(cin>>cash>>N){
		for(int i=0; i<N; i++)
			cin>>n[i]>>d[i];
		memset(dp, false, sizeof(dp));
		dp[0]=true;
		for(int i=0; i<N; i++)
			for(int j=cash; j>=0; j--)
				if(dp[j])
					for(int k=1; k<=n[i]&&j+k*d[i]<=cash; k++)
						dp[j+k*d[i]]=true;
		for(int i=cash; i>=0; i--)
			if(dp[i]){
				cout<<i<<endl;
				break;
			}
	}
	return 0;
}
```

##### Problem D Dollar Dayz
&nbsp;
It is a basic knapsack problem but the result may not fit into unsigned long long type in C++. High precision is needed here (or you can choose to use just two unsigned long long to do this simulation because the number is not insanely large).  
I chose Java for this problem because of the BigInteger class (also that python is not supported by poj, sigh).  
Time complexity: O(K\*N\*log(N))  

**Code:**
```java
import java.io.OutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.PrintWriter;
import java.util.Scanner;
import java.math.BigInteger;

/**
 * Built using CHelper plug-in
 * Actual solution is at the top
 *
 * @author Origenes
 */
public class Main {
    public static void main(String[] args) {
        InputStream inputStream = System.in;
        OutputStream outputStream = System.out;
        Scanner in = new Scanner(inputStream);
        PrintWriter out = new PrintWriter(outputStream);
        D solver = new D();
        solver.solve(1, in, out);
        out.close();
    }

    static class D {
        public void solve(int testNumber, Scanner in, PrintWriter out) {
            int N, K;
            N = in.nextInt();
            K = in.nextInt();
            final int maxn = 1234;
            final int maxk = 123;
            BigInteger[][] dp = new BigInteger[maxk][maxn];
            for (int i = 0; i <= K; i++)
                for (int j = N; j >= 0; j--)
                    dp[i][j] = new BigInteger("0");
            dp[0][0] = new BigInteger("1");

            for (int i = 1; i <= K; i++) {
                for (int k = 0; k <= N; k += i) {
                    for (int j = N; j >= k; j--) {
                        dp[i][j] = dp[i][j].add(dp[i - 1][j - k]);
                    }
                }
            }
            out.println(dp[K][N].toString());
        }
    }
}
```

##### Problem E Brackets Sequence
&nbsp;
This is an interval dp problem (the only problem does not belong to knapsack category).  
Let dp[L]\[R] denotes the number of characters need to be added to make s[L...R] regular and path[L]\[R] denotes the position of the outermost additional bracket.  We can dfs the string and recursively print the result.  
Time complexity: O(len(S)^3)  

**Code:**
```cpp
#include <cstdio>
#include <cstring>
#include <iostream>
using namespace std;
const int maxn=123;
const int inf=0x3f3f3f3f;
const char match[]={'(', ')', '[', ']'};
char s[maxn];
int dp[maxn][maxn], path[maxn][maxn];
int dfs(int L, int R){
	if(dp[L][R]!=inf) return dp[L][R];
	int &ans=dp[L][R];
	if(L>R) return ans=0;
	if(L==R) return ans=1;
	for(int i=0; i<4; i+=2){
		if(s[L]==match[i]&&s[R]==match[i^1]){		// can only match when the brackets are in the right order
			path[L][R]=-1;
			ans=dfs(L+1, R-1);
		}
	}
	for(int i=L; i<R; i++){
		if(ans>dfs(L, i)+dfs(i+1, R)){
			ans=dp[L][i]+dp[i+1][R];
			path[L][R]=i;
		}
	}
	return ans;
}
void print(int L, int R){
	if(L>R) return;
	if(L==R){
		if(s[L]=='('||s[L]==')') cout<<"()";
		else cout<<"[]";
		return;
	}
	if(path[L][R]==-1){
		cout<<s[L];
		print(L+1, R-1);
		cout<<s[R];
		return;
	}
	print(L, path[L][R]);
	print(path[L][R]+1, R);
}
int main(){
	fgets(s+1, 120, stdin);
	int n=strlen(s+1);
	n--;
	if(!n){
		puts("");
		return 0;
	}
	memset(dp, 0x3f, sizeof(dp));
	dfs(1, n);
	print(1, n);
	puts("");
	return 0;
}
```
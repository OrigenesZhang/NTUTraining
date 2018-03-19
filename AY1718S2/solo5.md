# Training Report

Author: Origenes  
Team: NTU_Origenes  
Time: 14 Mar 2018  

The report is for the fifth solo traning session of AY17/18 sem 2.  
The topic of the training is data structure.  

***

### Problem Analysis

##### Problem A Stars
&nbsp;
This is a simple Fenwick tree problem. Since the problem is asking about the number of points in the downleft quarter of the points and points are sorted respect to y-x coordinates in the input, we can construct a Fenwick tree regarding the x-coordinates. Each time we read in a point, we make a query first and then update the Fenwick tree.  
Time complexity: O(n\*log(n))  

**Code:**
```cpp
#include <cstdio>
#include <iostream>
using namespace std;
const int maxn=32123;
int ans[maxn], C[maxn], N;
inline int lowbit(int x){
	return x&-x;
}
int sum(int x){
	int ret=0;
	while(x>0){
		ret+=C[x];
		x-=lowbit(x);
	}
	return ret;
}
void add(int x, int d){
	while(x<=maxn){
		C[x]+=d;
		x+=lowbit(x);
	}
}
int main(){
	cin>>N;
	for(int i=0; i<N; i++){
		int x, y;
		scanf("%d%d", &x, &y);
		x++;
		ans[sum(x)]++;
		add(x, 1);
	}
	for(int i=0; i<N; i++)
		printf("%d\n", ans[i]);
	return 0;
}
```

##### Problem B Find them, Catch them
&nbsp;
The question is asking us to maintain the relation of the points, which suggests of using DSU.  
In a typical DSU, only merge and query operations are supported. But in this problem, we need to maintain the so-called opposite relation. This can be done through double the size of the parent array (or equivalently, use an extra array to represent the opposites of the points).  
Each time a new relation is added, we make queries respecting to each party whether they have got an enemy or not. If no, we simply add one of them to the diff set. If yes, me need to merge their enemies.  
Time complexity: O(T\*N\*log(M))  

**Code:**
```cpp
#include <cstdio>
#include <cstring>
using namespace std;
const int maxn=123456;
char ops[5];
int p[maxn], diff[maxn], N, M, T;
int Find(int x){
	return x==p[x]?x:p[x]=Find(p[x]);
}
void Union(int u, int v){
	int fu=Find(u), fv=Find(v);
	if(fu!=fv) p[fu]=fv;
}
void Op(int u, int v){
	if(diff[u]==-1) diff[u]=v;
	else Union(diff[u], v);
	if(diff[v]==-1) diff[v]=u;
	else Union(diff[v], u);
}
void init(){
	memset(diff, -1, sizeof(diff));
	for(int i=1; i<=N; i++) p[i]=i;
}
int main(){
	scanf("%d", &T);
	while(T--){
		scanf("%d%d", &N, &M);
		init();
		while(M--){
			scanf("%s", ops);
			int u, v;
			scanf("%d%d", &u, &v);
			if(ops[0]=='D') Op(u, v);
			else{
				if(Find(u)==Find(v)) puts("In the same gang.");
				else if(Find(diff[u])==Find(v)) puts("In different gangs.");
				else puts("Not sure yet.");
			}
		}
	}
	return 0;
}
```

##### Problem C Fence Repair
&nbsp;
The question can be solved by a greedy approach. Considering the reverse process, which is concating the blocks to make them together with the given cost constrain, intuitively, if we merge two shortest segments each time repetitively, we would reach an optimal solution. This can be done by maintaining a min heap (or simply use built-in priority queues).  
The proof of the correctness of the approach is not so intuitive. But we can convert this problem to a Huffman encoding (Huffman tree) problem and hence the correctness can be proven.  
Time complexity: O(N\*log(N))  

**Code:**
```cpp
#include <iostream>
#include <queue>
using namespace std;
typedef long long ll;
priority_queue<ll, vector<ll>, greater<ll> > q;
int N, L;
int main(){
	cin>>N;
	for(int i=0; i<N; i++){
		cin>>L;
		q.push(L);
	}
	ll ans=0;
	while(q.size()>1){
		ll u=q.top(); q.pop();
		ll v=q.top(); q.pop();
		ans+=u+v;
		q.push(u+v);
	}
	cout<<ans<<endl;
	return 0;
}
```

##### Problem D Area of Simple Polygons
&nbsp;
The problem would probably be the hardest amongst the problemset if you have never seen this type of problem before. The hard part of it is the input data is 2D instead of 1D. Therefore, a conversion from 2D to 1D would be very helpful if possible.  
Notice that the edges of the rectangles are parallel to either x-axis or y-axis, therefore if we only keep the edges parallel to x-axis (or y-axis) and tag them with signals indicating the upper or lower edge in rectangles, the shape of the graph could still be kept. One possible solution is to scan through bottom to top and maintaining the length of the union of all segments at the current y coordinate and update the answer whenever a change in length takes place. This can be done by a segment tree.  
Time complexity: O(N\*log(max(|x|)))  

**Code:**
```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;
const int maxn=51234;
const int maxlen=1<<16;
const int maxm=1234;
int addv[4*maxn], minv[4*maxn], lenv[4*maxn];
int N, _len, a, b, ans, flag;
int maintain2(int o, int L, int R){
	if(minv[o]) return lenv[o]=R-L+1;
	if(L==R) return lenv[o]=0;
	int lc=o*2, rc=lc+1, M=L+R>>1;
	lenv[o]=0;
	if(a<=M) lenv[o]+=maintain2(lc, L, M);
	else lenv[o]+=lenv[lc];
	if(b>M) lenv[o]+=maintain2(rc, M+1, R);
	else lenv[o]+=lenv[rc];
	return lenv[o];
}
void maintain1(int o, int L, int R){
	int lc=o*2, rc=lc+1;
	minv[o]=0;
	if(R>L) minv[o]=min(minv[lc], minv[rc]);
	minv[o]+=addv[o];
	lenv[o]=maintain2(o, L, R);
}
void add(int o, int L, int R, int v){
	int lc=o*2, rc=lc+1;
	if(a<=L&&R<=b) addv[o]+=v;
	else{
		int M=L+R>>1;
		if(a<=M) add(lc, L, M, v);
		if(b>M) add(rc, M+1, R, v);
	}
	maintain1(o, L, R);
}
void query(int o, int L, int R){
	if(a<=L&&R<=b) _len+=lenv[o];
	else{
		int M=L+R>>1;
		if(a<=M) query(o*2, L, M);
		if(b>M) query(o*2+1, M+1, R);
	}
}
void init(){
	memset(addv, 0, sizeof(addv));
	memset(lenv, 0, sizeof(lenv));
	ans=0;
	N=0;
	flag=true;
}
struct Seg{
	int x1, x2, y, v;
	bool operator < (const Seg& rhs) const{
		if(y!=rhs.y) return y<rhs.y;
		return v<rhs.v;
	}
}segs[maxm*2];
void solve(){
	sort(segs, segs+N);
	int cury=0;
	for(int i=0; i<N; i++){
		if(cury!=segs[i].y){
			a=0, b=49999, _len=0;
			query(1, 0, maxlen-1);
			ans+=_len*(segs[i].y-cury);
			cury=segs[i].y;
		}
		a=segs[i].x1, b=segs[i].x2;
		add(1, 0, maxlen-1, segs[i].v);
	}
}
int main(){
	int x1, y1, x2, y2;
	while(scanf("%d%d%d%d", &x1, &y1, &x2, &y2)==4){
		if(x1==-1){
			if(flag)
				return 0;
			solve();
			printf("%d\n", ans);
			init();
		}else{
			flag=false;
			x2--;
			Seg s1={x1, x2, y1, 1}, s2={x1, x2, y2, -1};
			segs[N++]=s1; segs[N++]=s2;
		}
	}
}
```

##### Problem E Black Box
&nbsp;
The problem can be solved by a max heap and a min heap.  
We need to ensure that after the ith query and before the (i+1)th query, the size of the max heap should always be i and all elements inside the maxheap should be smaller than the smallest element in the min heap. Each time after a query, we move the smallest one of the min heap to the max heap and each time after an addition, we maintain both heaps to fit the first requirement.  
Time complexity: O((N+M)\*log(N+M))  

**Code:**
```cpp
#include <cstdio>
#include <queue>
using namespace std;
const int maxn=31234;
priority_queue<int, vector<int>, greater<int> > p;		// min heap
priority_queue<int> q;		// max heap
int M, N, a[maxn];
int main(){
	scanf("%d%d", &M, &N);
	for(int i=0; i<M; i++)
		scanf("%d", &a[i]);
	int u, cur=0;
	for(int i=0; i<N; i++){
		scanf("%d", &u);
		while(cur<u){
			p.push(a[cur++]);
			if(!q.empty()&&q.top()>p.top()){
				int tmp1=p.top(), tmp2=q.top();
				p.pop(), q.pop();
				q.push(tmp1), p.push(tmp2);
			}
		}
		int tmp=p.top(); p.pop();
		printf("%d\n", tmp);
		q.push(tmp);
	}
	return 0;
}
```
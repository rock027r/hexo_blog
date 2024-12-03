

# 算法学习笔记

## 前缀和

**前缀和**（Prefix Sum）是一种常用的算法技巧，用于快速计算数组某个区间的元素总和。它通过预处理数组，将计算区间和的时间复杂度从 O(n) 降低到 O(1)。

例:计算区间 l 到 r 的和

$prefix[i]=\sum_{j=1}^i a[j] $

$sum=prefix[r]−prefix[l−1]$

### 一维前缀和

```c++
#include<bits/stdc++.h>
using namespace std;
const int N = 1e5 + 10;
int a[N],p[N];

int main()
{
    int n;cin >> n;
    for(int i = 1;i <= n;i++)
        cin >> a[i];
    for(int i = 1;i <= n;i++)
        p[i] = a[i] + p[i-1];
    int q;cin >> q;
    while(q--)
    {
		int l,r;cin >> l >> r;
        cout << p[r] - p[l-1] << '\n';
    }
	return 0;
}
```


###### tags: `Algorithm`
# Algorithm
## Homework1
[助教的](https://hackmd.io/@arasHi87/SJBPOCKWc)
## Homework2
[助教的](https://hackmd.io/@idoleat/Sk9tifsfq)
### 逆數序對
#### 解法:
將問題拆分成左半數列的 inversion number 加上右邊數列的 inversion number，要注意的是模數運算 (mod) 要在 overflow 之前就做，模數相關的運算規則可以參考 wiki，例如先取模數再加、減、乘和算完再取模數是一樣的。
:::spoiler
```cpp=1
#include<stdio.h>
#define SIZE 1000000
#define MOD 524287
int n,arr[SIZE],tmp[SIZE/2],i,ti,f;
long long sum;
void print(){
    for(int i=0;i<6;i++){
        printf("%d ",arr[i]);
    }
    printf("\n");
}
void count(int *left, int *right){ //right 其實是超過的index的
    if(right - left <= 1)
        return;
    
    int *m = left + (right - left) / 2; //指標相加做運算
    
    count(left,m);
    count(m,right);
    int i, j = 0, ls = m - left, rs = right - m;
    for(i = 0; i != ls; ++i)
        *(tmp + i) = *(left +i);        //複製
    for(i = 0; i != ls && j != rs; left++){
        if(*(tmp + i) <= *(m + j)){
            *left = *(tmp +i);
            i++;
            sum += j;
        }
        else{
            *left = *(m+j);
            j++;
        }
    }
    for(; i != ls; i++, left ++){
        *left = *(tmp + i);
        sum += j;
    }
    print();
}
int main (){
    scanf("%d", &n);
    for(i = 0; i != n; ++i)
        scanf("%d", arr + i);
    count(arr, arr + n);
    printf("%lld", sum % MOD);
    return 0;
}
```
:::
**圖解:**
:::spoiler
![](https://i.imgur.com/88zgYcL.png)
![](https://i.imgur.com/MiF4gRR.png)
![](https://i.imgur.com/9Z1j3y4.png)
:::


### 小嵐的大樂透
#### 解法:
:::spoiler
```cpp=1
#include <iostream>

using std::cin;
using std::cout;
using std::endl;
using std::ios_base;

int k, a[13], i, b[6];

void print() {
  cout << b[0];
  for (i = 1; i != 6; ++i)
    cout << ' ' << b[i];
  cout << endl;
}

void comb(int x, int y) {
  if (y == 6) {
    print();
    return;
  }
  if (x == k)
    return;
  int j;
  for (j = x; j != k; ++j) {
    b[y] = a[j];
    comb(j + 1, y + 1);
  }
}

int main() {
  ios_base::sync_with_stdio(false);
  cin.tie(NULL);
  cin >> k;
  for (i = 0; i != k; ++i)
    cin >> a[i];
  comb(0, 0);
  cout << endl;

  return 0;
}
:::

**圖解:**
:::spoiler
![](https://i.imgur.com/xt5f6LG.png)
:::

### 2DRanking
**解法:**
可用分治法（Divide And Conquer）的經典題目。以下為步驟：
一：將所有點按照 x 值由小到大排序，x 值一樣的則將 y 值由大到小排。

二：判斷點的個數，如果只有一個就不須做以下任何事。單一一個點的排名為 0 。

三：找到所有點的 x 座標之中位數。因為已經排序了，所以是一串數字中正中間的數字，偶數個數字則取最靠近的其中一個即可。然後以此作為分水嶺，將點分為左右兩側為 L 以及 R。

四：分別遞迴解 L 以及 R 。也就是回到步驟二。

五：合併 L 以及 R 的解——採用類似合併排序（Merge Sort）的方式合併，用兩個指標指向 L 以及 R 的開頭。每次看目前指到的點誰的 y 座標較小。如果是 L 方的，則表示接下來的所有剩下的在 R 中的點將支配 L 的該點（因此用一變數 c 儲存目前已經有多少的 L 點，c 一開始為 0）；如果是 R 方的，則將 R 該點的排名加上 c（表示其支配了 L 中的 c 個點）。

做完以上步驟即可得到每個點的排名。但是因為本題的輸出是要照原本的輸入順序而輸出排名的，因此我們需要額外紀錄每個點原本的索引值。當要更新排名時，就根據原本的索引值去更新對應的排名值，最後便可以直接輸出排名值。
:::spoiler 網路
```cpp=1
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

int ranks[10000];

struct point {
	int x, y, index;

	bool operator<(const point &other) const {
		if (this->x != other.x)
			return this->x < other.x;
		return this->y > other.y;
	}
} points[10000], buffer[10000];

void Rank2D(int lower, int upper) {
	if (upper - lower <= 1)
		return;
	int middle = (lower + upper) >> 1, medianX = points[middle].x, counts = 0, i = lower, j = middle, k = lower;
	Rank2D(lower, middle); Rank2D(middle, upper);
	while (i < middle || j < upper) {
		if (i == middle) {
			buffer[k] = points[j]; ranks[buffer[k].index] += counts;
			++j; ++k;
		}
		else if (j == upper) {
			buffer[k] = points[i];
			++i; ++k;
		}
		else if (points[i].y < points[j].y) {
			buffer[k] = points[i]; ++counts;
			++i; ++k;
		}
		else {
			buffer[k] = points[j]; ranks[buffer[k].index] += counts;
			++j; ++k;
		}
	}
	for (i = lower; i < upper; ++i)
		points[i] = buffer[i];
}

int main() {
	cin.sync_with_stdio(false); cin.tie(nullptr);
	int amount;
	while (cin >> amount) {
		memset(ranks, 0, sizeof(ranks));
		for (int i = 0; i < amount; ++i) {
			cin >> points[i].x >> points[i].y;
			points[i].index = i;
		}
		sort(points, points + amount);
		Rank2D(0, amount);
		for (int i = 0; i < amount; ++i)
			cout << ranks[i] << '\n';
	}
}
```
:::
:::spoiler 助教
```cpp
#include <stdio.h>
#include <stdlib.h>

typedef struct{
    int i, x, y;
} point;

int cmp(const void *a, const void *b){
    
    if ((*(point *)a).x == (*(point *)b).x){
        if ((*(point *)a).y < (*(point *)b).y)
            return -1;
        if ((*(point *)a).y == (*(point *)b).y)
            return 0;
        return 1;
    }
    if ((*(point *)a).x < (*(point *)b).x)
        return -1;
    return 1;
}

int n, ranks[300000];
point ps[300000];

void find_ranks(int first, int last){
    
    if (last - first <= 1)
        return;

    int mid = (first + last) / 2;
    find_ranks(first, mid);
    find_ranks(mid, last);

    int i, j = 0, k = mid, size = mid - first;
    point *tps = (point *)malloc(size * sizeof(point));
    for (i = 0; i != size; ++i)
        tps[i] = ps[i + first];

    for (i = first;; ++i){
        
        if (tps[j].y <= ps[k].y){
            
            ps[i] = tps[j++];
            if (j == size){
                for (; k != last; ++k)
                    ranks[ps[k].i] += j;
                break;
            }
        }
        else{
            
            ps[i] = ps[k];
            ranks[ps[k].i] += j;
            k++;
            if (k == last){
                
                for (++i; i != last; ++i, ++j)
                    ps[i] = tps[j];
                break;
            }
        }
    }
    free(tps);
}

int main(int argc, char *args[]){
    scanf("%d", &n);
    int i;
    for (i = 0; i != n; ++i){
        
        scanf("%d%d", &ps[i].x, &ps[i].y);
        ps[i].i = i;
        ranks[i] = 0;
    }

    qsort(ps, n, sizeof(point), cmp);

    find_ranks(0, n);

    printf("%d", ranks[0]);
    for (i = 1; i != n; ++i)
        printf(" %d", ranks[i]);
    puts("");
}
```
:::
:::spoiler 圖解
![](https://i.imgur.com/xp3hPlU.png)
:::
:::spoiler 自己的
```cpp
#include <cstdio>
#include <algorithm>
#include <cstring>
using namespace std;
static int ranks[300009];
typedef struct point POINT;

struct point{
	int x, y, index;
} temp[300009], points[300009];

void merge_recursive(int left, int mid, int right){
    int i, j=mid+1, k;
    POINT L[(mid - left + 1)], R[(right - mid)];

    for (i = 0; i < (mid - left + 1); i++)
        L[i] = points[left + i];

    for (j = 0; j < (right - mid); j++)
        R[j] = points[mid + 1 + j];

    i = 0; j = 0; k = left;

    while (i < (mid - left + 1) && j < (right - mid)) {
        if (L[i].x < R[j].x)
            points[k] = L[i++];

        else if(L[i].x == R[j].x && L[i].y < R[j].y)
            points[k] = L[i++];

        else
            points[k] = R[j++];
        k++;
    }

    while (i < (mid - left + 1))
        points[k++] = L[i++];

    while (j < (right - mid))
        points[k++] = R[j++];
}
void mergeSort_recursive(int left, int right){
    if (left < right) {
        int mid = (left + right) / 2;
        mergeSort_recursive(left, mid);
        mergeSort_recursive(mid + 1, right);
        merge_recursive(left, mid, right);
    }
}
void findRank(int l, int u) {
	if (u - l <= 1)
        return;
	int m = (l + u) /2, counts = 0, i = l, j = m, k = l;
	findRank(l, m);
	findRank(m, u);
	while (i < m || j < u) {
		if (i == m) {
			temp[k] = points[j]; ranks[temp[k].index] += counts;
			++j;
		}
		else if (j == u) {
			temp[k] = points[i];
			++i;
		}
		else if (points[i].y <= points[j].y) {
			temp[k] = points[i];
			++counts;
			++i;
		}
		else {
            temp[k] = points[j];
            ranks[temp[k].index] += counts;
            int temp_num = j;

            while(points[i].x == points[temp_num].x){
                ranks[points[i].index] ++;
                temp_num++;
            }
            ++j;
		}
		++k;
	}
	for (i = l; i < u; ++i)
		points[i] = temp[i];

}
int main() {
	long long int num;
	scanf("%d",&num);
	for(int i=0;i<num;i++){
		scanf("%d %d",&(points[i].x),&(points[i].y));
        points[i].index = i;
    }
    mergeSort_recursive(0,num-1);
    findRank(0, num);
    for (int i = 0; i < num; ++i){
        printf("%d ",ranks[i]);
	}
}
```
:::
## 紙筆作業
[紙筆作業](https://hackmd.io/@joey3639570/HkEIUT2-9)
3
:::spoiler
![](https://i.imgur.com/xh00hWk.png)
:::
4
## HW3
[助教](https://hackmd.io/@joey3639570/SkkoIFGmc)
## HW4
[助教](https://hackmd.io/@joey3639570/H1lH46zQc)
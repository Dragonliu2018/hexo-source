---
title: STL
tags:
  - STL
categories:
  - 02-数据结构
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-19 13:10:40
---

# 1 vector(向量)

vector是与数组相关的序列式容器，set模版类的定义在头文件\<vector\>中。当我们在程序中需要使用动态数组时， vector将会是理想的选择， vector可以在使用过程中动态地增长存储空间。

```c++
// 定义及初始化
vector<int> s;// 定义⼀个空的vector对象，存储的是int类型的元素
vector<int> s(n, 1);// 定义⼀个含有n个int元素(1)的vector对象
vector<int> s(first, last);// 从由迭代器first和last定义的序列[first, last)中复制初值初始化
vector<int> s = {1,7,3,6,5,6};

// 基本操作
s[i] // 直接以下标⽅式访问容器中的元素，编号从0开始
s.front() // 返回⾸元素
s.back() // 返回尾元素
s.push_back(x) // 向表尾插⼊元素x
s.pop_back() // 删除表尾元素
s.size() // 返回表⻓
s.empty() // 表为空时，返回真，否则返回假
s.begin() // 返回指向⾸元素的随机存取迭代器
s.end() // 返回指向尾元素的下⼀个位置的随机存取迭代器

// 插入
s.insert(it, val); // 向迭代器it指向的元素前插⼊新元素val
s.insert(s.begin()+pos, val);
s.insert(it, n, val); // 向迭代器it指向的元素前插⼊n个新元素val
s.insert(it, first, last);// 向迭代器it指向的元素前插⼊迭代器first和last所指定的序列[first, last)

s.erase(it) // 删除由迭代器it所指向的元素
s.erase(first, last) // 删除由迭代器first和last所指定的序列[first, last)
s.reserve(n) // 预分配缓冲空间，使存储空间至少可容纳n个元素
s.resize(n) // 改变序列长度，超出的元素将会全部被删除，需要扩展(原空间小于n)的用元素默认值填满
s.resize(n, val) // 改变序列长度，超出的元素将会全部被删除，需要扩展(原空间小于n)的用元素val填满
s.clear() // 删除容器中的所有元素。resize和clear是对表的有效元素进行的操作，但不⼀定改变缓冲空间的⼤小
s.swap(v) // 将s与另⼀个vector对象进行交换
s.assign(first, last)// 将序列替换成由迭代器指示的[first, last)，不能是原序列中的⼀部分
operator: > < >= <= == != //按照字典序⽐较两个序列
 
// 组合操作
s.insert(s.begin(), x) // 向表头插⼊元素x
s.erase(s.begin()) // 删除表头元素
reverse(s.begin(), s.end()) // 反转向量
sort(s.begin(), s.end()); // 排序
    
// 函数返回一个vector
return vector<int> {a, b};
```

# 2 pair(二元组)

pair用来表示一个二元组或元素对，定义在头文件\<utility\>中。pair模版类需要两个参数：首元素的数据类型和尾元素的数据类型。 pair模版类对象有两个成员： first和second，分别表示首元素和尾元素。  

```c++
// 定义
pair<double, double> p;
p = make_pair(int, int);

// 基本操作
operator: < > <= >= == != // 其规则是先⽐较first， first相等时再比较second，这符合⼤多数应⽤的逻辑
```

# 3 set(集合)

set是与集合相关的关联式容器，set模版类的定义在头文件\<set\>中。在set中每个元素的值都唯一，而且系统能根据元素的值自动进行排序(默认升序)。set中元素的值不能直接被改变。

标准库提供set关联容器分为：

1. 按关键字有序保存元素：set(关键字即值，即只保存关键字的容器)；multiset(关键字可重复出现的set)；
2. 无序集合：unordered_set(用哈希函数组织的set)；unordered_multiset(哈希组织的set，关键字可以重复出现)。

```c++
// 定义
set<T> s;
set<T>::iterator it;//迭代器
int arr[]={0,1,2,3};
set<int> ss(arr, arr+4);//数组初始化
set<int> ss(s);//集合初始化

// 基本操作
s.begin() // 返回指向第⼀个元素的迭代器，如果set为空则返回的迭代器不能解引用
s.clear() // 清除所有元素
s.count(T) // 返回某个元素的个数，多重集合也是返回个数
s.empty() // 如果集合为空，返回true(真）
s.end() // 返回指向最后⼀个元素之后的迭代器，不是最后⼀个元素
s.equal_range(T) // 返回集合中与给定值相等的上下限的两个迭代器(pair)
s.erase(T) // 删除迭代器指定位置的元素或与对象匹配的元素
s.find(T) // 返回⼀个指向被查找到元素的迭代器
s.get_allocator() // 返回集合的分配器
s.insert(T) // 在集合中插⼊元素
s.lower_bound(T) // 返回指向⼤于（或等于）某值的第⼀个元素的迭代器
s.key_comp() // 返回⼀个⽤于元素间值⽐较的函数，默认operator<
s.max_size() // 返回集合能容纳的元素的最⼤限值
s.rbegin() // 返回指向集合中最后⼀个元素的反向迭代器
s.rend() // 返回指向集合中第⼀个元素的反向迭代器
s.size() // 集合中元素的数⽬
s.swap(set<T>) // 交换两个集合变量
s.upper_bound(T) // 返回⼤于某个值元素的迭代器
s.value_comp() // 返回⼀个⽤于⽐较元素间的值的函数
//遍历输出
for (auto ptr = s.begin(); ptr != s.end(); ptr++) {
	cout<< *ptr<< " ";
}
// 修改排序
struct cmp {
    bool operator () (const int& u, const int& v) const {
        if (abs(u - v) <= k) return false;
        return u < v;
    }
};
set<int, cmp> se;
```

# 4 string(字符串)

string类的定义在头文件$<string>$中。string类其实可以看作是一个字符的vector， vector上的各种操作都可以适用于string，另外，string类对象还支持字符串的拼合、转换等操作。  

```c++
// 定义
string str;

// 基本操作
operator + // 拼合
//string转char指针
char *p=(char*)str.c_str();
char*p=(char*)str.data();
// 关于输入
cin>> str;//接受空、回车格结尾的字符串
cin.ignore();//配合使用，清除以回车结束的输入缓冲区的内容 
getline(cin, str);//读取一行
//字串型的串流，用于按空格间隔的字串分割
stringstream ss;
ss<< str;//输入str到ss(屏幕)，看方向
ss>> str;//ss(屏幕)输入到str
//@brief: 指定单个分隔符（单个字符或子串）分割字符串
//@param: src 原字符串；delimiter 分隔符，单个字符或子串
vector<string> splitStr(const string& src, const string& delimiter) {
	vector<string> vetStr;
	
	// 入参检查
	// 1.原字符串为空或等于分隔符，返回空 vector
	if (src == "" || src == delimiter) {
		return vetStr;
	}
	// 2.分隔符为空返回单个元素为原字符串的 vector
	if (delimiter == "") {
		vetStr.push_back(src);
		return vetStr;
	}

	string::size_type startPos = 0;
	auto index = src.find(delimiter);
	while (index != string::npos) {
		auto str = src.substr(startPos, index - startPos);
		if (str != "") {
			vetStr.push_back(str);
		}
		startPos = index + delimiter.length();
		index = src.find(delimiter, startPos);
	}
	// 取最后一个子串
	auto str = src.substr(startPos);
	if (str != "") {
		vetStr.push_back(str);
	}

	return vetStr;
}
//char字符处理
isalpha(ch);//字母（不区分大小写）
isupper(ch);//大写字母
islower();//小写字母
isdigit();//数字
isalnum();//字母或数字
toupper();//转大写
tolower();//转小写

//string转int等类型
int stoi (const string&  str, size_t* idx = 0, int base = 10);//*idx是指向size_t类型对象的指针，其值由函数设置为数值后str中下一个字符的位置；base是指str的基数，默认十进制
long stol (const string&  str, size_t* idx = 0, int base = 10);
unsigned long stoul (const string&  str, size_t* idx = 0, int base = 10);
long long stoll (const string&  str, size_t* idx = 0, int base = 10);
unsigned long long stoull (const string&  str, size_t* idx = 0, int base = 10);
float stof (const string&  str, size_t* idx = 0);
double stod (const string&  str, size_t* idx = 0);
long double stold (const string&  str, size_t* idx = 0);

//int等类型转string
string to_string (T);

//find函数
str.find(tmp_str);//返回下标位置，没有则返回str.npos
str.find_first_of(tmp_str);//第一次出现的位置
str.find_last_of(tmp_str);//最后一次出现的位置
str.find(tmp_str, pos);//从下标pos开始查找
str.rfind(tmp_str);//反向查找，与正向查找结合确定子串的唯一性

//取子串
string substr (size_t pos = 0, size_t len = npos) const;//在字符位置pos开始，跨越len个字符（或直到字符串的结尾，以先到者为准）对象的部分

//字符串大小写转化
transform(s.begin(),s.end(),s.begin(),::tolower);
transform(s.begin(),s.end(),s.begin(),::toupper);
```

# 5 list(列表)

list是由双向链表实现的，因此内存空间是不连续的。只能通过指针访问数据，所以list的随机存取非常没有效率，时间复杂度为$O(n)$；但由于链表的特点，能高效地进行插入和删除。

使用：

* 需要高效的随机存取，而不在乎插入和删除的效率，使用vector;
* 如果需要大量的插入和删除，而不关心随机存取，则应使用list

```C++
// 定义
list<int> l;//声明一个空列表
list<int> l(n);//声明一个有n个元素的列表，每个元素都是由其默认构造函数T()构造出来的
list<int> l(n,val);//声明一个由n个元素的列表，每个元素都是由其复制构造函数T(val)得来的
list<int> l();//声明一个空列表
list<int> l(first,last);//声明一个列表，其元素的初始值由区间所指定的序列中的元素;

// 基本操作
l.begin() // 得到一个指向容器起始位置的iterator
l.end() //返回list末端下一位置，如a[n]中的第n+1个位置a[n]，实际是不存在，经常作为循环结束判断条件使用
l.push_back(x) //list的末端插入x
l.push_front(x) // 从list的头部插入x
l.empty() // 利用empty() 判断list是否为空，空返回true
l.resize(n) //将l的长度改为n，超出的元素将被删除，如果需要扩展那么调用默认构造函数T()将元素加到list末端
l.resize(n, val)//扩展元素要调用构造函数T(val)函数进行元素构造，其余部分与resize相同
l.clear() //清空list中的所有元素
l.front() //获得list容器中的头部元素
l.back() //可以获得list容器的最后一个元素
l.pop_back // 删除最后一个元素
l.pop_front() // 删除第一个元素；序列必须不为空，当list为空时调用pop_back和pop_front会使程序崩掉
l.assign() //具体和vector中的操作类似
l.swap(ll) //交换两个链表，swap(l ,ll)也可
merge() //合并两个链表并使之默认升序(也可改)，l1.merge(l2，greater<int>()); 调用结束后l2变为空，l1中元素包含原来l1 和 l2中的元素，并且排好序，升序。其实默认是升序，greater<int>()可以省略，另外greater<int>()是可以变的，也可以不按升序排列。
```

# 6 stack(栈)

stack模版类的定义在\<stack\>头文件中，stack模版类需要两个模版参数，一个是元素类型，另一个是容器类型，但是只有元素类型是必要的，在不指定容器类型时，默认容器的类型为deque。

```c++  
// 定义
stack<int> s;

// 基本操作
s.push(x); // ⼊栈
s.pop(); // 出栈，不返回值
x = s.top(); // 访问栈顶，
s.empty(); // 当栈空时，返回true
s.size(); // 访问栈中元素个数
```

# 7 queue(队列)

queue模版类的定义在头文件\<queue\>中。queue与stack相似， queue模版类也需要两个模版参数，一个元素类型，一个容器类型，元素类型是必须的，容器类型是可选的。  

标准库提供queue分为：

1. queue(队列)：queue从队首弹出，先入先出，并且queue只能从队首删除元素，但是两端都能访问。
2. deque(双向队列)：可以访问两端并且可以在队首和队尾删除和插入元素
3. priority_queue(优先队列)：优先队列与队列的差别在于优先队列不是按照入队的顺序出队，而是按照队列中元素的优先权出队列（默认为大者优先，也可以通过指定算子来指定自己的优先顺序）；priority_queue模版类有三个模版参数，第一个是元素类型，第二个是容器类型，第三个是比较算子。其中后两者都可以忽略，默认容器为vector，默认算子为less，即小的往前排，大的往后排（出队列时列尾元素先出队）。  

```c++
// 队列定义
queue<int> q;

// 队列基本操作
q.push(x); // 从队尾⼊队列
q.pop(); // 从队首出队列
q.front(); // 访问队首元素
q.back(); // 访问队尾元素
q.empty(); // 如果队列为空，则返回true，否则返回false
q.size(); // 访问队列中的元素个数

// 双向队列定义
deque<int> dq; //创建一个int双端队列dq

// 双向队列基本操作
dq.empty(); //判断队列是否为空，为空返回true
dq.push_front(s); //将s从队头入队
dq.push_back(s); //将s从队尾入队，和普通队列方式一样
dq.front(); //只返回队头元素
dq.back(); //只返回队尾元素
dq.pop_front(); //将队头元素弹出
dq.pop_back(); //将队尾元素弹出
dq.clear(); //将队列清空
//组合操作
//C++中的queue自身是不支持clear操作的，但是双端队列deque是支持clear操作的。
queue<int> empty;
swap(empty, q);

// 优先队列定义
priority_queue<int> q;
priority_queue<pair<int, int> > qq; // 注意在两个尖括号之间⼀定要留空格，防止误判
priority_queue<int, vector<int>, greater<int> > qqq; // 定义小的先出队列

// 优先队列基本操作
q.empty() // 如果队列为空，则返回true，否则返回false
q.size() // 返回队列中元素的个数
q.pop() // 删除队首元素，但不返回其值
q.top() // 返回具有最高优先级的元素值，但不删除该元素
q.push(x) // 在基于优先级的适当位置插⼊新元素x
    
// 算子实现
class T{
	public:
		int x, y, z;
		T(int a, int b, int c) : x(a), y(b), z(c) {}
};
bool operator < (const T &tOne, const T &tTwo){
	return tOne.z < tTwo.z; // 按照z的顺序来决定tOne和tTwo的顺序
}

int main(){
	priority_queue<T> q;
	q.push(T(4, 4, 3));
	q.push(T(2, 2, 5));
	q.push(T(1, 5, 4));
	q.push(T(3, 3, 6));
	while (!q.empty()){
		T t = q.top();
		q.pop();
		cout << t.x << " " << t.y << " " << t.z << '\n';
	}
	return 0;
}
```

# 8 map(字典)

map是与字典相关的关联式容器，map模版类的定义在头文件$<map>$中，用有序二叉树表存储类型为pair<const Key, T>的元素对序列。  序列中的元素以const Key部分作为标识， map中所有元素的Key值必须是唯一的，multimap则允许有重复的Key值。unordered_map是无序 map 容器。

将map可以通过一个Key值来快速决定一个元素，因此非常适合于需要按照Key值查找元素的容器。map模版类需要四个模版参数，第一个是键值类型，第二个是元素类型，第三个是比较算子，第四个是分配器类型。其中键值类型和元素类型是必要的。

```c++
// 字典定义
map<string, int> m;

// 字典基本操作
/* 向map中插⼊元素 */
m[key] = value; // [key]操作是map很有特色的操作,如果在map中存在键值为key的元素对, 则返回该元素对的值域部分,否则将会创建⼀个键值为key的元素对,值域为默认值。所以可以⽤该操作向map中插⼊元素对或修改已经存在的元素对的值域部分。
m.insert(make_pair(key, value)); // 也可以直接调用insert方法插⼊元素对,insert操作会返回⼀个pair,当map中没有与key相匹配的键值时,其first是指向插⼊元素对的迭代器,其second为true;若map中已经存在与key相等的键值时,其first是指向该元素对的迭代器,second为false，并且不会更新value

/* 查找元素 */
int i = m[key]; // key不存在时,会创建键值为key(当另⼀个元素是整型时m[key]=0)的元素对
map<string, int>::iterator it = m.find(key); // 如果map中存在与key相匹配的键值时,find操作将返
//回指向该元素对的迭代器,否则,返回的迭代器等于map的end()(参见vector中提到的begin()和end()操作)

/* 删除元素 */
m.erase(key); // 删除与指定key键值相匹配的元素对,并返回被删除的元素的个数
m.erase(it); // 删除由迭代器it所指定的元素对,并返回指向下⼀个元素对的迭代器

/* 其他操作 */
m.size(); // 返回元素个数
m.empty(); // 判断是否为空，空返回true
m.clear(); // 清空所有元素
size_type count (const key_type& k) const;//按key搜索
```

# 9 bitset()

bitset模版类的定义在\<bitset\>头文件中，用来方便地管理一系列的bit位类。 bitset除了可以访问指定下标的bit位以外，还可以把它们作为一个整数来进行某些统计。bitset模板类需要一个模版参数，用来明确指定含有多少位。  

```c++
// 定义
const int MAXN = 32;
bitset<MAXN> bt; // bt 包括 MAXN 位，下标 0 ~ MAXN - 1，默认初始化为 0
bitset<MAXN> bt1(0xf); // 0xf 表示十六进制数 f，对应二进制 1111，将 bt1 低 4 位初始化为 1
bitset<MAXN> bt2(012); // 012 表示八进制数 12，对应二进制 1010，即将 bt2 低 4 位初始化为 1010
bitset<MAXN> bt3("1010"); // 将 bt3 低 4 位初始化为 1010
bitset<MAXN> bt4(s, pos, n); // 将 01 字符串 s 的 pos 位开始的 n 位初始化 bt4

// 基本操作
bt.any() // bt 中是否存在置为 1 的二进制位
bt.none() // bt 中不存在置为 1 的二进制位
bt.count() // bt 中置为 1 的二进制位的个数
bt.size() // bt 中二进制位的个数
bt[pos] // 访问 bt 中在 pos 处的二进制位
bt.test(pos) // bt 中在 pos 处的二进制位是否为 1
bt.set() // 把 bt 中所有二进制位都置为 1
bt.set(pos) // 把 bt 中在 pos 处的二进制位置为 1
bt.reset() // 把 bt 中所有二进制位都置为 0
bt.reset(pos) // 把 bt 中在pos处的二进制位置为0
bt.flip() // 把 bt 中所有二进制位逐位取反
bt.flip(pos) // 把 bt 中在 pos 处的二进制位取反
bt[pos].flip() // 同上
bt.to_ulong() // ⽤用 bt 中同样的二进制位返回⼀个 unsigned long 值
bt.to_string()//转字符串
os << bt // 把 bt 中的位集输出到 os 流
```

# 10 iterator(迭代器) 

iterator（迭代器）是用于访问容器中元素的指示器，从这个意义上说， iterator（迭代器）相当于数据结构中所说的“遍历指针”，也可以把iterator（迭代器）看作是一种泛化的指针。每种STL容器都有自己的iterator（迭代器）子类。

# 11 algorithm

```c++
// 最大值最小值
max_element(v.begin(),v.end()); //返回容器最大值的迭代器
min_element(v.begin(),v.end()); //返回容器最小值的迭代器

// 排序
sort(v.begin(), v.end(), cmp); //cmp可以自定义，默认升序排序
// 自定义cmp，使其降序
bool cmp(int a, int b){
	return a > b;	
}
// lamda表达式
sort(v.begin(), v.end(), [](vector<int>& a, vector<int>& b) {
    return a[1] < b[1];
});

reverse(v.begin(), v.end()); //反转容器
void * memset ( void * ptr, int value, size_t num );//将ptr所指向的某一块内存中的前num个字节的内容全部设置为ch指定的ASCII值， 第一个值为指定的内存地址，块的大小由第三个参数指定，这个函数通常为新申请的内存做初始化工作， 其返回值为指向s的指针，它是对较大的结构体或数组进行清零操作的一种最快方法。
fill(begin, last, val);//将[begin, last)赋值为val
memcpy(target, source, sizeof(int)*k);//复制source数组的前k个到target
memcpy(target, source, sizeof(source));//复制source数组到target
lower_bound(a, a+n, x);//查找>=x的第一个位置
__gcd(int x, int y)//求x与y的最大公约数
accumulate(vect.begin(), vect.end(), 0);//以0为累加初值累加
```

# 12 一些常量

|    name    |                          expresses                           |                            value                             |
| :--------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  INT_MIN   |          Minimum value for an object of type `int`           | `-32767` ($-2^{15}$​+1) or less { `-2147483647` ($-2^{31}$​+1)} |
|  INT_MAX   |          Maximum value for an object of type `int`           | `32767` ($2^{15}$-1​) or greater {`2147483647` ($2^{31}$-1)}  |
|  UINT_MAX  |      Maximum value for an object of type `unsigned int`      |         `65535` ($2^{16}-1$) or greater {$2^{32}-1$}         |
| LLONG_MIN  |     Minimum value for an object of type `long long int`      |         `-9223372036854775807` ($-2^{63}+1$) or less         |
| LLONG_MAX  |     Maximum value for an object of type `long long int`      |        `9223372036854775807` ($2^{63}-1$) or greater         |
| ULLONG_MAX | Maximum value for an object of type `unsigned long long int` |        `18446744073709551615` ($2^{64}-1$) or greater        |
|            |                                                              |                                                              |

# 13 一些技巧

## 13.1 ++ 和--的小技巧 

a++ 和 ++a 都是将 a 加 1，但是 a++ 返回值为 a，而 ++a 返回值为 a+1。如果只是希望增加 a 的值，而不需要返回值，则推荐使用 ++a，其运行速度会略快一些。  

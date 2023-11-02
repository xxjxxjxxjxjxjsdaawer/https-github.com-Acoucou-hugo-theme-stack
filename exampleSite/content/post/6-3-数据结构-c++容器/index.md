+++
author = "coucou"
title = "数据结构——c++容器"
date = "2023-08-01"
description = "数据结构专题之c++容器"
categories = [
    "数据结构"
]
tags = [
    "数据结构","c++容器"
]
+++

![](1.png)

## 数据结构——c++容器

### deque的基本使用

```c++
#include<iostream>
#include<deque>   // 双向数组 

/*
vector对于头部的插入删除效率低，数据量越大，效率越低
deque相对而言，对头部的插入删除速度回比vector快
vector访问元素时的速度会比deque快,这和两者内部实现有关
*/ 

using namespace std;

int main(){
	deque<int> d;
	
	for(int i=0; i<10; i++){
		d.push_back(i);
	}
	
	for(deque<int>::iterator it = d.begin(); it != d.end(); it++){
		cout<< *it << " ";
	}
	
	return 0;
} 
```

### list的基本使用

```c++
#include<iostream>
#include<list>

/*
lis.push_back() //尾插元素 
lis.push_front(elem);//在容器开头插入一个元素

lis.front(); //返回第一个元素.
lis.back(); //返回最后一个元素.

lis.pop_back();//删除容器中最后一个元素
lis.pop_front();//从容器开头移除第一个元素

lis.insert(pos,elem); //在pos位置插elem元素的拷贝，返回新数据的位置。
lis.erase(beg,end); //删除[beg,end)区间的数据，返回下一个数据的位置。
lis.erase(pos); //删除pos位置的数据，返回下一个数据的位置。
lis.remove(elem); //删除容器中所有与elem值匹配的元素。
lis.clear(); //移除容器的所有数据

lis.assign(beg, end); //将[beg, end)区间中的数据拷贝赋值给本身。
lis.assign(n, elem); //将n个elem拷贝赋值给本身。

lis.swap(lis1); //将lis与本身的元素互换。

lis.size(); //返回容器中元素的个数
lis.empty(); //判断容器是否为空
lis.resize(num); //重新指定容器的长度为num，若容器变长，则以默认值填充新位置

lis.reverse(); //反转链表
lis.sort(); //链表排序
*/

using namespace std;

// 迭代器遍历 
void print_list(list<int> &lis){
	for(list<int>::iterator it = lis.begin(); it != lis.end(); it++){
		cout<< *it <<" ";
	}
	cout<<endl;
}

int main(){
	list<int> lis;
	
	for(int i=0; i<10; i++){
		lis.push_back(i);
	}
	
	// 插入
	list<int>::iterator it = lis.begin();  //list容器的迭代器是双向迭代器，不支持随机访问
	++it;
	lis.insert(++it, 100);                 //it = it + 1;//错误，不可以跳跃访问，即使是+1 
	print_list(lis);
	
	// 删除
	lis.erase(++it);
	
	lis.remove(9);
	
	// 赋值 
	list<int> lis2(lis);
	list<int> lis3(lis.begin(), lis.end()); 
	
	list<int> lis4;
	lis4.assign(10, 100);
	
	// 交换 
	lis2.swap(lis4); 
	
	// 反转
	lis.reverse();
	print_list(lis); 
	
	// 排序
	lis.sort();    // 这里可以自定义规则 mycompare >> lis.sort(mycompare) 
	print_list(lis);
	
	lis.clear();
	
	return 0;
}
```

### map的基本使用

```c++
#include<iostream>
#include<string>
#include<map>  //map和multimap为关联式容器,底层结构是用二叉树实现。

/*
	map中所有元素都是pair
	pair中第一个元素为key（键值），起到索引作用，第二个元素为value（实值）
	所有元素都会根据元素的键值自动排序
	
	map和multimap区别：
		map不允许容器中有重复key值元素
		multimap允许容器中有重复key值元素	
*/

/*
m.size(); //返回容器中元素的数目
m.empty(); //判断容器是否为空
m.swap(st); //交换两个集合容器

m.insert( pair<T, T>(element) ); //在容器中插入元素。
m.clear(); //清除所有元素
m.erase(pos); //删除pos迭代器所指的元素，返回下一个元素的迭代器。
m.erase(begin, end); //删除区间[beg,end)的所有元素 ，返回下一个元素的迭代器。
m.erase(key); //删除容器中值为key的元素。

m.find(key); //查找key是否存在,若存在，返回该键的元素的迭代器；若不存在，返回set.end();
m.count(key); //统计key的元素个数
*/

/*
成对出现的数据，利用对组可以返回两个数据
	pair<type, type> p ( value1, value2 );
*/

using namespace std;

void print_map(map<int, string> &m){
	for(map<int, string>:: iterator it = m.begin(); it != m.end(); it++){
		cout<< it->first << it->second << endl;
	} 
}

// map容器自动按key排序 
int main(){
	map<int, string> m;
	int id = 0;
	string name;
	
	for(int i=0; i<5; i++){
		cin>> id >> name;
		m.insert(pair<int, string>(id, name));
	}

	map<int, string>:: iterator position = m.find(2);
	
	if(position != m.end()){
		cout<< position->first << " " << position->second << endl;
	}
	
	return 0;
}
```

### queue的基本使用

```c
#include<iostream>
#include<queue>

/*
入队 --- q.push()
出队 --- q.pop()
返回队头元素 --- q.front()
返回队尾元素 --- q.back()
判断队是否为空 --- q.empty()
返回队列大小 --- q.size()
*/

using namespace std;

int main(){
	queue<int> qu;
	
	for(int i=0; i<10; i++){
		qu.push(i);
	}
	
	cout<<qu.size()<<endl; 
	
	while(!qu.empty()){
		cout<<qu.front()<<" ";
		cout<<qu.back()<<" ";
		
		qu.pop(); 
	}
	
	return 0;
}
```

### set的基本使用

```c++
#include<iostream>
#include<set> // set/multiset属于关联式容器，底层结构是用二叉树实现。 

/*
set 所有元素都会在插入时自动被排序

set和multiset区别：
	set不允许容器中有重复的元素
	multiset允许容器中有重复的元素
*/ 

/*
s.size(); //返回容器中元素的数目
s.empty(); //判断容器是否为空
s.swap(st); //交换两个集合容器

s.insert(elem); //在容器中插入元素。
s.clear(); //清除所有元素
s.erase(pos); //删除pos迭代器所指的元素，返回下一个元素的迭代器。
s.erase(beg, end); //删除区间[beg,end)的所有元素 ，返回下一个元素的迭代器。
s.erase(elem); //删除容器中值为elem的元素。

s.find(key); //查找key是否存在,若存在，返回该键的元素的迭代器；若不存在，返回set.end();
s.count(key); //统计key的元素个数
*/

using namespace std;

void print_set(set<int> &s){
	for(set<int>:: iterator it = s.begin(); it != s.end(); it++){
		cout<< *it << " ";
	}
}

int main(){
	set<int> s;
	
	for(int i=10; i>0; i--){
		s.insert(i);
	}
	
	print_set(s);
	
	return 0;
}
```

### stack的基本使用

```c
#include<iostream>
#include<stack>

/*
入栈 --- s.push()
出栈 --- s.pop()
返回栈顶 --- s.top()
判断栈是否为空 --- s.empty()
返回栈大小 --- s.size()
*/

using namespace std;

int main(){
	stack<int> st; // 创建栈 
	
	for(int i=0; i<10; i++){
		st.push(i); // 入栈 
	}
	
	cout<<st.size()<<endl; 
	
	while( !st.empty() ){
		cout<<st.top()<<" ";  // 输出栈顶元素 
		
		st.pop(); // 出栈 
	}
	
	return 0;
}
```

### vector的基本使用

```c++
#include<iostream>
#include<vector>

/*
v.empty(); //判断容器是否为空
v.capacity(); //容器的容量
v.size(); //返回容器中元素的个数
v.resize(int num); //重新指定容器的长度为num，若容器变长，则以默认值填充新位置。
				//如果容器变短，则末尾超出容器长度的元素被删除。
				
v.push_back(ele); //尾部插入元素ele
v.pop_back(); //删除最后一个元素

v.insert(const_iterator pos, ele); //迭代器指向位置pos插入元素ele
v.insert(const_iterator pos, int count,ele); //迭代器指向位置pos插入count个元素ele

v.erase(const_iterator pos); //删除迭代器指向的元素
v.erase(const_iterator start, const_iterator end); //删除迭代器从start到end之间的元素
v.clear(); //删除容器中所有元素

v.at(int idx); //返回索引idx所指的数据
v.operator[]; //返回索引idx所指的数据

v.front(); //返回容器中第一个数据元素
v.back(); //返回容器中最后一个数据元素

v.swap(vec); // 互换容器 
*/

using namespace std;

void print_vector(vector<int> &v){
	for(vector<int>::iterator it = v.begin(); it != v.end(); it++){
		cout<< *it << " ";
	}
}

int main(){
	vector<int> v;
	
	for(int i=0; i<10; i++){
		v.push_back(i);
	}
	
	for(int i=0; i<v.size(); i++){
		cout<< v[i] << " ";
	}
	v.push_back(100);
	
	cout<<endl<< v.at(10) << endl;
	
	cout<< endl << v.capacity()<< endl;
	
	print_vector(v);
	
	return 0;
}
```


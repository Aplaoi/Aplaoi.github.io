---
category: 博客
tags: 
    - C++
    - STL
    - 语言基础
---

# 常用 C++ STL 容器的使用

我平时写算法题比较习惯用 C++，但是总是忘记 STL 容器的用法，所以就写了这篇文章来总结一下常用的 STL 容器，也算是给自己一个备忘吧。

----
## 共性

- 所有 STL 容器都可以使用迭代器进行初始化、遍历等操作。
- 所有 STL 容器都支持 `size()` 方法来获取容器的大小
- 所有 STL 容器都支持 `clear()` 方法来清空容器
- 所有 STL 容器都支持 `empty()` 方法来判断容器是否为空
- 所有 STL 容器都支持 `insert()` 方法来插入元素
- 所有 STL 容器都支持 `erase()` 方法来删除元素
- 所有 STL 容器都支持使用迭代器初始化（可以用另一个容器的 `begin()` 和 `end()` 方法）
- 所有 STL 容器都支持 `find()` 方法来查找元素
  - 对于 `vector`、`list` 等序列容器，`find()` 是一个全局函数，定义在 `<algorithm>` 头文件中，复杂度为 O(n)
  - 对于 `unordered_set`、`unordered_map` 等关联容器，`find()` 是成员函数，复杂度为 O(1)


## vector

`vector` 是 C++ STL 里最常用的动态数组容器。它可以根据需要动态调整大小，支持随机访问和高效的插入/删除操作（在末尾）。常用操作包括插入、删除、访问元素等。

`vector` 支持使用迭代器进行操作，其中 `vec.begin()` 返回指向第一个元素的迭代器，`vec.end()` 返回指向最后一个元素**后面**的迭代器。

### 初始化

`vector` 可以通过多种方式初始化，代码如下：

```cpp
// 默认初始化，创建一个空的vector
vector<int> vec;                
// 创建一个包含5个元素，默认值为0的vector
vector<int> vec2(5);            
// 创建一个包含5个元素，初始值为10的vector
vector<int> vec3(5, 10);        
// 使用初始化列表初始化
vector<int> vec4 = {1, 2, 3};   
// 使用另一个vector初始化
vector<int> vec5(vec4);         
// 使用迭代器初始化
vector<int> vec6(vec4.begin(), vec4.end());     
```

### 插入和删除

`vector` 可以使用 `push_back()` 方法在末尾插入元素，使用 `insert()` 方法在指定位置插入元素，使用 `pop_back()` 方法删除末尾元素，使用 `erase()` 方法删除指定位置的元素。

这里需要注意的是，`insert()` 和 `erase()` 方法会返回一个迭代器，指向插入或删除操作后的位置。其中 `insert()` 返回指向新插入第一个元素的迭代器，`erase()` 返回指向删除元素中最后一个元素后的下一个元素的迭代器。当然如果不需要使用迭代器，返回值也可以忽略，直接使用即可。

代码如下：
```cpp
auto it = vec.begin() + 3;
// 在末尾插入元素1
vec.push_back(1);               
// 在索引1的位置插入元素10
it = vec.insert(it, 10); 
// 也可以忽略返回值
// vec.insert(begin() + 9, 10);  
// 在索引1的位置插入3个元素20
it = vec.insert(it, 3, 20); 
// 在索引1的位置插入另一个vector的所有元素
it = vec.insert(it, vec4.begin(), vec4.end()); 

// 删除最后一个元素
vec.pop_back();                 
// 删除指定元素
it = vec.erase(it); 
// 删除一段元素
it = vec.erase(it, it + 3); 
```

### 查找元素

`vector` 需要使用全局函数 `find()`查找，它定义在 `<algorithm>` 头文件中。`find()` 函数接受三个参数：起始迭代器、结束迭代器和要查找的值，返回一个迭代器，指向找到的元素，如果没有找到则返回结束迭代器。

```cpp
auto it = find(vec.begin(), vec.end(), 10);
if (it != vec.end()) {
    cout << "Found 10 at index " << distance(vec.begin(), it) << endl;
} else {
    cout << "10 not found" << endl;
}
```

## unordered_set

`unordered_set` 是一个基于哈希表实现的集合容器，它**不允许重复元素**，并且元素的顺序是**不确定**的。常用操作包括插入、删除和查找元素，时间复杂度平均为 O(1)。

### 初始化

`unordered_set` 可以通过多种方式初始化，代码如下：

```cpp
// 默认初始化，创建一个空的unordered_set
unordered_set<int> uset;
// 使用初始化列表初始化
unordered_set<int> uset2 = {1, 2, 3};
// 使用另一个unordered_set初始化
unordered_set<int> uset3(uset2);
// 使用迭代器初始化
vector<int> temp = {4, 5, 6};
unordered_set<int> uset4(temp.begin(), temp.end());
```

### 插入和删除
`unordered_set` 可以使用 `insert()` 方法插入元素，使用 `erase()` 方法删除元素。
```cpp
uset.insert(1);
uset.insert(1); // 重复插入不会有任何效果
uset.insert({2, 3, 4}); // 可以一次插入多个元素
uset.erase(2);
// 通过迭代器删除元素
auto it = uset.find(3);
if (it != uset.end()) {
    uset.erase(it); 
}
```

## unordered_map

`unordered_map` 是一个基于哈希表实现的键值对容器，它**不允许重复的键**，并且键值对的顺序是**不确定**的。常用操作包括插入、删除和查找元素，时间复杂度平均为 O(1)。

### 初始化

`unordered_map` 可以通过多种方式初始化，代码如下：

```cpp
// 默认初始化，创建一个空的unordered_map
unordered_map<int, string> umap;
// 使用初始化列表初始化
unordered_map<int, string> umap2 = { {1, "one"}, {2, "two"}, {3, "three"} };
// 使用另一个unordered_map初始化
unordered_map<int, string> umap3(umap2);
```

### 插入和删除

`unordered_map` 可以使用 `insert()` 方法或者下标插入键值对，使用 `erase()` 方法删除键值对。

```cpp
umap.insert({1, "one"});
umap[2] = "two"; // 也可以使用下标操作符插入或更新元素
// 删除键为1的元素
umap.erase(1);
// 通过迭代器删除键为2的元素
auto it = umap.find(2);
if (it != umap.end()) {
    umap.erase(it); 
}
```

### 访问元素

`unordered_map` 可以使用下标操作符 `[]` 、迭代器或者 `at()` 方法访问元素。需要注意的是，使用下标操作符访问一个不存在的键时，会插入一个默认值如0，空字符串等，而 `at()` 方法则会抛出异常。迭代器可以指向`first` 和 `second` 成员来访问键和值。

```cpp
unordered_map<int, string> umap = { {1, "one"}, {2, "two"} };
string value1 = umap.at(4); // 会抛出异常
string value = umap[4]; // 会插入一个默认值0
auto it = umap.find(2);
if (it != umap.end()) {
    cout << it->first << ": " << it->second << endl;
}
```
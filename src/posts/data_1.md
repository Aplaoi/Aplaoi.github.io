---
category: 博客
tags: 
    - 数据结构
    - 链表
---

# 数据结构笔记（一）—— 链表（更新中）

写一点在学习数据结构时的笔记，主要是为了加深理解和备忘，也希望能帮助到和我一样在学习数据结构的同学。

----

## 单链表

单链表是一种线性数据结构，由一系列节点组成，每个节点包含数据和一个指向下一个节点的指针。单链表的第一个节点称为头节点，最后一个节点的指针指向空（null），表示链表的结束。

在 C++ 中，单链表的节点可以定义如下：

```cpp
typedef int DataType; 

class Node {
private:
    DataType data; // 节点数据
    Node* next;    // 指向下一个节点的指针
public:
    Node(DataType val) : data(val), next(nullptr) {} // 构造函数
    DataType getData() const { return data; }        // 获取节点数据
    Node* getNext() const { return next; }           // 获取下一个节点指针
    void setNext(Node* nextNode) { next = nextNode; } // 设置下一个节点指针
};

class LinkedList {
private:
    Node* head; // 头节点指针
public:
    LinkedList() : head(nullptr) {} // 构造函数
    ~LinkedList();                  // 析构函数

    void insertAtHead(DataType val); // 在头部插入节点
    void insertAtTail(DataType val); // 在尾部插入节点
    void deleteNode(DataType val);   // 删除指定值的节点
    Node* search(DataType val);      // 查找指定值的节点
    void display() const;            // 显示链表内容
};
```

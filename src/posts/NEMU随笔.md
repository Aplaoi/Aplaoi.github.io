---
category: 博客
tag:
  - 操作系统
  - 学校

---

写一点我在做Nemu实验的时候的个人随笔和感悟，这个实验没有CSAPP的实验那么难，很多其实是比较好想到的，只是C语言可能不是很熟，导致很多操作很难想到。

----


# NEMU随笔

## PA1

### 任务1

利用union共享数据的特点就行，但是比较难想
```c
union {
    union {
      uint32_t _32;
      uint16_t _16;
      uint8_t _8[2];
    } gpr[8];

    struct {
      uint32_t eax, ecx, edx, ebx, esp, ebp, esi, edi;
    };
  };
```


### 任务2

利用strtok读取参数，利用sscanf将字符串转换为数字，单步执行调用cpu_exec(n)
，打印寄存器利用循环打印cpu变量的成员变量，扫描内存调用swaddr_read读取



```c
static int cmd_si(char* args) {
  char* arg = strtok(NULL, " ");
  uint32_t n;
  if (arg != NULL) {
    if (arg <= 0)
      return -1;
    sscanf(arg, "%u", &n);
    cpu_exec(n);
    return 0;
  } else {
    cpu_exec(1);
    return 0;
  }
}
```

### 任务3

写正则表达式之后利用switch语句匹配并记录到tokens数组里，每记录一次nr_token加一，其中数字单独记录str变量，使用Assert来断言



### 任务4

使用递归求解，每次先检查左右是否有空格，如果有递归求解没有空格的表达式，对于多个括号的表达式，利用栈的思想使用一个常量通过记录左括号与右括号的差值来判断是否在括号里，实现方式是如果扫描到左括号则+1，右括号则-1，如果右括号多余则括号内表达式不合法。

### 选做任务1

单独加一个NEG type，如果“-”签名有运算符或者隔着空格有运算符，则标记为NEG，在记录数字的时候如果前面有NEG则记录为其相反数，并将NEG置为NOTYPE

### 任务5

按照运算符优先级记录下优先级最小的主运算符，由于`!`优先级最小所以每次先判断第一个token是否为取反，如果是递归对剩余expr取反

### 选做任务2

和前面取反的操作类似，再使用swaddr_read即可




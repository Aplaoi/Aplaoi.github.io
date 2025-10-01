---
category: 博客
tag:
  - 操作系统
  - 学校

---
# NEMU随笔-PA1

写一点我在做Nemu实验的时候的个人随笔和感悟，这个实验没有CSAPP的实验那么难，很多其实是比较好想到的，只是C语言可能不是很熟以及没有中大型项目开发经历，导致很多操作很难想到。

----


## PA1

### 任务1

利用union共享数据的特点就行，但是比较难想，结构如下
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

这个任务主要是让你熟悉一下C语言和项目架构，熟悉了不难想到。技术上，利用`strtok`读取参数，利用`sscanf`将字符串转换为数字，单步执行调用`cpu_exec(n)`，打印寄存器利用循环打印cpu变量的成员变量，扫描内存调用`swaddr_read(addr,len)`读取，具体实现如下：



::: code-tabs#c

@tab 单步执行(si)
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

@tab 打印寄存器(info)
```c
static int cmd_info(char *args) {
  char *arg = strtok(NULL, " ");
  if (strcmp(arg, "r") == 0) {
    printf(
        "$eax\t(0x%08x)\n$ecx\t(0x%08x)\n$edx\t(0x%08x)\n$ebx\t(0x%08x)\n$"
        "esp\t(0x"
        "%08x)\n$ebp\t(0x%08x)"
        "\n$esi\t(0x%08x)\n"
        "$edi\t(0x%08x)\n$eip\t(0x%08x)\n$eflags\t(0x%08x)\n",
        cpu.eax, cpu.ecx, cpu.edx, cpu.ebx, cpu.esp, cpu.ebp, cpu.esi, cpu.edi,
        cpu.eip, cpu.eflags.val);
    return 0;
  } 
}
```

@tab 扫描内存(x)
```c
static int cmd_x(char *args) {
  char *arg = strtok(NULL, " ");
  if (arg == NULL) return -1;
  int n;
  sscanf(arg, "%d", &n);
  char *arg2 = strtok(NULL, "");
  if (arg2 == NULL) return -1;
  uint32_t addr;
  sscanf(arg2, "%d", &addr);
  int i = 0;
  while (i < n) {
    uint32_t data = swaddr_read(addr + i * 4, 4);
    printf("0x%08x ", data);
    ++i;
  }
  printf("\n");
  return 0;
}
```

:::

### 任务3

这个任务会难一点，首先需要对正则表达式比较熟悉，参考[正则表达式教程](https://www.runoob.com/regexp/regexp-metachar.html)。先按照格式补充正则表达式，如下
```c
// nemu/src/monitor/debug/expr.c
rules[]={
    {" +", NOTYPE},           // spaces
    {"\\+", '+'},             // plus
    {"\\-", '-'},             // minus
    {"\\*", '*'},             // multiply
    {"/", '/'},               // divide
    {"==", EQ},               // equal
    {"(0x)?[0-9a-f]+", NUM},  // number
    {"\\(", LEFT_BRACKET},
    {"\\)", RIGHT_BRACKET},
}
```

写完之后会进行匹配和记录，由于空格并不参与运算，因此我不在`token`数组记录空格，这个在后面处理负号等非常关键。匹配完对`nr_token`进行+1，匹配过程如下，其中数字需要单独记录`str`成员，其余默认即可：

```c
// nemu/src/monitor/debug/expr.c
if (rules[i].token_type == NOTYPE) {
  continue;
}
switch (rules[i].token_type) {
  case NUM:
    tokens[nr_token].type = NUM;
    Assert(substr_len < 31, "number is too long");
    strncpy(tokens[nr_token].str, substr_start, substr_len);
    tokens[nr_token].str[substr_len] = '\0';
    break;
  default:
    tokens[nr_token].type = rules[i].token_type;
    break;
}
```

这样简单的匹配和记录的任务就完成了


### 任务4

这个难度比较大，主要使用递归求解。按照指导书的说明，我们可以先写一个框架，如下：

```c
// nemu/src/monitor/debug/expr.c
uint32_t expr(char *e, bool *success) {
  if (!make_token(e)) {
    *success = false;
    return 0;
  }
  int p = 0, q = nr_token - 1;
  int i = 0;
  printf("\n");
  for (; i <= q; i++) {
    Log("tokens[%d]: type=%d, str=%s", i, tokens[i].type, tokens[i].str);
  }
  printf("\n");
  return eval(p, q, success);
}
```

然后再在ui里对相关指令进行完善，如下：

```c {8-10}
// nemu/src/monitor/debug/ui.c
static int cmd_x(char *args) {
  char *arg = strtok(NULL, " ");
  if (arg == NULL) return -1;
  int n;
  sscanf(arg, "%d", &n);
  char *arg2 = strtok(NULL, "");
  if (arg2 == NULL) return -1;
  uint32_t addr;
  bool success = true;
  addr = expr(arg2, &success);
  Assert(success, "Wrong expression");
  int i = 0;
  while (i < n) {
    uint32_t data = swaddr_read(addr + i * 4, 4);
    printf("0x%08x ", data);
    ++i;
  }
  printf("\n");
  return 0;
}
```

`eval(int p, int q, bool *success)`函数实现起来会有点复杂，先看简单的部分，p和q可以看作表达式的左右双指针，当`p > q`时直接报错就行，`p = q`时表示表达式是一个数字，返回这个数字，接着检查双指针是否为括号，如果是统一位移一个单位求解内部表达式，实现如下：

```c
if (p > q) {
  *success = false;
  printf("Bad expression\n");
  return 0;
} else if (p == q) {
  if (tokens[p].type == NUM) {
    uint32_t num;
    sscanf(tokens[p].str, "%i", &num);
    return num;
  } else if (tokens[p].type == REG) {
    return get_reg_val(tokens[p].str, success);
  } else {
    *success = false;
    printf(RED "lack number" RESET "\n");
    return 0;
  }
} else if (check_bracket(p, q) == true) {
  return eval(p + 1, q - 1, success);
}else{
  // ...
}
```

诶有人就要问了，那直接看p, q处是否为左右括号就行了，为什么还要一个函数？事实上，`check_bracket(p, q)`函数不仅仅判断是否为括号，还判断是否括号内的括号是否合法，比如说`(1+(2+4*9)/6))`这个表达式，如果只判断括号那肯定会返回`true`，然后一直扫然后扫到了一个单独的右括号，然后程序就懵逼了，狠狠的报错。自然只判断左右括号也可以得出正确的报错，但是会进行不必要的计算导致程序运行慢，提前判断括号内的合法性相当于进行简单的剪枝，可以提高程序的效率。

具体实现方式可以使用栈的思想，其本质是用一个变量记录括号的差值，如果扫描到左括号则+1，右括号则-1，如果右括号多余则括号内表达式不合法。实现如下：

```c
static bool check_bracket(int p, int q) {
  if (tokens[p].type != LEFT_BRACKET || tokens[q].type != RIGHT_BRACKET) {
    return false;
  }
  int count = 0;
  int i;
  for (i = p + 1; i < q; i++) {
    if (tokens[i].type == LEFT_BRACKET) {
      count++;
    } else if (tokens[i].type == RIGHT_BRACKET) {
      count--;
      if (count < 0) {
        return false;
      }
    }
  }
  return count == 0;
}
```

接下来问题是判断主要操作符，按照指导书写就行。这个我以后水课详细写一下（



### 选做任务1

单独加一个NEG type，如果“-”签名有运算符或者隔着空格有运算符，则标记为NEG，在记录数字的时候如果前面有NEG则记录为其相反数，并将NEG置为NOTYPE

### 任务5

按照运算符优先级记录下优先级最小的主运算符，由于`!`优先级最小所以每次先判断第一个token是否为取反，如果是递归对剩余expr取反

### 选做任务2

和前面取反的操作类似，再使用swaddr_read即可




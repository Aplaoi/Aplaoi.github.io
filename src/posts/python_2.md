---
category: 博客
tags: 
    - python
    - 语言基础
---
# Python 学习笔记（二） —— typing 类型提示和 dataclasses

在现代 Python 编程中，类型提示和静态类型检查变得越来越重要，在很多开源项目里都有使用，这可以方便 IDE 及时检查出错误。`typing` 模块提供了丰富的类型提示功能，可以帮助我们编写更清晰、更易维护的代码。本文将介绍 `typing` 模块的一些常用功能。

----
## 类型提示与 `typing` 模块

`typing` 模块是 Python 3.5 引入的，它提供了对类型提示的支持。类型提示可以帮助开发者更好地理解代码的意图，并且可以利用 IDE 来检测潜在的错误。在使用时，要尽可能对输入值类型宽松，对输出值类型严格，这样可以提高代码的灵活性和鲁棒性。

### 基本类型提示

在 `typing` 模块中，我们可以使用`List`、`Dict`、`Tuple` 等类型来表示复杂的数据结构。这些泛型类型可以指示容器中元素的类型。在 Python 3.9 及以上版本中，也可以直接使用内置的 `list`、`dict` 等类型。

```python
from typing import List, Dict, Tuple

def relu(values: List[float]) -> List[float]:
    return [max(0, v) for v in values]

def normalize(data: Dict[str, List[float]]) -> Dict[str, List[float]]:
    return {k: [v / sum(v) for v in values] for k, values in data.items()}

def fibonacci(nums: Tuple[int, int]) -> Tuple[int, int]:
    a, b = nums
    return b, a + b
```

### 可迭代对象
`typing` 模块还提供了 `Iterable` 类型，用于表示可以迭代的对象。它可以用于函数参数和返回值的类型提示。使用 `Iterable` 可以让函数接受任何可迭代的对象，如列表、元组、集合等，而不仅限于特定的容器类型。

```python
from typing import Iterable
import numpy as np

def softmax(values: Iterable[float]) -> float:
    exp_values = [math.exp(v) for v in values]
    return sum(exp_values) / len(exp_values)

if __name__ == "__main__":
    a = [1.0, 2.0, 3.0]
    b = (4.0, 5.0, 6.0)
    c = np.array([7.0, 8.0, 9.0])
    print(softmax(a))  # 使用列表
    print(softmax(b))  # 使用元组
    print(softmax(c))  # 使用 NumPy 数组
```

### 函数指针

`typing` 模块还提供了 `Callable` 类型，用于表示函数指针。它可以用于类型提示函数参数和返回值的类型。使用 `Callable` 可以让我们指定函数的参数类型和返回值类型。

```python
from typing import Callable

def func_logger(func: Callable[[int], int]) -> Callable[[int], int]:
    def wrapper(x: int) -> int:
        result = func(x)
        print(f"Function {func.__name__} called with argument {x}, result: {result}")
        return result
    return wrapper

@func_logger
def square(x: int) -> int:
    return x * x
```

### 泛型

`typing` 模块还支持泛型编程，可以定义通用的类型。使用 `TypeVar` 可以创建一个类型变量，然后在函数或类中使用这个类型变量来表示任意类型。

```python
from typing import TypeVar
T = TypeVar('T', bound=[int, str])

def add(x: T, y: T) -> T:
    return x + y

if __name__ == "__main__":
    print(add(1, 2))        # 整数相加，输出3
    print(add("Hello, ", "World!"))  # 字符串相加，输出"Hello, World!"
    # print(add("Hello, ", 123))  # 会报错，因为类型不匹配
```

### Literal 类型
`Literal` 类型用于表示特定的值，可以用于函数参数和返回值的类型，这样可以限制函数只能接受特定的值。

```python
from typing import Literal

def set_attention_type(type: Literal['decoder', 'encoder', 'encoder-decoder']) -> None:
    print(f"Attention type set to: {type}")
    # do something with the attention type

if __name__ == "__main__":
    set_attention_type('decoder')  # 有效
    # set_attention_type('decoder-only')  # 会报错，因为不是指定的值
```

### 未实现类型与 `Type` 类型
在出现需要使用尚未实现的类型时，可以使用字符串形式的类型提示来避免循环引用的问题，称为“前向引用”。`Type` 类型用于表示类本身，可以用于函数参数和返回值的类型提示。这个在定义工厂函数或类方法时非常有用。

```python
from typing import Type
from abc import ABC, abstractmethod

class Attention(ABC):
    @abstractmethod
    def get_impl_cls(self) -> Type['Attention']:
        raise NotImplementedError
        # 这里子类必须实现这个方法，返回类本身或者其子类，不可以返回实例

    @abstractmethod
    def build(self) -> 'AttentionMetadata':
        raise NotImplementedError
        # 这里子类必须实现这个方法，返回一个AttentionMetadata实例

class AttentionMetadata:
    pass

```

## dataclasses

`dataclasses` 模块是 Python 3.7 引入的，它提供了一种简洁的方式来定义类，并自动生成常用的方法，如 `__init__`、`__repr__` 和 `__eq__`。使用 `dataclass` 装饰器可以大大减少样板代码，使代码更简洁易读。

参数上有以下几个常用选项：
- `init`: 是否生成 `__init__` 方法，默认为 `True`。
- `repr`: 是否生成 `__repr__` 方法，默认为 `True`。
- `eq`: 是否生成 `__eq__` 方法，默认为 `True`。
- `frozen`: 是否生成不可变类（即属性不可修改），默认为 `False`。

```python
from dataclasses import dataclass
from typing import List

@dataclass(frozen=True)
class ModelConfig:
    input_size: int
    hidden_size: int
    output_size: int
    layers: List[int]
    dropout: float = 0.5  # 默认值为0.5

if __name__ == "__main__":
    config = ModelConfig(input_size=128, hidden_size=256, output_size=10, layers=[128, 256, 512])
    print(config)
    # config.input_size = 256  # 会报错，因为类是不可变的
```

## 总结
本文介绍了 Python 中的 `typing` 模块和 `dataclasses` 模块。这两个在现代 Python 编程中非常有用，可以帮助我们编写更清晰、更易维护的代码。通过类型提示，我们可以更好地理解代码的意图，并利用 IDE 来检测潜在的错误。通过使用 `dataclass`，我们可以减少样板代码，使代码更简洁易读。希望本文能帮助你更好地理解和使用这些特性喵~





---
category: 博客
tags: 
    - python
    - 语言基础
---

# Python 学习笔记（一） —— @property装饰器和抽象基类abc

写一点Python学习笔记，记录一些在开源项目里见到的一些用法和技巧。本文主要介绍`property`装饰器和抽象基类`abc`模块。通过这两个特性的使用，可以让代码更加简洁、易读和可维护。

----

## property

在Python中，`property`是一个内置的装饰器，用于将类的方法转换为属性。它允许你定义一个方法，并通过属性访问的方式来调用它，而不需要显式地调用方法。这种方式可以让代码更简洁、更易读，同时也可以实现对属性的控制和验证。

例如，假设我们有一个 Student 类，我们希望通过属性访问学生的姓名和年龄，同时对年龄进行验证，传统方法需要写一个 getter 和 setter 方法，如下所示：

```python
class Student:
    def __init__(self, name, age):
        self._name = name
        self._age = age

    def get_name(self):
        return self._name

    def get_age(self):
        return self._age

    def set_age(self, value):
        if value < 0:
            raise ValueError("Age cannot be negative")
        self._age = value

if __name__ == "__main__":
    student = Student("Alice", 20)
    print(student.get_name())  # 通过方法访问姓名
    print(student.get_age())   # 通过方法访问年龄
    student.set_age(25)        # 通过方法设置年龄
    print(student.get_age())

```
这种方式虽然可行，但显得冗长且不够直观。使用 `property` 装饰器可以简化代码，如下所示：

```python
class Student:
    def __init__(self, name, age):
        self._name = name
        self._age = age

    # 相当于 getter 方法
    @property
    def name(self):
        return self._name

    @property
    def age(self):
        return self._age

    # 相当于 setter 方法
    @age.setter
    def age(self, value):
        if value < 0:
            raise ValueError("Age cannot be negative")
        self._age = value
    
if __name__ == "__main__":
    student = Student("Alice", 20)
    print(student.name)  # 通过属性访问姓名
    print(student.age)   # 通过属性访问年龄

    student.age = 25     # 通过属性设置年龄
    print(student.age)

    try:
        student.age = -5  # 尝试设置负数年龄，会引发异常
    except ValueError as e:
        print(e)
```

在这个例子中，我们使用 `@property` 装饰器将 `name` 和 `age` 方法转换为属性。这样，我们可以通过 `student.name` 和 `student.age` 来访问它们，而不需要调用方法。同时，我们还定义了一个 `age` 的 setter 方法，用于设置年龄并进行验证。


## abc基类

在Python中，`abc`模块提供了抽象基类（Abstract Base Classes）的支持。抽象基类允许你定义接口，并强制子类实现这些接口的方法。这对于确保代码的一致性和可维护性非常有用。在使用时，需要 import `ABC` 和 `abstractmethod`。不可以实例化抽象基类，只能实例化其子类，否则会引发错误。

同样使用 Student 类为例，假设其有子类 Freshman 和 Graduate，我们希望确保所有子类都实现一个 `study` 方法。可以通过定义一个抽象基类来实现：

```python
from abc import ABC, abstractmethod
class Student(ABC):
    def __init__(self, name, age):
        self.name = name
        self.age = age

    @abstractmethod
    def study(self):
        pass
    
class Freshman(Student):
    def study(self):
        return f"{self.name} is studying as a freshman."

class Graduate(Student):
    def study(self):
        return f"{self.name} is studying as a graduate student."

if __name__ == "__main__":
    freshman = Freshman("Bob", 18)
    graduate = Graduate("Charlie", 24)
    print(freshman.study()) # 输出: Bob is studying as a freshman.
    print(graduate.study()) # 输出: Charlie is studying as a graduate student.

    # 尝试实例化抽象基类会引发错误
    try:
        student = Student("Dave", 20)  # 这将引发 TypeError
    except TypeError as e:
        print(e)
```

在这个例子中，`Student` 类是一个抽象基类，定义了一个抽象方法 `study`。`Freshman` 和 `Graduate` 类继承自 `Student` 并实现了 `study` 方法。尝试实例化 `Student` 类会引发 `TypeError`，因为它是一个抽象类。

## 总结

通过使用 `property` 装饰器和 `abc` 模块，我们可以让Python代码更加简洁、易读和可维护。`property` 允许我们通过属性访问类的方法，而 `abc` 提供了定义接口和强制子类实现的方法的能力。这些特性在实际开发中非常有用，值得深入学习和应用。
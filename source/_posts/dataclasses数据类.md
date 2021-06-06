---
title: dataclasses数据类
toc: true
mathjax: true
date: 2020-07-07 16:01:04
tags:
categories:
- Python
- 基础
---

# dataclasses
官方文档：https://docs.python.org/zh-cn/3/library/dataclasses.html
<!--more-->

## 概要
这个模块提供了一个装饰器和一些函数，用于自动添加生成的 special methods ，例如 `__init__()` 和 `__repr__()` 到用户定义的类。 它最初描述于PEP557。

```python
from dataclasses import dataclass

@dataclass
class InventoryItem:
    '''Class for keeping track of an item in inventory.'''
    name: str
    unit_price: float
    quantity_on_hand: int = 0

    def total_cost(self) -> float:
        return self.unit_price * self.quantity_on_hand
```

除其他事情外，将添加`__init__()`，其看起来像:

```python
def __init__(self, name: str, unit_price: float, quantity_on_hand: int=0):
    self.name = name
    self.unit_price = unit_price
    self.quantity_on_hand = quantity_on_hand
```
请注意，此方法会自动添加到类中：它不会在上面显示的 InventoryItem 定义中直接指定。

## 模块级装饰器、类和函数
1. `@dataclasses.dataclass(*, init=True, repr=True, eq=True, order=False, unsafe_hash=False, frozen=False)`
**dataclass() 的参数有：**
- `init`: 如果为真值（默认），将生成一个`__init__()`方法。
如果类已定义 `__init__()` ，则忽略此参数。

- `repr`：如果为真值（默认），将生成一个`__repr__()`方法。 生成的 repr 字符串将具有类名以及每个字段的名称和 repr ，按照它们在类中定义的顺序。不包括标记为从 repr 中排除的字段。 例如：InventoryItem(name='widget', unit_price=3.0, quantity_on_hand=10)。

如果类已定义`__repr__()`，则忽略此参数。

- `eq`：如果为true（默认值），将生成`__eq__()`方法。此方法将类作为其字段的元组按顺序比较。比较中的两个实例必须是相同的类型。

如果类已定义 `__eq__()` ，则忽略此参数。

- `order` ：如果为真值（默认为 False ），则 `__lt__()` 、` __le__()` 、 `__gt__()` 和 `__ge__()` 方法将生成。 这将类作为其字段的元组按顺序比较。比较中的两个实例必须是相同的类型。如果 order 为真值并且 eq 为假值 ，则引发 ValueError 。

如果类已经定义了 `__lt__()` 、` __le__()` 、 `__gt__()` 或者 `__ge__()` 中的任意一个，将引发 TypeError 。

- `unsafe_hash` ：如果为 False （默认值），则根据 eq 和 frozen 的设置方式生成 `__hash__()` 方法。

`__hash__()` 由内置的 hash() 使用，当对象被添加到散列集合（如字典和集合）时。有一个`__hash__()` 意味着类的实例是不可变的。可变性是一个复杂的属性，取决于程序员的意图， `__eq__()` 的存在性和行为，以及 dataclass() 装饰器中 eq 和 frozen 标志的值。

默认情况下， dataclass() 不会隐式添加 `__hash__()` 方法，除非这样做是安全的。 它也不会添加或更改现有的明确定义的 `__hash__()` 方法。 设置类属性 `__hash__ = None` 对 Python 具有特定含义，如 `__hash__()` 文档中所述。

如果 `__hash__()` 没有显式定义，或者它被设置为 None ，那么 dataclass() 可以 添加一个隐式 `__hash__()` 方法。虽然不推荐，但你可以强制 dataclass() 用 unsafe_hash=True 创建一个 `__hash__()` 方法。 如果你的类在逻辑上是不可变的但实际仍然可变，则可能就是这种情况。这是一个特殊的用例，应该仔细考虑。

以下是隐式创建 `__hash__()` 方法的规则。请注意，你不能在数据类中都使用显式的 `__hash__()` 方法并设置 unsafe_hash=True ；这将导致 TypeError 。

如果 eq 和 frozen 都是 true，默认情况下 dataclass() 将为你生成一个 `__hash__()` 方法。如果 eq 为 true 且 frozen 为 false ，则 `__hash__()` 将被设置为 None ，标记它不可用（因为它是可变的）。如果 eq 为 false ，则 `__hash__()` 将保持不变，这意味着将使用超类的 `__hash__()` 方法（如果超类是 object ，这意味着它将回到基于id的hash）。

- frozen: 如为真值 (默认值为 False)，则对字段赋值将会产生异常。 这模拟了只读的冻结实例。 如果在类中定义了 `__setattr__()` 或` __delattr__()` 则将会引发 TypeError。 参见下文的讨论。

2. `dataclasses.field(*, default=MISSING, default_factory=MISSING, repr=True, hash=None, init=True, compare=True, metadata=None)`

field() 参数有：

- default ：如果提供，这将是该字段的默认值。这是必需的，因为 field() 调用本身会替换一般的默认值。

- default_factory ：如果提供，它必须是一个零参数可调用对象，当该字段需要一个默认值时，它将被调用。除了其他目的之外，这可以用于指定具有可变默认值的字段，如下所述。 同时指定 default 和 default_factory 将产生错误。

- init ：如果为true（默认值），则该字段作为参数包含在生成的 `__init__()` 方法中。

- repr ：如果为true（默认值），则该字段包含在生成的 `__repr__()` 方法返回的字符串中。

- compare ：如果为true（默认值），则该字段包含在生成的相等性和比较方法中（ `__eq__()` ， `__gt__()` 等等）。

- hash ：这可以是布尔值或 None 。如果为true，则此字段包含在生成的 `__hash__()` 方法中。如果为 None （默认值），请使用 compare 的值，这通常是预期的行为。如果字段用于比较，则应在 hash 中考虑该字段。不鼓励将此值设置为 None 以外的任何值。

设置 hash=False 但 compare=True 的一个可能原因是，如果一个计算 hash 的代价很高的字段是检验等价性需要的，但还有其他字段可以计算类型的 hash 。 即使从 hash 中排除某个字段，它仍将用于比较。

- metadata ：这可以是映射或 None 。 None 被视为一个空的字典。这个值包含在 MappingProxyType() 中，使其成为只读，并暴露在 Field 对象上。数据类根本不使用它，它是作为第三方扩展机制提供的。多个第三方可以各自拥有自己的键值，以用作元数据中的命名空间。


## 初始化后处理
```python
@dataclass
class C:
    a: float
    b: float
    c: float = field(init=False)

    def __post_init__(self):
        self.c = self.a + self.b
```


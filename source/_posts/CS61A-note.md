---
title: CS61A-note
abbrlink: 83c24a0e
date: 2024-02-24 17:12:39
tags: 知识
---

## 高阶函数

### 抽象
本质是更上一级的抽象  
函数可以作为函数的参数,不能作为控制结构的  
关键在于抽象出概念  
函数所在的帧是直接父级  

eg1:

```py
def average(x, y):
    return (x + y)/2

def improve(update, close, guess=1):
    while not close(guess):
        guess = update(guess)
    return guess

def approx_eq(x, y, tolerance=1e-3):
    return abs(x - y) < tolerance

def sqrt(a):
    def sqrt_update(x):
        return average(x, a/x)
    def sqrt_close(x):
        return approx_eq(x * x, a)
    return improve(sqrt_update, sqrt_close)

result = sqrt(256)
```

函数设计是一种思想，对问题从顶层到底层一步步抽象，，数学中将特殊问题一般化，而模板就是面向一般化的“步骤本身”

为了能使用抽象模板，一个技巧是以函数为参数可以得到需求形式的函数
```py
>>> def compose1(f, g):
        def h(x):
            return f(g(x))
        return h
>>> add_one_and_square = compose1(square, successor)
>>> add_one_and_square(12)
169
```

### lamda

也可以使用lamda

```py
>>> def compose1(f,g):
        return lambda x: f(g(x))
```

或
```py
>>> compose1 = lambda f,g: lambda x: f(g(x))
```

lamda：
```py
     lambda            x            :          f(g(x))
"A function that    takes x    and returns     f(g(x))"
```

### 装饰器

思想：等到给参数的时候才进行进一步调用，所以要用到高阶

用法：
```py
>>> def trace1(fn):
        def wrapped(x):
            print('-> ', fn, '(', x, ')')
            return fn(x)
        return wrapped
>>> @trace1
    def triple(x):
        return 3 * x
>>> triple(12)
->  <function triple at 0x102a39848> ( 12 )
36
```

本质：
```py
>>> def triple(x):
        return 3 * x
>>> triple = trace1(triple)
```

装饰器接受一个函数为参数，返回的本质上也是这个函数

使用装饰函数，以目标函数作为参数，进行指定的**处理**(装饰)

可以多重装饰，只要返回值是原函数，装饰只是增加副作用
```py
>>> def count(f):           #这种格式的，经过count和memo的修饰，就会有父帧的call_count属性，以及一些和属性相关的功能，反正不修改函数本身
        def counted(n):
            counted.call_count += 1
            return f(n)
        counted.call_count = 0
        return counted
>>> def memo(f):
        cache = {}
        def memorized(n):
            if n not in cache:
                cache[n] = f(n)
            return cache[n]
        return memorized
>>> counted_fib = count(fib)
>>> fib =memo(count_fib)
>>> fib(19)
4181
>>> counted_fib.call_count
20
>>> fib(34)
5702887
>>> counted_fib.call_count
35
```

函数式编程思想：    
所谓函数就是接受对象作为参数，并返回对象的一个盒子，这样看来函数就是一种自己定义的数学运算符，即像是自己定义了一种新的计算，也像是一个数学公式    
对象可以是类、基本数据、函数，而且函数本身也可以作为对象，这就意味着最小操作类型可以看作是对象，而所谓的类、函数等也不过是一种新的数据类型组织方式  
所以编写程序的过程就是自己构造一个数学公式来产生目标输出的功能  
所以关键是，要知道要接受一个怎么样的对象，返回一个什么样功能的对象  


再例如Web中：
```py
def PlatformAuthenticated(auth_level):      #将验证handler的函数作为修饰器，通过类设置一些外部属性，调用get方法时自动调用call，自动验证
    class StrictPlatformAuth:
        def __init__(self, handler):
            self.handler = handler
            
        def __call__(self, request, response):
            auth = request.authorization(auth_level)
            if not auth or not check_auth(auth.username, auth.password):
                raise Exception("BlahBlahBlah")
            self.handler(request, response)

    return StrictPlatformAuth

class WebHandler:
    @PlatformAuthenticated(ADMIN)
    def get(self, request, response):
        do_something()

class RobotWebHandler:
    @PlatformAuthenticated(ROBOT)
    def get(self, request, response):
        do_something()
```

### 柯里化

多参数函数和指定参数函数之间的转换：
```py
>>> def curry2(f):
        """返回给定的双参数函数的柯里化版本"""
        def g(x):
            def h(y):
                return f(x, y)
            return h
        return g
>>> def uncurry2(g):
        """返回给定的柯里化函数的双参数版本"""
        def f(x, y):
            return g(x)(y)
        return f
>>> pow_curried = curry2(pow)
>>> pow_curried(2)(5)
32
>>> map_to_range(0, 10, pow_curried(2))
1
2
4
8
16
32
64
128
256
512
```

### 递归

递归思路：自顶向下，只关心下一层，不关心全部(return)   
只有最后一层才有细节(if n==1)

要清楚**迭代对象**和**迭代关系**
以函数为整体看待
对应数学归纳法，归纳的时候考虑的是过程中的一般情况以及临界条件，不应该自动带入考虑程序怎么跑
只要是序列间有关系的问题都可以用递归解决

hw03的count_change

hw04的balance,归纳法：假设这个函数已经成立，可以用这个函数本身来递推的实现功能，再验证边界成立
    preorder: 列表本身可以被迭代，所以可以写二重循环

## 数据抽象

### 序列

序列解构：将序列元素的子序列元素映射到指定的name上

range是序列

有.count和.index行为
count行为匹配的是非重叠的字符串

字符串本质也是一种序列，换行符也会被认为是字符串元素
字符串的in匹配的是字符串

接口：使用map，reduce，filter来实现对数据的一系列组件操作，  
也可以使用生成器表达式：
```py
>>> def acronym(name):
        return tuple(w[0] for w in name.split() if iscap(w))
>>> def sum_even_fibs(n):
        return sum(fib(k) for k in range(1, n+1) if fib(k) % 2 == 0)
```

使用构造器和选择器对抽象数据进行操作

## 2.4 可变数据

对象是一种数据类型，其中包含函数(操作数据的行为)以及数据，不仅是值，也是一个过程  
**python万物皆对象**

#### 为什么要设计类？ 
将数据和行为为基本元素抽象成一个新的数据类型，这种数据类型更加真实地映射了现实世界的对象，从功能上看更便于实现过程和行为的构建，从结构上看提供了强大的数据和行为之间的交互和传递能力(继承，多态)，以及极大地方便了对数据的处理和对行为的调用

### 理解抽象

抽象的最终目的是产生更加通用的模板来简化操作，更通用、更简单地实现更多的任务  
抽象必然是一层一层的，从计算机硬件电路到硬件模块的封装，从硬件到终端的交互，从基本数据类型到复合数据类型(OOP)，从符合数据类型到一系列数据和行为构成的集合体(包和库)，再到软件的实现     
实现功能的抽象模块之间应该是尽量独立的，不互相影响的，否则负责的交互关系会使底层出现问题的时候产生不可预测的对高级抽象的影响，以及加大了抽象修改的难度      
所以从底层一步步向上理解学习吧，才能有更扎实的基础

### 列表

使用变异(mutating)操作会修改可变对象  
对于可变对象，关联名称不是创建副本，而是绑定到同一个对象上：
```py
>>> chinese = ['apple', 'pear', 'monkey']
>>> suits = chinese
>>> suits.pop()
'monkey'
>>> chinese
['apple', 'pear']
```

创建副本的方法：
```py
>>> nest = list(suits)
```

判断两个对象是否是同一个，用`is`或`is not`，判读是否相等用`==`

### 元组

不可变对象

### 字典

描述性检索的数据抽象

dict()可以将键值对列表转化为字典：
```py
>>> dict([(3, 9), (4, 16), (5, 25)])
{3: 9, 4: 16, 5: 25}
```

可以用推导式语法创建：
```py
>>> {x: x*x for x in range(3,6)}
{3: 9, 4: 16, 5: 25}
```

方法：
- get(key,default_value)    return the value if key exist or default value
- items()                   return key and value that is iterable
- values()                  return iterable values
- keys()                    ................keys

### 局部状态

对函数可变数据的构建用的是nonlocal —— 用于将数据置为非局部状态  
**python限制一个名称对应的实例必须绑定在同一个局部帧中，所以不存在两个帧使用同一个名称的情况**      
查找非局部帧的值不需要非局部语句，修改需要  
python预处理名称来限制绑定的帧只能在局部帧，所以不允许提前引用变量  

#### 好处

非局部赋值实际上是将balance与with_draw单独绑定，便于各个函数独立分治自己的局部状态
创建一个实例就对应创建一个绑定  
所以引出了一个问题：
```py
1   def make_withdraw(balance):
2	    def withdraw(amount):
3	        nonlocal balance
4	        if amount > balance:
5	            return 'Insufficient funds'
6	        balance = balance - amount
7	        return balance
8	    return withdraw
9	
10	wd = make_withdraw(12)
11	wd2 = wd
12	wd2(1)
13	wd(1)
```
两个相同的名称连接到一个绑定上，所处的帧并没有改变，**只有函数调用才会创建新帧**

#### 实现列表和字典结构

使用非局部状态函数可以实现可变数据类型，因为非局部帧相当于提供使数据可变的能力，既不帧提供操作数据的方法   
用函数来作为可变数据结构
```py
>>> def mutable_link():
        """返回一个可变链表的函数"""
        contents = empty
        def dispatch(message, value=None):
            nonlocal contents
            if message == 'len':
                return len_link(contents)
            elif message == 'getitem':
                return getitem_link(contents, value)
            elif message == 'push_first':
                contents = link(value, contents)
            elif message == 'pop_first':
                f = first(contents)
                contents = rest(contents)
                return f
            elif message == 'str':
                return join_link(contents, ", ")
        return dispatch

>>> def to_mutable_link(source):
        """返回一个与原列表相同内容的函数列表"""
        s = mutable_link()
        for element in reversed(source):
            s('push_first', element)
        return s
```
可以用两种方法来实现对数据类型的变异操作：
- 将方法放到一个局部帧中
- 将方法放到不同局部帧中，提供一个消息接口来调用(消息传递)

消息传递的实现可以模拟一个send_message的过程

```py
>>> def dictionary():
        """返回一个字典的函数实现"""
        records = []
        def getitem(key):
            matches = [r for r in records if r[0] == key]
            if len(matches) == 1:
                key, value = matches[0]
                return value
        def setitem(key, value):
            nonlocal records
            non_matches = [r for r in records if r[0] != key]
            records = non_matches + [[key, value]]
        def dispatch(message, key=None, value=None):
            if message == 'getitem':
                return getitem(key)
            elif message == 'setitem':
                setitem(key, value)
        return dispatch
```

### 声明式编程实践：约束传递(Propagating Constraints)

声明式编程的特性是声明要解决的问题的结构  
编程中的表达式的计算都是单项的，要实现多项的解方程求值要用变量关系的约束器(Constrainter)和提醒其他约束器的连接器(connector)构成网络  
使用消息传递以及对消息进行响应的数据结构(类)来构建网络

```py
>>> connector ['set_val'](source, value)  """表示 source 在请求连接器将当前值设为 value"""
>>> connector ['has_val']()  """返回连接器是否已经具有值"""
>>> connector ['val']  """是连接器的当前值"""
>>> connector ['forget'](source)  """告诉连接器 source 请求遗忘它的值"""
>>> connector ['connect'](source)  """告诉连接器参与新的约束，即 source"""

>>> constraint['new_val']()  """表示与约束相连的某个连接器具有新的值。"""
>>> constraint['forget']()  """表示与约束相连的某个连接器遗忘了值。"""

>>> def converter(c, f):
        """用约束条件连接 c 到 f ，将摄氏度转换为华氏度."""
        u, v, w, x, y = [connector() for _ in range(5)]
        multiplier(c, w, u)
        multiplier(v, x, u)
        adder(v, y, f)
        constant(w, 9)
        constant(x, 5)
        constant(y, 32)

>>> from operator import add, sub
>>> def adder(a, b, c):
        """约束a+b=c"""
        return make_ternary_constraint(a, b, c, add, sub, sub)

>>> def constant(connector, value):
        """常量赋值."""
        constraint = {}
        connector['set_val'](constraint, value)
        return constraint

>>> def make_ternary_constraint(a, b, c, ab, ca, cb):       #不存储连接的连接器，只提供功能
        """约束ab(a,b)=c，ca(c,a)=b，cb(c,b)=a。"""
        def new_value():
            av, bv, cv = [connector['has_val']() for connector in (a, b, c)]
            if av and bv:
                c['set_val'](constraint, ab(a['val'], b['val']))
            elif av and cv:
                b['set_val'](constraint, ca(c['val'], a['val']))
            elif bv and cv:
                a['set_val'](constraint, cb(c['val'], b['val']))
        def forget_value():
            for connector in (a, b, c):
                connector['forget'](constraint)
        constraint = {'new_val': new_value, 'forget': forget_value}
        for connector in (a, b, c):
            connector['connect'](constraint)
        return constraint

>>> def connector(name=None):
        """限制条件之间的连接器."""
        informant = None
        constraints = []
        def set_value(source, value):
            nonlocal informant
            val = connector['val']
            if val is None:
                informant, connector['val'] = source, value
                if name is not None:
                    print(name, '=', value)
                inform_all_except(source, 'new_val', constraints)
            else:
                if val != value:
                    print('Contradiction detected:', val, 'vs', value)
        def forget_value(source):
            nonlocal informant
            if informant == source:
                informant, connector['val'] = None, None
                if name is not None:
                    print(name, 'is forgotten')
                inform_all_except(source, 'forget', constraints)
        connector = {'val': None,
                     'set_val': set_value,
                     'forget': forget_value,
                     'has_val': lambda: connector['val'] is not None,
                     'connect': lambda source: constraints.append(source)}
        return connector

>>> def inform_all_except(source, message, constraints):
        """告知信息除了source外的所有约束条件，。"""
        for c in constraints:
            if c != source:
                c[message]()
```

## OOP

类的构造器：`__init__`  
`self`绑定对象，调用时实例会绑定到self中  
赋值将对象绑定到新名称不会创建新对象:  
`c = a`  
作为类的属性，方法只是一个函数，但作为实例的属性，它是一个绑定方法：
```py
>>> type(Account.deposit)
<class 'Function'>
>>> type(spock_account.deposit)
<class 'method'>
```

我们可以通过两种方式调用 deposit ：作为函数和作为绑定方法。在前一种情况下，我们必须显式地为 self 参数提供一个参数。在后一种情况下， self 参数会自动绑定。
```py
>>> Account.deposit(spock_account, 1001)	# 函数 deposit 接受两个参数
1011
>>> spock_account.deposit(1000) 			# 方法 deposit 接受一个参数
2011
```

如果属性名称以下划线开头，则只能在类中本地访问，不能被用户访问

**类属性**是所有实例共享的静态变量

对象的点表达式，如果是实例，存在则调用，不存在则创建，但是不会访问类属性，只有对类的调点表达式才会修改类属性  
同名时，本地的实例属性是优先于类属性的

baseclass, superclass, parentclass  
subclass, childclass

接受参数作为父类，重写属性或者方法，注意父类中对实例属性的调用最好都用self.attr，方便作继承

尽量不要使用Account.method，使用实例的方法更具普遍性

**重写**：方法可以重写，注意最后好使用self.attribution来在类中定义属性或方法，因为在该类的子类中继承此方法，要使用的是子类的新值


**多继承**：
```py
>>> class AsSeenOnTVAccount(CheckingAccount, SavingsAccount):
        def __init__(self, account_holder):
            self.holder = account_holder
            self.balance = 1           # 赠送的 1 $!
```
对于多继承，调用不同类的同一个名称的方法时，检索顺序为从左到右，从子到基    
查看解析对于类和属性的查询顺序：
```py
>>> [c.__name__ for c in AsSeenOnTVAccount.mro()]
['AsSeenOnTVAccount', 'CheckingAccount', 'SavingsAccount', 'Account', 'object']
```

### 使用函数来实现类

```py
>>> def make_instance(cls):
		"""Return a new object instance, which is a dispatch dictionary."""
		def get_value(name):
			if name in attributes:
				return attributes[name]
			else:
				value = cls['get'](name)
				return bind_method(value, instance)
		def set_value(name, value):
			attributes[name] = value
		attributes = {}
		instance = {'get': get_value, 'set': set_value}
		return instance

>>> def bind_method(value, instance):
		"""Return a bound method if is callable, or value otherwise."""
		if callable(value):
			def method(*args):
				return value(instance, *args)
			return method
		else:
			return value
```
使用响应消息的调度字典来实现，如果获取到的name在attribute中找不到，则进入bind_method来寻找以及绑定类和方法，找到了是函数的话，返回使用alue的函数，并且将instance作为self

### 专用方法

- `__repr__`在使用交互环境时自动调用  
- `__str__`在打印时自动调用  
- `__bool__`构造，可以判断逻辑时自动调用  
- `__len__`长度，使用len()时调用  
- `__getitem__`，获取元素时调用  
- `__call__`可以使类可以像高阶函数一样被传播调用  
```py
>>> class Adder(object):
		def __init__(self, n):
			self.n = n
		def __call__(self, k):
			return self.n + k
>>> add_three_obj = Adder(3)
>>> add_three_obj(4)
7
```

### 多重表示

接口是一组共享的属性名称，以及对它们的行为的规范  
从最高层抽象向下实现：数值类型——拥有的操作——具体属性定义
Complex实现了接口，保证了具体方法的属性的一致性，达成了命名规范   
@property修饰符可以将函数变为无参数，直接当属性值使用   
```py
>>> class Number:
		def __add__(self, other):
			return self.add(other)
		def __mul__(self, other):
			return self.mul(other)

>>> class Complex(Number):
		def add(self, other):
			return ComplexRI(self.real + other.real, self.imag + other.imag)
		def mul(self, other):
			magnitude = self.magnitude * other.magnitude
			return ComplexMA(magnitude, self.angle + other.angle)

>>> from math import atan2
>>> class ComplexRI(Complex):
		def __init__(self, real, imag):
			self.real = real
			self.imag = imag
		@property
		def magnitude(self):
			return (self.real ** 2 + self.imag ** 2) ** 0.5
		@property
		def angle(self):
			return atan2(self.imag, self.real)
		def __repr__(self):
			return 'ComplexRI({0:g}, {1:g})'.format(self.real, self.imag)

>>> from math import sin, cos, pi
>>> class ComplexMA(Complex):
		def __init__(self, magnitude, angle):
			self.magnitude = magnitude
			self.angle = angle
        	@property
		def real(self):
			return self.magnitude * cos(self.angle)
		@property
		def imag(self):
			return self.magnitude * sin(self.angle)
		def __repr__(self):
			return 'ComplexMA({0:g}, {1:g} * pi)'.format(self.magnitude, self.angle/pi)
```

优势：   
类似complex的接口类是可以开发添加的

### 泛型表示

#### 类型派发

依据受到的参数的类型(类的种类)来派发响应，保证数据跨类型的时候仍然可以被正确地对待

内置的函数 isinstance 接受一个对象或一个类。如果对象的类是所给的类或者继承自所给的类，它会返回一个真值  
```py
>>> c = ComplexRI(1, 1)
>>> isinstance(c, ComplexRI)
True
>>> isinstance(c, Complex)
True
>>> isinstance(c, ComplexMA)
False
```

基于不同类型的接口类一个type_tag，来在进行操作的时候检查type_tag来进行类型派发
```py
>>> class Number:
		def __add__(self, other):
			if self.type_tag == other.type_tag:
				return self.add(other)
			elif (self.type_tag, other.type_tag) in self.adders:
				return self.cross_apply(other, self.adders)
		def __mul__(self, other):
			if self.type_tag == other.type_tag:
				return self.mul(other)
			elif (self.type_tag, other.type_tag) in self.multipliers:
				return self.cross_apply(other, self.multipliers)
		def cross_apply(self, other, cross_fns):
			cross_fn = cross_fns[(self.type_tag, other.type_tag)]
			return cross_fn(self, other)
		adders = {("com", "rat"): add_complex_and_rational,
					("rat", "com"): add_rational_and_complex}
		multipliers = {("com", "rat"): mul_complex_and_rational,
						("rat", "com"): mul_rational_and_complex}
```

### 强制类型转换

将一种数据类型和另一种数据类型在符合某种条件时可以实施强制类型转换来简化操作
```py
>>> class Number:
		def __add__(self, other):
			x, y = self.coerce(other)
			return x.add(y)
		def __mul__(self, other):
			x, y = self.coerce(other)
			return x.mul(y)
		def coerce(self, other):
			if self.type_tag == other.type_tag:
				return self, other
			elif (self.type_tag, other.type_tag) in self.coercions:
				return (self.coerce_to(other.type_tag), other)
			elif (other.type_tag, self.type_tag) in self.coercions:
				return (self, other.coerce_to(self.type_tag))
		def coerce_to(self, other_tag):
			coercion_fn = self.coercions[(self.type_tag, other_tag)]
			return coercion_fn(self)
        def rational_to_complex(r):
		    return ComplexRI(r.numer/r.denom, 0)
		coercions = {('rat', 'com'): rational_to_complex}
        
```

## 效率

可以用记忆化来提前存储值：
```py
>>> def memo(f):
        cache = {}
        def memorized(n):
            if n not in cache:
                cache[n] = f(n)
            return cache[n]
        return memorized
```

```py
>>> def count(f):
        def counted(n):
            counted.call_count += 1
            return f(n)
        counted.call_count = 0
        return counted
```
这种方式可以构造类似于类的属性值，作为外部变量来使用
```py
>>> fib = count(fib)
>>> fib(19)
4181
>>> fib.call_count
10946
```

阶乘的时间复杂度 $\Theta(\log_{2}{n})$ 的优化  
$$
b^{n}=\begin{cases}(b^{\frac{n}{2}})^{2}\qquad\qquad 如果n是偶数\\ b\cdot{b^{n-1}}\qquad\quad 如果n是奇数\end{cases}
$$
```py
>>> def square(x):
        return x * x

>>> def fast_exp(b, n):
        if n == 0:
            return 1
        if n % 2 == 0:
            return square(fast_exp(b, n // 2))
        else:
            return b * fast_exp(b, n - 1)

>>> fast_exp(2, 100)
1267650600228229401496703205376
```

python中的`if in`是有n的时间复杂度的

feibonaqi数列最接近黄金比例的n-2次方除以根号5

## 递归对象

可以使用类方法的递归来构建诸如**链表、树、集合**的数据结构，以及**add、repr**等方法，和很多操作，其本质和递归函数是一样的，可以用`Link.__add__ = extend`来直接进行专用方法赋值，具体方法如下：

链表
```py
>>> class Link:
        """一个链表"""
        empty = ()
        def __init__(self, first, rest=()):
            assert rest == Link.empty or isinstance(rest, Link)
            self.first = first
            self.rest = rest
        def __getitem__(self, i):
            if i == 0:
                return self.first
            else:
                return self.rest[i-1]
        def __len__(self):
            return 1 + len(self.rest)
>>> s = Link(3, Link(4, Link(5)))
>>> len(s)
3
>>> s[1]
4

>>> def link_expression(s):
        """返回一个可以计算得到 s 的字符串表达式。"""
        if s.rest is Link.empty:
            rest = ''
        else:
            rest = ', ' + link_expression(s.rest)
        return 'Link({0}{1})'.format(s.first, rest)
>>> link_expression(s)
'Link(3, Link(4, Link(5)))'

>>> Link.__repr__ = link_expression
>>> s
Link(3, Link(4, Link(5)))

>>> s_first = Link(s, Link(6))
>>> s_first
Link(Link(3, Link(4, Link(5))), Link(6))

>>> len(s_first)
2
>>> len(s_first[0])
3
>>> s_first[0][2]
5

>>> def extend_link(s, t):
        if s is Link.empty:
            return t
        else:
            return Link(s.first, extend_link(s.rest, t))
>>> extend_link(s, s)
Link(3, Link(4, Link(5, Link(3, Link(4, Link(5))))))
>>> Link.__add__ = extend_link
>>> s + s
Link(3, Link(4, Link(5, Link(3, Link(4, Link(5))))))

>>> def map_link(f, s):
        if s is Link.empty:
            return s
        else:
            return Link(f(s.first), map_link(f, s.rest))
>>> map_link(square, s)
Link(9, Link(16, Link(25)))

>>> def filter_link(f, s):
        if s is Link.empty:
            return s
        else:
            filtered = filter_link(f, s.rest)
            if f(s.first):
                return Link(s.first, filtered)
            else:
                return filtered
>>> odd = lambda x: x % 2 == 1
>>> map_link(square, filter_link(odd, s))
Link(9, Link(25))
>>> [square(x) for x in [3, 4, 5] if odd(x)]
[9, 25]

>>> def join_link(s, separator):
        if s is Link.empty:
            return ""
        elif s.rest is Link.empty:
            return str(s.first)
        else:
            return str(s.first) + separator + join_link(s.rest, separator)
>>> join_link(s, ", ")
'3, 4, 5'

>>> def partitions(n, m):
        """Return a linked list of partitions of n using parts of up to m.
        Each partition is represented as a linked list.
        """
        if n == 0:
            return Link(Link.empty) # A list containing the empty partition
        elif n < 0 or m == 0:
            return Link.empty
        else:
            using_m = partitions(n-m, m)
            with_m = map_link(lambda s: Link(m, s), using_m)
            without_m = partitions(n, m-1)
            return with_m + without_m

>>> def print_partitions(n, m):
        lists = partitions(n, m)
        strings = map_link(lambda s: join_link(s, " + "), lists)
        print(join_link(strings, "\n"))
>>> print_partitions(6, 4)
4 + 2
4 + 1 + 1
3 + 3
3 + 2 + 1
3 + 1 + 1 + 1
2 + 2 + 2
2 + 2 + 1 + 1
2 + 1 + 1 + 1 + 1
1 + 1 + 1 + 1 + 1 + 1
```

树：
```py
>>> class Tree:
        def __init__(self, label, branches=()):
            self.label = label
            for branch in branches:
                assert isinstance(branch, Tree)
            self.branches = branches
        def __repr__(self):
            if self.branches:
                return 'Tree({0}, {1})'.format(self.label, repr(self.branches))
            else:
                return 'Tree({0})'.format(repr(self.label))
        def is_leaf(self):
            return not self.branches

>>> def fib_tree(n):
        if n == 1:
            return Tree(0)
        elif n == 2:
            return Tree(1)
        else:
            left = fib_tree(n-2)
            right = fib_tree(n-1)
            return Tree(left.label + right.label, (left, right))
>>> fib_tree(5)
Tree(3, (Tree(1, (Tree(0), Tree(1))), Tree(2, (Tree(1), Tree(1, (Tree(0), Tree(1)))))))

>>> def sum_labels(t):
        """对树的 label 求和，可能得到 None。"""
        return t.label + sum([sum_labels(b) for b in t.branches])
>>> sum_labels(fib_tree(5))
10

>>> fib_tree = memo(fib_tree)
>>> big_fib_tree = fib_tree(35)
>>> big_fib_tree.label
5702887
>>> big_fib_tree.branches[0] is big_fib_tree.branches[1].branches[1]
True
>>> sum_labels = memo(sum_labels)
>>> sum_labels(big_fib_tree)
142587180
```

集合：
```py
>>> 3 in s
True
>>> len(s)
4
>>> s.union({1, 5})
{1, 2, 3, 4, 5}
>>> s.intersection({6, 5, 4, 3})
{3, 4}

>>> def empty(s):
        return s is Link.empty
>>> def set_contains(s, v):
        """当且仅当 set s 包含 v 时返回 True。"""
        if empty(s):
            return False
        elif s.first == v:
            return True
        else:
            return set_contains(s.rest, v)
>>> s = Link(4, Link(1, Link(5)))
>>> set_contains(s, 2)
False
>>> set_contains(s, 5)
True

>>> def adjoin_set(s, v):
        """返回一个包含 s 的所有元素和元素 v 的所有元素的集合。"""
        if set_contains(s, v):
            return s
        else:
            return Link(v, s)
>>> t = adjoin_set(s, 2)
>>> t
Link(2, Link(4, Link(1, Link(5))))

>>> def union_set(set1, set2):
        """返回一个集合，包含 set1 和 set2 中的所有元素。"""
        set1_not_set2 = keep_if_link(set1, lambda v: not set_contains(set2, v))
        return extend_link(set1_not_set2, set2)
>>> union_set(t, s)
Link(2, Link(4, Link(1, Link(5))))

>>> def set_contains(s, v):
        if empty(s) or s.first > v:
            return False
        elif s.first == v:
            return True
        else:
            return set_contains(s.rest, v)
>>> u = Link(1, Link(4, Link(5)))
>>> set_contains(u, 0)
False
>>> set_contains(u, 4)
True

>>> def intersect_set(set1, set2):
        if empty(set1) or empty(set2):
            return Link.empty
        else:
            e1, e2 = set1.first, set2.first
            if e1 == e2:
                return Link(e1, intersect_set(set1.rest, set2.rest))
            elif e1 < e2:
                return intersect_set(set1.rest, set2)
            elif e2 < e1:
                return intersect_set(set1, set2.rest)
>>> intersect_set(s, s.rest)
Link(4, Link(5))

>>> def set_contains(s, v):
        if s is None:
            return False
        elif s.entry == v:
            return True
        elif s.entry < v:
            return set_contains(s.right, v)
        elif s.entry > v:
            return set_contains(s.left, v)

>>> def adjoin_set(s, v):
        if s is None:
            return Tree(v)
        elif s.entry == v:
            return s
        elif s.entry < v:
            return Tree(s.entry, s.left, adjoin_set(s.right, v))
        elif s.entry > v:
            return Tree(s.entry, adjoin_set(s.left, v), s.right)
>>> adjoin_set(adjoin_set(adjoin_set(None, 2), 3), 1)
Tree(2, Tree(1), Tree(3))


```

## 迭代器

为了不预先存储值，但是能动态生成所要的数据或者操作，使用惰态计算，给予时延  
iterator使用next返回下一个值，当没有值时返回StopIteration   
迭代器用iter构造可迭代对象为迭代器，iter本身也可以被使用iter方法，作用是绑定名称        

map 函数是惰性的：调用它时并不会执行计算，直到返回的迭代器被 next 调用      
相反，会创建一个迭代器对象，如果使用 next 查询， 该迭代器对象可以返回结果。我们可以在下面的示例中观察到这一事实，其中对 print 的调用被 延迟，直到从 doubled 迭代器请求相应的元素为止。
```py
>>> def double_and_print(x):
        print('***', x, '=>', 2*x, '***')
        return 2*x
>>> s = range(3, 7)
>>> doubled = map(double_and_print, s)  # double_and_print 未被调用
>>> next(doubled)                       # double_and_print 调用一次
*** 3 => 6 ***
6
>>> next(doubled)                       # double_and_print 再次调用
*** 4 => 8 ***
8
>>> list(doubled)                       # double_and_print 再次调用兩次
*** 5 => 10 ***                         # list() 会把剩余的值都计算出来并生成一个列表
*** 6 => 12 ***
[10, 12]
```
filter 函数返回一个迭代器， zip 和 reversed 函数也返回迭代器。  


for循环是调用了可迭代对象本身的__iter__方法，返回next值给绑定值，然后执行\<suite>主体部分   
构造for循环：
```py
>>> items = counts.__iter__()
>>> try:
        while True:
             item = items.__next__()
             print(item)
    except StopIteration:
        pass
1
2
3
```

### 生成器

生成器是一种迭代器，为了控制迭代函数的进度，使用生成器，关键词为yield，当函数内有yield关键词时，默认返回一个生成器，每次next返回下一个yield对应的值

### 可迭代接口
可以使类的__iter__方法返回一个迭代器，但是类本身是不变的
```py
>>> class Letters:
        def __init__(self, start='a', end='e'):
            self.start = start
            self.end = end
        def __iter__(self):
            return LetterIter(self.start, self.end)
```

自定义迭代器，next方法，使用next时调用：
```py
>>> class LetterIter:
        """依照 ASCII 码值顺序迭代字符的迭代器。"""
        def __init__(self, start='a', end='e'):
            self.next_letter = start
            self.end = end
        def __next__(self):
            if self.next_letter == self.end:
                raise StopIteration
            letter = self.next_letter
            self.next_letter = chr(ord(letter)+1)
            return letter
```
### 流：惰性递归

Stream是一种对递归使用惰性的方法，不会提前计算下一个递归结果是什么，除非手动调用它
```py
>>> s = Stream(1, lambda: Stream(2+3, lambda: Stream.empty))

>>> class Stream(object):
        """A lazily computed recursive list."""
        def __init__(self, first, compute_rest, empty=False):
            self.first = first
            self._compute_rest = compute_rest
            self.empty = empty
            self._rest = None
            self._computed = False
        @property
        def rest(self):
            """Return the rest of the stream, computing it if necessary."""
            assert not self.empty, 'Empty streams have no rest.'
            if not self._computed:
                self._rest = self._compute_rest()
                self._computed = True
            return self._rest
        def __repr__(self):
            if self.empty:
                return '<empty stream>'
            return 'Stream({0}, <compute_rest>)'.format(repr(self.first))
>>> Stream.empty = Stream(None, None, True)
```

可以修改定义map函数或者filter函数，使map功能可以给流的每个递归
```py
>>> def map_stream(fn, s):
        if s.empty:
            return s
        def compute_rest():
            return map_stream(fn, s.rest)
        return Stream(fn(s.first), compute_rest)
```
## Scheme

compiler编译器：编译为底层语言供机器识别    
interpreter解释器：将一种语言解释为另一种语言，python的解释器就是用c写的，因为解释器是按照每一行来读取输入的，所以解释器是行解释的  

对一个语言的解释：

- 词法解析(lexical analyzer)：使用分词器将语法分为标记(token)，之后传给语法解析器
- 语法解析(syntactic analyzer)：对标记使用可递归的数据结构来构架语法树
- 计算(calculate)：对语法树进行计算，计算面向的是递归的数据结构


## 语法杂记

- 定义函数参数给出后视作默认值，使用时可以不写

- 文档字符串，三个双引号，使用help(fun)查看

- 元组的加与乘返回结果是组合以后的元组，map函数可将函数应用于数据，reduce可以将函数作用于序列返回单值(归约)，tuple可以将数据类型变为元组

- python对变量的定义是非严格的，所以想要在函数内使用父帧的变量要进行声明，否则默认为局部帧的新变量  
声明方法有`nonlocal`(嵌套函数)和`global`(全局)

is和is not用于判断两个对像是否相同，这个相同指的是相同的内存地址

强制类型转换，类型不加括号

print('a'),打印：a

and运算时，从左到右演算表达式的值。0, '', [], {}, None在布尔表达式环境下为假，其他任何东西都为真，返回作后一个真值或假值
or返回最后一个真值或最后一个假值
not 10 返回False， not None 返回True   注意0是真值,但是not 0中0为假
1/0 or True 为Error，从左到右判断，遇到Error直接Error

元组元素不可修改

assert用法：
```py
assert type(sides) == int and sides >= 1, 'Illegal value for sides' 
```

return A if condition else 

.strip()方法消除字符串首尾空白字符

字典中的in是判断的key值

random.sample接受list和指定个数，返回列表   一定要注意函数的返回类型

列表和字符串[1:]没有的时候会返回空，不会报错

字典可以多对一

for 迭代变量会自动修正

del删除列表元素

生成迭代器：iter()， 下一个next()，对应类的__iter__和__next__方法，迭代器的一个好处是不会和列表一样占用内存空间

生成器：是一种返回一个值的迭代器，使用yield产生  
有next方法，send方法  
生成器函数的调用会返回一个生成器(就像是类和实例)，生成器也就是迭代器，好处是可以暂存挂起当前状态，来节省状态  
生成器中的return被调用时会抛出报错，当函数中含有yield关键词的时候，这个函数就已经是一个用来返回生成器的道具了，不是一般的函数  

```py
In [1]: type([i for i in range(5)])     #推导式
Out[1]: list

In [2]: type((i for i in range(5)))     #生成器
Out[2]: generator

```
常见的使用生成器来实现的斐波那契数列
```shell
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
def fab(max): 
    n, a, b = 0, 0, 1 
    while n < max: 
        yield b      # 使用 yield
        # print b 
        a, b = b, a + b 
        n = n + 1
 
for n in fab(5): 
    print n
```

可以用send方法来发送yield的赋值对象，直接赋值会是None，因为从yield处继续开始迭代，所以不会有值
```py
In [1]: def fun():
   ...:     for i in range(5):
   ...:         print("Start...")
   ...:         receive =  yield i
   ...:         print(receive)
   ...:

In [2]: f = fun()

In [3]: next(f)
Start...
Out[3]: 0

In [4]: next(f)
None
Start...
Out[4]: 1

In [5]: next(f)
None
Start...
Out[5]: 2

In [6]: f.send("hello")
hello
Start...
Out[6]: 3
```

对iter调用iter，是同一个迭代器
```py
s = [1,2,3,4]
t = iter(s)
q = iter(t)
```

close方法可以关闭生成器

无穷：value = float('inf')

zip函数：返回可迭代对象元素组成的元组的列表
```py
>>> ls = zip([1,2,3], [4,5,6])
>>> ls
<zip object at 0x7f47f15ef040>
>>> list(ls)
[(1, 4), (2, 5), (3, 6)]
```

## 例题

```py
def path_yielder(t, value):
    """Yields all possible paths from the root of t to a node with the label value
    as a list.

    >>> t1 = Tree(1, [Tree(2, [Tree(3), Tree(4, [Tree(6)]), Tree(5)]), Tree(5)])
    >>> print(t1)
    1
      2
        3
        4
          6
        5
      5
    >>> next(path_yielder(t1, 6))
    [1, 2, 4, 6]
    >>> path_to_5 = path_yielder(t1, 5)
    >>> sorted(list(path_to_5))
    [[1, 2, 5], [1, 5]]

    >>> t2 = Tree(0, [Tree(2, [t1])])
    >>> print(t2)
    0
      2
        1
          2
            3
            4
              6
            5
          5
    >>> path_to_2 = path_yielder(t2, 2)
    >>> sorted(list(path_to_2))
    [[0, 2], [0, 2, 1, 2]]
    """

    ## 递归的信仰之越，无关返回类型，只关心现在的判断和下一个关系
    ## 生成器遇到return，会停止这一项的yield，但是不会返回值，这题表现为停止一条路径，继续执行下面的yield
    ## 因为到叶子节点的时候，path函数是函数，所以返回上一级
    ## 抽象思维，先抽象再具象，如果以上来就具象的话会很难完成
    if t.label == value:
        yield [t.label]
    elif t.is_leaf():
        #use return instead of yield! because the road does not exit, so we shouldn't yield it
        return []   
    for b in t.branches:
        for road in path_yielder(b, value):
            "*** YOUR CODE HERE ***"
            road.insert(0, t.label)
            yield road
```

```py
def remove_all(link , value):
    """Remove all the nodes containing value in link. Assume that the
    first element is never removed.

    >>> l1 = Link(0, Link(2, Link(2, Link(3, Link(1, Link(2, Link(3)))))))
    >>> print(l1)
    <0 2 2 3 1 2 3>
    >>> remove_all(l1, 2)
    >>> print(l1)
    <0 3 1 3>
    >>> remove_all(l1, 3)
    >>> print(l1)
    <0 1>
    >>> remove_all(l1, 3)
    >>> print(l1)
    <0 1>
    """
    "*** YOUR CODE HERE ***"
    #?为什么做起来难度这么大，对于递归要清楚函数的功能，先将函数本身的功能定义好
    if link is Link.empty:
        return
    while not link.rest is Link.empty and link.rest.first == value:
        link.rest = link.rest.rest
    remove_all(link.rest, value)
```

scheme尾递归：需要新定义函数
要存储功能的话就要将存储表作为参数传

```scheme
(define (replicate x n)
  'YOUR-CODE-HERE
  (define (repl lst x n)
    (if (= n 0)
      lst
      (repl (cons x lst) x (- n 1))))
  (repl nil x n)
  )
```

又比如：
```scheme
(define (accumulate combiner start n term)
  'YOUR-CODE-HERE
  (begin
    (define (acc result combiner start n term)
      (if (= n 0)
        result
        (acc (combiner (term start) result) combiner (+ 1 start) (- n 1) term)))
    (acc start combiner 1 n term)
  )
)
```




# 面向对象编程

## 访问限制

在Python中，变量名类似`__xxx__`的，也就是以双下划线开头，并且以双下划线结尾的，是**特殊变量**，特殊变量是可以直接访问的，不是private变量，所以，**不能用`__name__`、`__score__`这样的变量名。**

有些时候，你会看到**以一个下划线开头的实例变量名**，比如`_name`，这样的实例变量外部是可以访问的，但是，按照约定俗成的规定，当你看到这样的变量时，意思就是，**“虽然我可以被访问，但是，请把我视为私有变量，不要随意访问”**。

双下划线开头的实例变量是不是一定不能从外部访问呢？其实也不是。**不能直接访问`__name`是因为Python解释器对外把`__name`变量改成了`_Student__name`，所以，仍然可以通过`_Student__name`来访问`__name`变量**：

```
>>> bart._Student__name
'Bart Simpson'
```

**但是强烈建议你不要这么干，因为不同版本的Python解释器可能会把`__name`改成不同的变量名。**

总的来说就是，Python本身没有任何机制阻止你干坏事，一切全靠自觉。

最后注意下面的这种*错误写法*：

```
>>> bart = Student('Bart Simpson', 59)
>>> bart.get_name()
'Bart Simpson'
>>> bart.__name = 'New Name' # 设置__name变量！
>>> bart.__name
'New Name'
```

表面上看，外部代码“成功”地设置了`__name`变量，但实际上这个`__name`变量和class内部的`__name`变量*不是*一个变量！内部的`__name`变量已经被Python解释器自动改成了`_Student__name`，而外部代码给`bart`新增了一个`__name`变量。不信试试：

```
>>> bart.get_name() # get_name()内部返回self.__name
'Bart Simpson'
```



## 继承

```
class Animal(object):
    def run(self):
        print('Animal is running...')
```

当我们需要编写`Dog`和`Cat`类时，就可以直接从`Animal`类继承：

```
class Dog(Animal):
    pass

class Cat(Animal):
    pass
```

对于`Dog`来说，`Animal`就是它的父类，对于`Animal`来说，`Dog`就是它的子类。`Cat`和`Dog`类似。

继承有什么好处？**最大的好处是子类获得了父类的全部功能**。由于`Animial`实现了`run()`方法，因此，`Dog`和`Cat`作为它的子类，什么事也没干，就自动拥有了`run()`方法



## 获取对象信息

### Why

当我们拿到一个对象的引用时，如何知道这个对象是什么类型、有哪些方法呢？

### type()

**如果判断一个对象是否是函数？使用types模块中的定义的常量**

>>> import types
>>> def fn():     
>>> ...     pass
>>> ...
>>> type(fn)==types.FunctionType     
>>> True
>>> type(abs)==types.BuiltinFunctionType
>>> True
>>> type(lambda x: x)==types.LambdaType
>>> True
>>> type((x for x in range(10)))==types.GeneratorType      
>>> True

### isinstance()

对于class的继承关系来说，使用type()就很不方便。我们要判断class的类型，可以使用isinstance()函数。

**总是优先使用isinstance()判断类型，可以将指定类型及其子类“一网打尽”。**

### dir()

如果要获得一个对象的所有属性和方法，可以使用dir()函数，它返回一个包含字符串的list.

配合`getattr()`、`setattr()`以及`hasattr()`，我们可以直接操作一个对象的状态

如果试图获取不存在的属性，会抛出AttributeError的错误



### 小结

通过内置的一系列函数，我们可以对任意一个Python对象进行剖析，拿到其内部的数据。要注意的是，只有在不知道对象信息的时候，我们才会去获取对象信息。如果可以直接写：

```python
sum = obj.x + obj.y
```

就不要写：

```python
sum = getattr(obj, 'x') + getattr(obj, 'y')
```

一个正确的用法的例子如下：

```python
def readImage(fp):
    if hasattr(fp, 'read'):
        return readData(fp)
    return None
```

假设我们希望从文件流fp中读取图像，我们首先要判断该fp对象是否存在read方法，如果存在，则该对象是一个流，如果不存在，则无法读取。`hasattr()`就派上了用场。

请注意，在Python这类动态语言中，根据鸭子类型，有`read()`方法，不代表该fp对象就是一个文件流，它也可能是网络流，也可能是内存中的一个字节流，但只要`read()`方法返回的是有效的图像数据，就不影响读取图像的功能。

## 实例属性和类属性

给实例绑定属性的方法是通过实例变量，或者通过`self`变量：

```python
class Student(object):
    def __init__(self, name):
        self.name = name

s = Student('Bob')
s.score = 90
```

但是，如果`Student`类本身需要绑定一个属性呢？可以直接在class中定义属性，这种属性是类属性，归`Student`类所有：

```python
class Student(object):
    name = 'Student'
```



# 面向对象高级编程

## 给实例/class动态绑定属性和方法

给**实例**绑定一个**属性**：

```python
>>> class Student(object):
    	pass
>>> s = Student()
>>> s.name = 'Michael' # 动态给实例绑定一个属性
>>> print(s.name)
Michael
```

还可以尝试给**实例**绑定一个**方法**：

```python
>>> def set_age(self, age): # 定义一个函数作为实例方法
...     self.age = age
...
>>> from types import MethodType
>>> s.set_age = MethodType(set_age, s) # 给实例绑定一个方法
>>> s.set_age(25) # 调用实例方法
>>> s.age # 测试结果
25
```

为了给所有实例都绑定方法，可以给**class**绑定**方法**：

```python
>>> def set_score(self, score):
...     self.score = score
...
>>> Student.set_score = set_score
```

### 如果要限制实例属性怎么办？

比如，只允许对Student实例添加`name`和`age`属性。比如，只允许对Student实例添加`name`和`age`属性。

```
class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称
```

使用`__slots__`要注意，`__slots__`定义的属性仅对当前类实例起作用，对继承的子类是不起作用的。除非在子类中也定义`__slots__`，这样，子类实例允许定义的属性就是自身的`__slots__`加上父类的`__slots__`。

## @property

注意到这个神奇的`@property`，我们在对实例属性操作的时候，就知道**该属性很可能不是直接暴露的**，而是通过getter和setter方法来实现的。

还可以定义只读属性，只定义getter方法，不定义setter方法就是一个只读属性：

```
class Student(object):

    @property
    def birth(self):
        return self._birth

    @birth.setter
    def birth(self, value):
        self._birth = value

    @property
    def age(self):
        return 2015 - self._birth
```

上面的`birth`是可读写属性，而`age`就是一个*只读*属性，因为`age`可以根据`birth`和当前时间计算出来。



## 多重继承

子类可以继承多个父类。但由于继承遵守“is a”关系，从按以上依然遵守单继承的原则。

```
class Dog(Mammal, RunnableMixIn, CarnivorousMixIn):
    pass
```

如上，上面Dog实现了多继承，不过它的第二个和第三个类我们在起名的时候加上了“-Mixin”的后缀，这个不影响功能，但是会告诉读代码的人：这个是一个Mixin类。从含义上理解，Dog只是一个Mammal，不是一个Rannable。这个Mixin表示混入（mix in），它告诉别人，这个类是作为功能添加到子类中，而不是作为父类。

使用Mixin类实现多重继承要非常小心

* 他必须表示一种功能，而不是某个物品。
* 它必须责任单一，没有多个功能；如果有多个功能，那就写多个Mixin类
* 它不依赖于子类的实现
* 子类即便没有继承这个Mixin类，也照样可以工作，就是缺少某个功能。



## 定制类

### \__getattr__()

如果没有属性，则自动调用类中的\__getattr__()函数。

**应用**

现在很多网站都搞REST API，比如新浪微博、豆瓣啥的，调用API的URL类似：

- http://api.server/user/friends
- http://api.server/user/timeline/list

如果要写SDK，给每个URL对应的API都写一个方法，那得累死，而且，API一旦改动，SDK也要改。

利用完全动态的`__getattr__`，我们可以写出一个链式调用：

```python
class Chain(object):

    def __init__(self, path=''):
        self._path = path

    def __getattr__(self, path):
        return Chain('%s/%s' % (self._path, path))
        
    def __call__(self, path):
        return Chain('%s/%s' % (self.__path, path))

    def __str__(self):
        return self._path

    __repr__ = __str__
```

试试：

```
>>> Chain().status.user.timeline.list
'/status/user/timeline/list'
```

**解析**

```python
Step 1：
Chain()  # 实例化
Step 2：
Chain().users
# 由于没有给实例传入初始化对应属性的具体信息，从而自动调用__getattr__()函数，从而有：
Chain().users = Chain('\users') # 这是重建实例
Step 3:
Chain().users('michael')
Chain().users('michael') = Chain('\users')('michael') # 这是对实例直接调用，相当于调用普通函数一样
# 关键就在这步，这一步返回的是Chain('\users\michael'),再一次重建实例，覆盖掉Chain('\users'),
#记 renew = Chain('\users\michael')， 此时新实例的属性renew.__path = \users\michael;
Step 4:
Chain().users('michael').repos
# 这一步是查询renew实例的属性repos，由于没有这一属性，就会执行__getattr__()函数，再一次返回新的实例Chain('\users\michael\repos')并且覆盖点之前的实例，
# 这里记 trinew =Chain('\users\michael\repos')，不要忘了，一单定义了一个新的实例，就会执行__init__方法；
Step 5：
print(Chain().users('michael').repos) = print(trinew)  #由于我们定义了__str__()方法，那么打印的时候就会调用此方法，据此方法的定义，打印回来的是trinew的__path属#性，即——\users\michael\repos  。至此，我们也把所有定义的有特殊用的方法都用上了，完毕。
```

### 枚举类

枚举类型可以看作是一种标签或是一系列常量的集合，通常用于表示某些特定的有限集合，例如星期、月份、状态等。在没有专门提供枚举类型的时候我们是怎么做呢，一般就通过字典或类来实现：

```
Color = {
    'RED'  : 1,
    'GREEN': 2,
    'BLUE' : 3,
}

class Color:
    RED   = 1
    GREEN = 2
    BLUE  = 3
```

这种来实现枚举如果小心翼翼地使用当然没什么问题，毕竟是一种妥协的解决方案。它的隐患在于可以被修改。

更好的方式是使用标准库提供的 `Enum` 类型，官方库值得信赖。3.4 之前的版本也可以通过 `pip install enum` 下载支持的库。简单的示例：

```
from enum import Enum
class Color(Enum):
    red = 1
    green = 2
    blue = 3
```

如果不赋值，则`value`属性则是自动赋给成员的`int`常量，默认从`1`开始计数。

如果需要更精确地控制枚举类型，可以从`Enum`派生出自定义类：

```
from enum import Enum, unique

@unique
class Weekday(Enum):
    Sun = 0 # Sun的value被设定为0
    Mon = 1
    Tue = 2
    Wed = 3
    Thu = 4
    Fri = 5
    Sat = 6
```

**定义枚举时，成员名不允许重复**

**成员值允许相同，第二个成员的名称被视作第一个成员的别名**

**若要不能定义相同的成员值，可以通过 `@unique` 装饰**


## 元类



### type()

动态语言和静态语言最大的不同，就是函数和类的定义，不是编译时定义的，而是运行时动态创建的。

比方说我们要定义一个`Hello`的class，就写一个`hello.py`模块：

```python
class Hello(object):
    def hello(self, name='world'):
        print('Hello, %s.' % name)
```

当Python解释器载入`hello`模块时，就会依次执行该模块的所有语句，执行结果就是动态创建出一个`Hello`的class对象，测试如下：

```python
>>> from hello import Hello
>>> h = Hello()
>>> h.hello()
Hello, world.
>>> print(type(Hello))
<class 'type'>
>>> print(type(h))
<class 'hello.Hello'>
```

`type()`函数可以查看一个类型或变量的类型，`Hello`是一个class，它的类型就是`type`，而`h`是一个实例，它的类型就是class `Hello`。

我们说class的定义是运行时动态创建的，而创建class的方法就是使用`type()`函数。

`type()`函数既可以返回一个对象的类型，又可以创建出新的类型，比如，我们可以通过`type()`函数创建出`Hello`类，而无需通过`class Hello(object)...`的定义：

```python
>>> def fn(self, name='world'): # 先定义函数
...     print('Hello, %s.' % name)
...
>>> Hello = type('Hello', (object,), dict(hello=fn)) # 创建Hello class
>>> h = Hello()
>>> h.hello()
Hello, world.
>>> print(type(Hello))
<class 'type'>
>>> print(type(h))
<class '__main__.Hello'>
```

要创建一个class对象，`type()`函数依次传入3个参数：

1. class的名称；
2. 继承的父类集合，注意Python支持多重继承，如果只有一个父类，别忘了tuple的单元素写法；
3. class的方法名称与函数绑定，这里我们把函数`fn`绑定到方法名`hello`上。

#### 疑惑： 什么时候使用type（）来定义类呢？感觉好不清晰。

通过`type()`函数创建的类和直接写class是完全一样的，因为Python解释器遇到class定义时，仅仅是扫描一下class定义的语法，然后调用`type()`函数创建出class。

正常情况下，我们都用`class Xxx...`来定义类，但是，`type()`函数也允许我们动态创建出类来，也就是说，动态语言本身支持运行期动态创建类，这和静态语言有非常大的不同，要在静态语言运行期创建类，必须构造源代码字符串再调用编译器，或者借助一些工具生成字节码实现，本质上都是动态编译，会非常复杂。



### metaclass

metaclass是Python中非常具有魔术性的对象，它可以改变类创建时的行为。这种强大的功能使用起来务必小心。



# 错误、调试和测试

## 错误处理

在程序运行的过程中，如果发生了错误，可以事先约定返回一个错误代码，这样，就可以知道是否有错，以及出错的原因。

用错误码来表示是否出错十分不便，因为函数本身应该返回的正常结果和错误码混在一起，造成调用者必须用大量的代码来判断是否出错，一旦出错，还要一级一级上报，直到某个函数可以处理该错误（比如，给用户输出一个错误信息）。

所以高级语言通常都内置了一套`try...except...finally...`的错误处理机制，Python也不例外。

### try

让我们用一个例子来看看`try`的机制：

```python
try:
    print('try...')
    r = 10 / 0
    print('result:', r)
except ZeroDivisionError as e:
    print('except:', e)
finally:
    print('finally...')
print('END')
```

* 当我们认为某些代码可能会出错时，就可以用`try`来运行这段代码，
* 如果执行出错，则后续代码不会继续执行，而是直接跳转至错误处理代码，即`except`语句块，
* 执行完`except`后，如果有`finally`语句块，则执行`finally`语句块，至此，执行完毕。
* 由于没有错误发生，所以`except`语句块不会被执行，但是`finally`如果有，则一定会被执行（可以没有`finally`语句）。

**Python所有的错误都是从`BaseException`类派生的**，常见的错误类型和继承关系看这里：

https://docs.python.org/3/library/exceptions.html#exception-hierarchy

使用`try...except`捕获错误还有一个巨大的好处，就是可以跨越多层调用，比如函数`main()`调用`bar()`，`bar()`调用`foo()`，结果`foo()`出错了，这时，只要`main()`捕获到了，就可以处理：

```python
def foo(s):
    return 10 / int(s)

def bar(s):
    return foo(s) * 2

def main():
    try:
        bar('0')
    except Exception as e:
        print('Error:', e)
    finally:
        print('finally...')
```

也就是说，不需要在每个可能出错的地方去捕获错误，只要在合适的层次去捕获错误就可以了。这样一来，就大大减少了写`try...except...finally`的麻烦。

### 调用栈

```python
$ python3 err.py
Traceback (most recent call last):
  File "err.py", line 11, in <module>
    main()
  File "err.py", line 9, in main
    bar('0')
  File "err.py", line 6, in bar
    return foo(s) * 2
  File "err.py", line 3, in foo
    return 10 / int(s)
ZeroDivisionError: division by zero
```

**出错的时候，一定要分析错误的调用栈信息，才能定位错误的位置。**

### 记录错误

如果不捕获错误，自然可以让Python解释器来打印出错误堆栈，但程序也被结束了。**既然我们能捕获错误，就可以把错误堆栈打印出来，然后分析错误原因，同时，让程序继续执行下去。**

Python内置的`logging`模块可以非常容易地记录错误信息：

```python
# err_logging.py

import logging

def foo(s):
    return 10 / int(s)

def bar(s):
    return foo(s) * 2

def main():
    try:
        bar('0')
    except Exception as e:
        logging.exception(e)

main()
print('END')
```

同样是出错，但程序打印完错误信息后会继续执行，并正常退出：

```python
$ python3 err_logging.py
ERROR:root:division by zero
Traceback (most recent call last):
  File "err_logging.py", line 13, in main
    bar('0')
  File "err_logging.py", line 9, in bar
    return foo(s) * 2
  File "err_logging.py", line 6, in foo
    return 10 / int(s)
ZeroDivisionError: division by zero
END
```

通过配置，`logging`还可以把错误记录到日志文件里，方便事后排查。

### 抛出错误

因为错误是class，捕获一个错误就是捕获到该class的一个实例。因此，错误并不是凭空产生的，而是有意创建并抛出的。Python的内置函数会抛出很多类型的错误，我们自己编写的函数也可以抛出错误。

如果要抛出错误：

* 首先根据需要，可以定义一个错误的class，选择好继承关系，
* 然后，用`raise`语句抛出一个错误的实例：

```python
# err_raise.py
class FooError(ValueError):
    pass

def foo(s):
    n = int(s)
    if n==0:
        raise FooError('invalid value: %s' % s)
    return 10 / n

foo('0')
```

执行，可以最后跟踪到我们自己定义的错误：

```python
$ python3 err_raise.py 
Traceback (most recent call last):
  File "err_throw.py", line 11, in <module>
    foo('0')
  File "err_throw.py", line 8, in foo
    raise FooError('invalid value: %s' % s)
__main__.FooError: invalid value: 0
```

**只有在必要的时候才定义我们自己的错误类型。如果可以选择Python已有的内置的错误类型（比如`ValueError`，`TypeError`），尽量使用Python内置的错误类型。**

#### 另一种错误处理的方式

```python
# err_reraise.py

def foo(s):
    n = int(s)
    if n==0:
        raise ValueError('invalid value: %s' % s)
    return 10 / n

def bar():
    try:
        foo('0')
    except ValueError as e:
        print('ValueError!')
        raise

bar()
```

在`bar()`函数中，我们明明已经捕获了错误，但是，打印一个`ValueError!`后，又把错误通过`raise`语句抛出去了，这不有病么？

其实这种错误处理方式不但没病，而且相当常见。**捕获错误目的只是记录一下，便于后续追踪。但是，由于当前函数不知道应该怎么处理该错误，所以，最恰当的方式是继续往上抛，让顶层调用者去处理。**好比一个员工处理不了一个问题时，就把问题抛给他的老板，如果他的老板也处理不了，就一直往上抛，最终会抛给CEO去处理。

**`raise`语句如果不带参数，就会把当前错误原样抛出。**

此外，在`except`中`raise`一个Error，还可以把一种类型的错误转化成另一种类型：

```python
try:
    10 / 0
except ZeroDivisionError:
    raise ValueError('input error!')
```

只要是合理的转换逻辑就可以，但是，**决不应该把一个`IOError`转换成毫不相干的`ValueError`**。

### 小结

**Python内置的`try...except...finally`用来处理错误十分方便。出错时，会分析错误信息并定位错误发生的代码位置才是最关键的。**

**程序也可以主动抛出错误，让调用者来处理相应的错误。但是，应该在文档中写清楚可能会抛出哪些错误，以及错误产生的原因。**





## 调试

### print()

第一种方法简单直接粗暴有效，就是用`print()`把可能有问题的变量打印出来看看。**最大的坏处是将来还得删掉它**



### 断言

凡是用`print()`来辅助查看的地方，都可以用断言（assert）来替代

```
def foo(s):
    n = int(s)
    assert n != 0, 'n is zero!'
    return 10 / n

def main():
    foo('0')
```

`assert`的意思是，表达式`n != 0`应该是`True`，否则，根据程序运行的逻辑，后面的代码肯定会出错。

如果断言失败，`assert`语句本身就会抛出`AssertionError`：

```
$ python err.py
Traceback (most recent call last):
  ...
AssertionError: n is zero!
```

程序中如果到处充斥着`assert`，和`print()`相比也好不到哪去。不过，启动Python解释器时可以用`-O`参数来关闭`assert`：

```
$ python -O err.py
Traceback (most recent call last):
  ...
ZeroDivisionError: division by zero
```

**注意：断言的开关“-O”是英文大写字母O，不是数字0。**

关闭后，你可以把所有的`assert`语句当成`pass`来看。

### logging

把`print()`替换为`logging`是第3种方式，和`assert`比，`logging`不会抛出错误，而且可以输出到文件：

```
import logging

s = '0'
n = int(s)
logging.info('n = %d' % n)
print(10 / n)
```

`logging.info()`就可以输出一段文本。运行，发现除了`ZeroDivisionError`，没有任何信息。怎么回事？

别急，在`import logging`之后添加一行配置再试试：

```
import logging
logging.basicConfig(level=logging.INFO)
```

看到输出了：

```
$ python err.py
INFO:root:n = 0
Traceback (most recent call last):
  File "err.py", line 8, in <module>
    print(10 / n)
ZeroDivisionError: division by zero
```

这就是`logging`的好处，它**允许你指定记录信息的级别，有`debug`，`info`，`warning`，`error`等几个级别，当我们指定`level=INFO`时，`logging.debug`就不起作用了。同理，指定`level=WARNING`后，`debug`和`info`就不起作用了**。这样一来，你可以放心地输出不同级别的信息，也不用删除，最后统一控制输出哪个级别的信息。

`logging`的另一个好处是**通过简单的配置，一条语句可以同时输出到不同的地方，比如console和文件**。

### pdb

第4种方式是启动Python的调试器pdb，让程序以单步方式运行，可以随时查看运行状态。我们先准备好程序：

```
# err.py
s = '0'
n = int(s)
print(10 / n)
```

然后启动：

```
$ python -m pdb err.py
> /Users/michael/Github/learn-python3/samples/debug/err.py(2)<module>()
-> s = '0'
```

以参数`-m pdb`启动后，pdb定位到下一步要执行的代码`-> s = '0'`。输入**命令`l`来查看代码**：

```
(Pdb) l
  1     # err.py
  2  -> s = '0'
  3     n = int(s)
  4     print(10 / n)
```

输入**命令`n`可以单步执行代码**：

```
(Pdb) n
> /Users/michael/Github/learn-python3/samples/debug/err.py(3)<module>()
-> n = int(s)
(Pdb) n
> /Users/michael/Github/learn-python3/samples/debug/err.py(4)<module>()
-> print(10 / n)
```

任何时候都可以输入**命令`p 变量名`来查看变量**：

```
(Pdb) p s
'0'
(Pdb) p n
0
```

输入**命令`q`结束调试，退出程序**：

```
(Pdb) q
```

这种通过pdb在命令行调试的方法理论上是万能的，但实在是太麻烦了，如果有一千行代码，要运行到第999行得敲多少命令啊。还好，我们还有另一种调试方法。

### pdb.set_trace()

这个方法也是用pdb，但是不需要单步执行：

* 我们只需要`import pdb`，
* 然后，在可能出错的地方放一个`pdb.set_trace()`，就可以设置一个断点：

```
# err.py
import pdb

s = '0'
n = int(s)
pdb.set_trace() # 运行到这里会自动暂停
print(10 / n)
```

运行代码，程序会自动在`pdb.set_trace()`暂停并进入pdb调试环境，可以用命令`p`查看变量，或者用**命令`c`继续运行**：

```
$ python err.py 
> /Users/michael/Github/learn-python3/samples/debug/err.py(7)<module>()
-> print(10 / n)
(Pdb) p n
0
(Pdb) c
Traceback (most recent call last):
  File "err.py", line 7, in <module>
    print(10 / n)
ZeroDivisionError: division by zero
```

这个方式比直接启动pdb单步调试效率要高很多，但也高不到哪去。

### IDE

如果要比较爽地设置断点、单步执行，就需要一个支持调试功能的IDE。目前比较好的Python IDE有：

Visual Studio Code：https://code.visualstudio.com/，需要安装Python插件。

PyCharm：http://www.jetbrains.com/pycharm/

### 小结

虽然用IDE调试起来比较方便，但是最后你会发现，logging才是终极武器。



## 单元测试

单元测试是用来对一个模块、一个函数或者一个类来进行正确性检验的测试工作。

比如对函数`abs()`，我们可以编写出以下几个测试用例：

1. 输入正数，比如`1`、`1.2`、`0.99`，期待返回值与输入相同；
2. 输入负数，比如`-1`、`-1.2`、`-0.99`，期待返回值与输入相反；
3. 输入`0`，期待返回`0`；
4. 输入非数值类型，比如`None`、`[]`、`{}`，期待抛出`TypeError`。

把上面的测试用例放到一个测试模块里，就是一个完整的单元测试。

如果单元测试通过，说明我们测试的这个函数能够正常工作。如果单元测试不通过，要么函数有bug，要么测试条件输入不正确，总之，需要修复使单元测试能够通过。

单元测试通过后有什么意义呢？如果我们对`abs()`函数代码做了修改，只需要再跑一遍单元测试，如果通过，说明我们的修改不会对`abs()`函数原有的行为造成影响，如果测试不通过，说明我们的修改与原有行为不一致，要么修改代码，要么修改测试。

这种以测试为驱动的开发模式最大的好处就是确保一个程序模块的行为符合我们设计的测试用例。在将来修改的时候，可以极大程度地保证该模块行为仍然是正确的。

我们来编写一个`Dict`类，这个类的行为和`dict`一致，但是可以通过属性来访问，用起来就像下面这样：

```
>>> d = Dict(a=1, b=2)
>>> d['a']
1
>>> d.a
1
```

`mydict.py`代码如下：

```
class Dict(dict):

    def __init__(self, **kw):
        super().__init__(**kw) # 继承父类（dict类）的__init__方法

    def __getattr__(self, key):# 定义
        try:
            return self[key]
        except KeyError:
            raise AttributeError(r"'Dict' object has no attribute '%s'" % key)

    def __setattr__(self, key, value):
        self[key] = value
```

为了编写单元测试，我们需要引入Python自带的`unittest`模块，编写`mydict_test.py`如下：

```python
import unittest

from mydict import Dict

class TestDict(unittest.TestCase):# 从unittest.TestCase继承

# 以test开头的方法就是测试方法，不以test开头的方法不被认为是测试方法，测试的时候不会被执行。    
    
    def test_init(self):
        d = Dict(a=1, b='test')
        self.assertEqual(d.a, 1) # 断言d.a与1相等
        self.assertEqual(d.b, 'test')
        self.assertTrue(isinstance(d, dict))# 断言d是dict类型

    def test_key(self):
        d = Dict()
        d['key'] = 'value'
        self.assertEqual(d.key, 'value')

    def test_attr(self):
        d = Dict()
        d.key = 'value'
        self.assertTrue('key' in d)
        self.assertEqual(d['key'], 'value')

    def test_keyerror(self):
        d = Dict()
        with self.assertRaises(KeyError):#通过d['empty']访问不存在的key时，断言会抛出KeyError
            value = d['empty']

    def test_attrerror(self):
        d = Dict()
        with self.assertRaises(AttributeError):#通过d.empty访问不存在的key时，我们期待抛出AttributeError
            value = d.empty
```

编写单元测试时，我们需要编写一个测试类，从`unittest.TestCase`继承。

以`test`开头的方法就是测试方法，不以`test`开头的方法不被认为是测试方法，测试的时候不会被执行。

对每一类测试都需要编写一个`test_xxx()`方法。由于`unittest.TestCase`提供了很多内置的条件判断，我们只需要调用这些方法就可以断言输出是否是我们所期望的。最常用的断言就是`assertEqual()`：

```
self.assertEqual(abs(-1), 1) # 断言函数返回的结果与1相等
```

**另一种重要的断言就是期待抛出指定类型的Error**，比如通过`d['empty']`访问不存在的key时，断言会抛出`KeyError`：

```
with self.assertRaises(KeyError):
    value = d['empty']
```

而通过`d.empty`访问不存在的key时，我们期待抛出`AttributeError`：

```
with self.assertRaises(AttributeError):
    value = d.empty
```

### 运行单元测试

一旦编写好单元测试，我们就可以运行单元测试。

**最简单的运行方式**是**在`mydict_test.py`的最后加上两行代码**：

```
if __name__ == '__main__':
    unittest.main()
```

这样就可以把`mydict_test.py`当做正常的python脚本运行：

```
$ python mydict_test.py
```

**另一种方法是在命令行通过参数`-m unittest`直接运行单元测试**：

```
$ python -m unittest mydict_test
.....
----------------------------------------------------------------------
Ran 5 tests in 0.000s

OK
```

**这是推荐的做法**，因为这样可以一次批量运行很多单元测试，并且，有很多工具可以自动来运行这些单元测试。

### setUp与tearDown

可以在单元测试中编写两个特殊的`setUp()`和`tearDown()`方法。这两个方法会**分别在每调用一个测试方法的前后分别被执行**。

`setUp()`和`tearDown()`方法有什么用呢？设想你的测试需要启动一个数据库，这时，就可以在`setUp()`方法中连接数据库，在`tearDown()`方法中关闭数据库，这样，不必在每个测试方法中重复相同的代码：

```
class TestDict(unittest.TestCase):

    def setUp(self):
        print('setUp...')

    def tearDown(self):
        print('tearDown...')
```

可以再次运行测试看看每个测试方法调用前后是否会打印出`setUp...`和`tearDown...`。



## 文档测试doctest

可不可以自动执行写在注释中的这些代码呢？

答案是肯定的。当我们编写注释时，如果写上这样的注释：

```
def abs(n):
    '''
    Function to get absolute value of number.
    
    Example:
    
    >>> abs(1)
    1
    >>> abs(-1)
    1
    >>> abs(0)
    0
    '''
    return n if n >= 0 else (-n)
```

无疑更明确地告诉函数的调用者该函数的期望输入和输出。

并且，Python内置的“文档测试”（doctest）模块可以直接提取注释中的代码并执行测试。

### 使用方法：

doctest严格按照Python交互式命令行的输入和输出来判断测试结果是否正确，正常的时候，不会有输出，只有测试异常的时候，才会出现异常输出。

* 使用的时候，只需要加入：

* ```
  if __name__=='__main__':
      import doctest
      doctest.testmod()
  ```

* 对于注释部分，可以用`...`表示中间一大段烦人的输出。



让我们用doctest来测试上次编写的`Dict`类：

```python
# mydict2.py
class Dict(dict):
    '''
    Simple dict but also support access as x.y style.

    >>> d1 = Dict()
    >>> d1['x'] = 100
    >>> d1.x
    100
    >>> d1.y = 200
    >>> d1['y']
    200
    >>> d2 = Dict(a=1, b=2, c='3')
    >>> d2.c
    '3'
    >>> d2['empty']
    Traceback (most recent call last):
        ...
    KeyError: 'empty'
    >>> d2.empty
    Traceback (most recent call last):
        ...
    AttributeError: 'Dict' object has no attribute 'empty'
    '''
    def __init__(self, **kw):
        super(Dict, self).__init__(**kw)

    def __getattr__(self, key):
        try:
            return self[key]
        except KeyError:
            raise AttributeError(r"'Dict' object has no attribute '%s'" % key)

    def __setattr__(self, key, value):
        self[key] = value

if __name__=='__main__':# 当模块正常导入时，doctest不会被执行。
    import doctest
    doctest.testmod()
```

运行`python mydict2.py`：

```python
$ python mydict2.py
```

**什么输出也没有。这说明我们编写的doctest运行都是正确的。**

如果程序有问题，比如把`__getattr__()`方法注释掉，再**运行就会报错：**

```python
$ python mydict2.py
**********************************************************************
File "/Users/michael/Github/learn-python3/samples/debug/mydict2.py", line 10, in __main__.Dict
Failed example:
    d1.x
Exception raised:
    Traceback (most recent call last):
      ...
    AttributeError: 'Dict' object has no attribute 'x'
**********************************************************************
File "/Users/michael/Github/learn-python3/samples/debug/mydict2.py", line 16, in __main__.Dict
Failed example:
    d2.c
Exception raised:
    Traceback (most recent call last):
      ...
    AttributeError: 'Dict' object has no attribute 'c'
**********************************************************************
1 items had failures:
   2 of   9 in __main__.Dict
***Test Failed*** 2 failures.
```

注意到最后3行代码。当模块正常导入时，doctest不会被执行。只有在命令行直接运行时，才执行doctest。所以，不必担心doctest会在非测试环境下执行。



# I/O编程

## 文件读写

在磁盘上读写文件的功能都是由操作系统提供的。

* 现代操作系统不允许普通的程序直接操作磁盘，所以，读写文件就是请求操作系统打开一个文件对象（通常称为文件描述符），
* 然后，通过操作系统提供的接口从这个文件对象中读取数据（读文件），或者把数据写入这个文件对象（写文件）。

### 读文件

要以读文件的模式打开一个文件对象，使用Python内置的`open()`函数，传入文件名和标示符：

```python
>>> f = open('/Users/michael/test.txt', 'r') # 标示符'r'表示读
```

这样，我们就成功地打开了一个文件。

* 如果文件不存在，`open()`函数就会抛出一个`IOError`的错误，并且给出错误码和详细的信息告诉你文件不存在：

  ```
  >>> f=open('/Users/michael/notfound.txt', 'r')
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  FileNotFoundError: [Errno 2] No such file or directory: '/Users/michael/notfound.txt'
  ```

* 如果文件打开成功，接下来，调用`read()`方法可以一次读取文件的全部内容，Python把内容读到内存，用一个`str`对象表示：

  ```python
  >>> f.read()
  'Hello, world!'
  ```

* 最后一步是调用`close()`方法关闭文件。文件使用完毕后必须关闭，因为文件对象会占用操作系统的资源，并且操作系统同一时间能打开的文件数量也是有限的：

  ```
  >>> f.close()
  ```

#### try ... finally

由于文件读写时都有可能产生`IOError`，一旦出错，后面的`f.close()`就不会调用。所以，为了保证无论是否出错都能正确地关闭文件，我们可以使用`try ... finally`来实现：

```
try:
    f = open('/path/to/file', 'r')
    print(f.read())
finally:
    if f:
        f.close()
```

#### with

但是每次都这么写实在太繁琐，所以，Python引入了`with`语句来自动帮我们调用`close()`方法：

```
with open('/path/to/file', 'r') as f:
    print(f.read())
```

**这和前面的`try ... finally`是一样的，但是代码更佳简洁，并且不必调用`f.close()`方法。**

#### tips of read()

调用`read()`会一次性读取文件的全部内容，如果文件有10G，内存就爆了。

所以，要保险起见，可以反复调用**`read(size)`方法**，每次最多读取size个字节的内容。

另外，调用`readline()`可以每次读取一行内容，调用`readlines()`一次读取所有内容并按行返回`list`。因此，要根据需要决定怎么调用。

* 如果文件很小，`read()`一次性读取最方便；
* 如果不能确定文件大小，反复调用`read(size)`比较保险；
* 如果是配置文件，调用`readlines()`最方便：

```
for line in f.readlines():
    print(line.strip()) # 把末尾的'\n'删掉
```





###　file-like Object

像`open()`函数返回的这种有个`read()`方法的对象，在Python中统称为file-like Object。除了file外，还可以是内存的字节流，网络流，自定义流等等。file-like Object不要求从特定类继承，只要写个`read()`方法就行。

`StringIO`就是在内存中创建的file-like Object，常用作临时缓冲。



### 二进制文件

前面讲的默认都是读取文本文件，并且是UTF-8编码的文本文件。

**要读取二进制文件，比如图片、视频等等，用`'rb'`模式打开文件**即可：

```
>>> f = open('/Users/michael/test.jpg', 'rb')
>>> f.read()
b'\xff\xd8\xff\xe1\x00\x18Exif\x00\x00...' # 十六进制表示的字节
```

### 字符编码

**要读取非UTF-8编码的文本文件，需要给`open()`函数传入`encoding`参数**，例如，读取GBK编码的文件：

```
>>> f = open('/Users/michael/gbk.txt', 'r', encoding='gbk')
>>> f.read()
'测试'
```

**遇到有些编码不规范的文件，你可能会遇到`UnicodeDecodeError`，因为在文本文件中可能夹杂了一些非法编码的字符**。**遇到这种情况，`open()`函数还接收一个`errors`参数，表示如果遇到编码错误后如何处理。最简单的方式是直接忽略**：

```
>>> f = open('/Users/michael/gbk.txt', 'r', encoding='gbk', errors='ignore')
```



### 写文件

写文件和读文件是一样的，**唯一区别是调用`open()`函数时，传入标识符`'w'`或者`'wb'`表示写文本文件或写二进制文件**：

```
>>> f = open('/Users/michael/test.txt', 'w')
>>> f.write('Hello, world!')
>>> f.close()
```

你可以反复调用`write()`来写入文件，但是**务必要调用`f.close()`来关闭文件**。当我们写文件时，操作系统往往不会立刻把数据写入磁盘，而是放到内存缓存起来，空闲的时候再慢慢写入。只有调用`close()`方法时，操作系统才保证把没有写入的数据全部写入磁盘。忘记调用`close()`的后果是数据可能只写了一部分到磁盘，剩下的丢失了。所以，**还是用`with`语句来得保险**：

```
with open('/Users/michael/test.txt', 'w') as f:
    f.write('Hello, world!')
```

**要写入特定编码的文本文件，请给`open()`函数传入`encoding`参数，将字符串自动转换成指定编码**。

#### tips of write() 

* 以`'w'`模式写入文件时，如果文件已存在，**会直接覆盖**（相当于删掉后新写入一个文件）。
* 如果我们希望追加到文件末尾怎么办？**可以传入`'a'`以追加（append）模式写入**。

所有模式的定义及含义可以参考Python的[官方文档](https://docs.python.org/3/library/functions.html#open)。



## StringIO & BytesIO

### StringIO

顾名思义就是在内存中读写str。

Step:

* 先创建一个StringIO
* 像文件一样写入即可

```python
>>> from io import StringIO
>>> f = StringIO()
>>> f.write('hello')
5
>>> f.write(' ')
1
>>> f.write('world!')
6
>>> print(f.getvalue()) # 方法用于获得写入后的str
hello world!
```

要读取StringIO，可以用一个str初始化StringIO，然后，像读文件一样读取：

```
>>> from io import StringIO
>>> f = StringIO('Hello!\nHi!\nGoodbye!')
>>> while True:
...     s = f.readline()
...     if s == '':
...         break
...     print(s.strip())
...
Hello!
Hi!
Goodbye!
```



### BytesIO

如果要操作二进制数据，就需要使用BytesIO。

BytesIO实现了在内存中读写bytes，我们创建一个BytesIO，然后写入一些bytes：

```
>>> from io import BytesIO
>>> f = BytesIO()
>>> f.write('中文'.encode('utf-8'))
6
>>> print(f.getvalue())
b'\xe4\xb8\xad\xe6\x96\x87'
```

请注意，写入的不是str，而是经过UTF-8编码的bytes。

和StringIO类似，可以用一个bytes初始化BytesIO，然后，像读文件一样读取：

```
>>> from io import BytesIO
>>> f = BytesIO(b'\xe4\xb8\xad\xe6\x96\x87')
>>> f.read()
b'\xe4\xb8\xad\xe6\x96\x87'
```



## 操作文件和目录

如果要在Python程序中执行这些目录和文件的操作怎么办？Python内置的`os`模块也可以直接调用操作系统提供的接口函数。

打开Python交互式命令行，我们来看看如何使用`os`模块的基本功能：

```
>>> import os
>>> os.name # 操作系统类型
'posix'
```

如果是`posix`，说明系统是`Linux`、`Unix`或`Mac OS X`，如果是`nt`，就是`Windows`系统。

要获取详细的系统信息，可以调用`uname()`函数：

```
>>> os.uname()
posix.uname_result(sysname='Darwin', nodename='MichaelMacPro.local', release='14.3.0', version='Darwin Kernel Version 14.3.0: Mon Mar 23 11:59:05 PDT 2015; root:xnu-2782.20.48~5/RELEASE_X86_64', machine='x86_64')
```

注意`uname()`函数在Windows上不提供，也就是说，`os`模块的某些函数是跟操作系统相关的。

### 环境变量

在操作系统中定义的环境变量，全部保存在`os.environ`这个变量中，可以直接查看：

```
>>> os.environ
environ({'VERSIONER_PYTHON_PREFER_32_BIT': 'no', 'TERM_PROGRAM_VERSION': '326', 'LOGNAME': 'michael', 'USER': 'michael', 'PATH': '/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/opt/X11/bin:/usr/local/mysql/bin', ...})
```

要获取某个环境变量的值，可以调用`os.environ.get('key')`：

```
>>> os.environ.get('PATH')
'/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/opt/X11/bin:/usr/local/mysql/bin'
>>> os.environ.get('x', 'default')
'default'
```

### 操作文件和目录

操作文件和目录的函数一部分放在`os`模块中，一部分放在`os.path`模块中，这一点要注意一下。查看、创建和删除目录可以这么调用：

```
# 查看当前目录的绝对路径:
>>> os.path.abspath('.')
'/Users/michael'
# 在某个目录下创建一个新目录，首先把新目录的完整路径表示出来:
>>> os.path.join('/Users/michael', 'testdir')
'/Users/michael/testdir'
# 然后创建一个目录:
>>> os.mkdir('/Users/michael/testdir')
# 删掉一个目录:
>>> os.rmdir('/Users/michael/testdir')
```

**把两个路径合成一个时，不要直接拼字符串，而要通过`os.path.join()`函数，这样可以正确处理不同操作系统的路径分隔符**。在Linux/Unix/Mac下，`os.path.join()`返回这样的字符串：

```
part-1/part-2
```

而Windows下会返回这样的字符串：

```
part-1\part-2
```

同样的道理，**要拆分路径时，也不要直接去拆字符串，而要通过`os.path.split()`函数**，这样可以把一个路径拆分为两部分，**后一部分总是最后级别的目录或文件名：**

```
>>> os.path.split('/Users/michael/testdir/file.txt')
('/Users/michael/testdir', 'file.txt')
```

**`os.path.splitext()`可以直接让你得到文件扩展名**，很多时候非常方便：

```
>>> os.path.splitext('/path/to/file.txt')
('/path/to/file', '.txt')
```

这些合并、拆分路径的函数并**不要求目录和文件要真实存在**，它们只对字符串进行操作。

文件操作使用下面的函数。假定当前目录下有一个`test.txt`文件：

```
# 对文件重命名:
>>> os.rename('test.txt', 'test.py')
# 删掉文件:
>>> os.remove('test.py')
```

但是**复制文件的函数居然在`os`模块中不存在**！原因是复制文件并非由操作系统提供的系统调用。理论上讲，我们通过上一节的读写文件可以完成文件复制，只不过要多写很多代码。

幸运的是**`shutil`模块提供了`copyfile()`的函数**，你还可以在`shutil`模块中找到很多实用函数，它们可以看做是`os`模块的补充。

###　过滤文件

比如我们要列出当前目录下的所有目录，只需要一行代码：

```
>>> [x for x in os.listdir('.') if os.path.isdir(x)]
['.lein', '.local', '.m2', '.npm', '.ssh', '.Trash', '.vim', 'Applications', 'Desktop', ...]
```

要列出所有的`.py`文件，也只需一行代码：

```
>>> [x for x in os.listdir('.') if os.path.isfile(x) and os.path.splitext(x)[1]=='.py']
['apis.py', 'config.py', 'models.py', 'pymonitor.py', 'test_db.py', 'urls.py', 'wsgiapp.py']
```

是不是非常简洁？

### 小结

Python的`os`模块封装了操作系统的目录和文件操作，**要注意这些函数有的在`os`模块中，有的在`os.path`模块中**。



## 序列化



### pickle

在程序运行的过程中，所有的变量都是在内存中，比如，定义一个dict：

```
d = dict(name='Bob', age=20, score=88)
```

可以随时修改变量，比如把`name`改成`'Bill'`，但是一旦程序结束，变量所占用的内存就被操作系统全部回收。如果没有把修改后的`'Bill'`存储到磁盘上，下次重新运行程序，变量又被初始化为`'Bob'`。

我们**把变量从内存中变成可存储或传输的过程称之为序列化**，在Python中叫**pickling**，在其他语言中也被称之为**serialization，marshalling，flattening**等等，都是一个意思。

序列化之后，就可以把序列化后的内容写入磁盘，或者通过网络传输到别的机器上。

反过来，**把变量内容从序列化的对象重新读到内存里称之为反序列化，即unpickling**。

Python提供了`pickle`模块来实现序列化。

首先，我们尝试把一个对象序列化并写入文件：

```
>>> import pickle
>>> d = dict(name='Bob', age=20, score=88)
>>> pickle.dumps(d)
b'\x80\x03}q\x00(X\x03\x00\x00\x00ageq\x01K\x14X\x05\x00\x00\x00scoreq\x02KXX\x04\x00\x00\x00nameq\x03X\x03\x00\x00\x00Bobq\x04u.'
```

`pickle.dumps()`方法把任意对象序列化成一个`bytes`，然后，就可以把这个`bytes`写入文件。或者用另一个方法`pickle.dump()`直接把对象序列化后写入一个file-like Object：

```
>>> f = open('dump.txt', 'wb')
>>> pickle.dump(d, f)
>>> f.close()
```

**看看写入的`dump.txt`文件，一堆乱七八糟的内容，这些都是Python保存的对象内部信息。**

当我们要把对象从磁盘读到内存时，可以先把内容读到一个`bytes`，然后用`pickle.loads()`方法反序列化出对象，也可以直接用`pickle.load()`方法从一个`file-like Object`中直接反序列化出对象。我们打开另一个Python命令行来反序列化刚才保存的对象：

```
>>> f = open('dump.txt', 'rb')
>>> d = pickle.load(f)
>>> f.close()
>>> d
{'age': 20, 'score': 88, 'name': 'Bob'}
```

变量的内容又回来了！

当然，这个变量和原来的变量是完全不相干的对象，它们只是内容相同而已。

Pickle的问题和所有其他编程语言特有的序列化问题一样，就是**它只能用于Python，并且可能不同版本的Python彼此都不兼容，因此，只能用Pickle保存那些不重要的数据，不能成功地反序列化也没关系。**

### JSON

如果我们要在不同的编程语言之间传递对象，就必须把对象序列化为标准格式，比如XML，但**更好的方法是序列化为JSON，因为JSON表示出来就是一个字符串，可以被所有语言读取，也可以方便地存储到磁盘或者通过网络传输**。JSON**不仅是标准格式，并且比XML更快，而且可以直接在Web页面中读取，非常方便**。

JSON表示的对象就是标准的JavaScript语言的对象，JSON和Python内置的数据类型对应如下：

| JSON类型   | Python类型 |
| :--------- | :--------- |
| {}         | dict       |
| []         | list       |
| "string"   | str        |
| 1234.56    | int或float |
| true/false | True/False |
| null       | None       |

Python内置的`json`模块提供了非常完善的Python对象到JSON格式的转换。我们先看看如何**把Python对象变成一个JSON**：

```
>>> import json
>>> d = dict(name='Bob', age=20, score=88)
>>> json.dumps(d)
'{"age": 20, "score": 88, "name": "Bob"}'
```

**`dumps()`方法返回一个`str`，内容就是标准的JSON**。类似的，`dump()`方法可以直接把JSON写入一个`file-like Object`。

**要把JSON反序列化为Python对象，用`loads()`或者对应的`load()`方法，前者把JSON的字符串反序列化，后者从`file-like Object`中读取字符串并反序列化：**

```
>>> json_str = '{"age": 20, "score": 88, "name": "Bob"}'
>>> json.loads(json_str)
{'age': 20, 'score': 88, 'name': 'Bob'}
```

由于JSON标准规定JSON编码是UTF-8，所以我们总是能正确地在Python的`str`与JSON的字符串之间转换。

### JSON进阶

Python的`dict`对象可以直接序列化为JSON的`{}`，不过，很多时候，我们更喜欢用`class`表示对象，比如定义`Student`类，然后序列化：

```
import json

class Student(object):
    def __init__(self, name, age, score):
        self.name = name
        self.age = age
        self.score = score

s = Student('Bob', 20, 88)
print(json.dumps(s))
```

运行代码，毫不留情地得到一个`TypeError`：

```
Traceback (most recent call last):
  ...
TypeError: <__main__.Student object at 0x10603cc50> is not JSON serializable
```

错误的原因是`Student`对象不是一个可序列化为JSON的对象。

如果连`class`的实例对象都无法序列化为JSON，这肯定不合理！

别急，我们仔细看看`dumps()`方法的参数列表，可以发现，除了第一个必须的`obj`参数外，`dumps()`方法还提供了一大堆的可选参数：

https://docs.python.org/3/library/json.html#json.dumps

这些可选参数就是让我们来定制JSON序列化。**前面的代码之所以无法把`Student`类实例序列化为JSON，是因为默认情况下，`dumps()`方法不知道如何将`Student`实例变为一个JSON的`{}`对象**。

可选参数`default`就是把任意一个对象变成一个可序列为JSON的对象，我们只需要为`Student`专门写一个转换函数，再把函数传进去即可：

```python
def student2dict(std):
    return {
        'name': std.name,
        'age': std.age,
        'score': std.score
    }
```

这样，`Student`实例首先被`student2dict()`函数转换成`dict`，然后再被顺利序列化为JSON：

```
>>> print(json.dumps(s, default=student2dict))
{"age": 20, "name": "Bob", "score": 88}
```

不过，下次如果遇到一个`Teacher`类的实例，照样无法序列化为JSON。我们可以偷个懒，**把任意`class`的实例变为`dict`：**

```
print(json.dumps(s, default=lambda obj: obj.__dict__))
```

**因为通常`class`的实例都有一个`__dict__`属性，它就是一个`dict`，用来存储实例变量。也有少数例外，比如定义了`__slots__`的class。**

同样的道理，如果我们要把JSON反序列化为一个`Student`对象实例，

* `loads()`方法首先转换出一个`dict`对象，
* 然后，我们传入的`object_hook`函数负责把`dict`转换为`Student`实例：

```
def dict2student(d):
    return Student(d['name'], d['age'], d['score'])
```

运行结果如下：

```
>>> json_str = '{"age": 20, "score": 88, "name": "Bob"}'
>>> print(json.loads(json_str, object_hook=dict2student))
<__main__.Student object at 0x10cd3c190>
```

打印出的是反序列化的`Student`实例对象。



### 小结

Python语言特定的序列化模块是`pickle`，**但如果要把序列化搞得更通用、更符合Web标准，就可以使用`json`模块**。

`json`模块的`dumps()`和`loads()`函数是定义得非常好的接口的典范。当我们使用时，只需要传入一个必须的参数。但是，当默认的序列化或反序列机制不满足我们的要求时，我们又可以传入更多的参数来定制序列化或反序列化的规则，既做到了接口简单易用，又做到了充分的扩展性和灵活性。



## 进程和线程

很多同学都听说过，现代操作系统比如Mac OS X，UNIX，Linux，Windows等，都是支持“多任务”的操作系统。

“多任务”: 操作系统可以同时运行多个任务。还有很多任务悄悄地在后台同时运行着，只是桌面上没有显示而已。



**单核CPU，也可以执行多任务**。由于CPU执行代码都是顺序执行的，那么，单核CPU是怎么执行多任务的呢？

答案就是操作系统轮流让各个任务交替执行，任务1执行0.01秒，切换到任务2，任务2执行0.01秒，再切换到任务3，执行0.01秒……这样反复执行下去。表面上看，**每个任务都是交替执行的**，但是，**由于CPU的执行速度实在是太快了，我们感觉就像所有任务都在同时执行一样**。



**真正的并行执行多任务只能在多核CPU上实现**，但是，由于任务数量远远多于CPU的核心数量，所以，操作系统也会自动把很多任务轮流调度到每个核心上执行。



对于操作系统来说，一个任务就是一个进程（Process）。

有些进程还不止同时干一件事，比如Word，它可以同时进行打字、拼写检查、打印等事情。**在一个进程内部，要同时干多件事，就需要同时运行多个“子任务”**，我们把进程内的这些“子任务”称为**线程（Thread）**。

由于每个进程至少要干一件事，所以，**一个进程至少有一个线程**。当然，像Word这种复杂的进程可以有多个线程，多个线程可以同时执行，**多线程的执行方式**和多进程是一样的，也是**由操作系统在多个线程之间快速切换，让每个线程都短暂地交替运行，看起来就像同时执行一样。当然，真正地同时执行多线程需要多核CPU才可能实现**。

我们前面编写的所有的Python程序，都是执行单任务的进程，也就是只有一个线程。如果我们**要同时执行多个任务怎么办？**

有两种解决方案：

* 启动多个进程，每个进程虽然只有一个线程，但多个进程可以一块执行多个任务。
* 启动一个进程，在一个进程内启动多个线程，这样，多个线程也可以一块执行多个任务。
* 启动多个进程，每个进程再启动多个线程，这样同时执行的任务就更多了，当然这种模型更复杂，实际很少采用。

总结一下就是，多任务的实现有3种方式：

- 多进程模式；
- 多线程模式；
- 多进程+多线程模式。

同时执行多个任务通常各个任务之间并不是没有关联的，而是需要相互通信和协调，有时，任务1必须暂停等待任务2完成后才能继续执行，有时，任务3和任务4又不能同时执行，所以，多进程和多线程的程序的复杂度要远远高于我们前面写的单进程单线程的程序。

因为复杂度高，调试困难，所以，不是迫不得已，我们也不想编写多任务。但是，有很多时候，没有多任务还真不行。想想在电脑上看电影，就必须由一个线程播放视频，另一个线程播放音频，否则，单线程实现的话就只能先把视频播放完再播放音频，或者先把音频播放完再播放视频，这显然是不行的。

Python既支持多进程，又支持多线程，我们会讨论如何编写这两种多任务程序。

### 小结

线程是最小的执行单元，而进程由至少一个线程组成。如何调度进程和线程，完全由操作系统决定，程序自己不能决定什么时候执行，执行多长时间。

多进程和多线程的程序涉及到同步、数据共享的问题，编写起来更复杂。



## 多进程















# sides

## string.split()

通过指定分隔符对字符串进行切片，如果参数 num 有指定值，则分隔 num+1 个子字符串

### 语法

```
str.split(str="", num=string.count(str)).
```

### 参数

- str -- 分隔符，默认为所有的空字符，包括空格、换行(\n)、制表符(\t)等。
- num -- 分割次数。默认为 -1, 即分隔所有。

### 返回值

返回分割后的字符串列表。

以下实例展示了 split() 函数的使用方法：

### 实例(Python 2.0+)

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
str = "Line1-abcdef \nLine2-abc \nLine4-abcd";
print str.split( );       # 以空格为分隔符，包含 \n
print str.split(' ', 1 ); # 以空格为分隔符，分隔成两个
```

输出结果：

```
['Line1-abcdef', 'Line2-abc', 'Line4-abcd']
['Line1-abcdef', '\nLine2-abc \nLine4-abcd']
```



##

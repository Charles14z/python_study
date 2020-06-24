



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

**给实例绑定一个属性：**

```python
>>> class Student(object):
    	pass
>>> s = Student()
>>> s.name = 'Michael' # 动态给实例绑定一个属性
>>> print(s.name)
Michael
```

还可以尝试给实例绑定一个方法：

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

**为了给所有实例都绑定方法，可以给class绑定方法：**

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

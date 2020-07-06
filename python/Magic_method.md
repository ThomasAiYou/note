# Magic Method

## __slots__

​		当程序创建了大量的类，为此占用了大量的内存时候，可以在类定义中增加slots属性，以此来大量减少对内存的使用。当定义了slots属性后，python会针对实例采用一种更加紧凑的内部表示方式。不再每个实例创建一个dict。使用slots可以阻止用户添加slots以外的属性。

```python
class Person
		__slots__=("name")
  	
    def __init__(self,name):
      	self.name = name
```

​		上述Person类能添加name属性，不能动态添加其他属性。

```python
class Person:
    __slots__ = ("name")
    
    def __init__(self,name):
        self.name = name

class Teacher(Person):
    __slots__ = ("age")
```

​		当遇到继承的时候，子类的属性集合是子类slots加上父类slots。
## Python中的for循环、可迭代对象、迭代器和生成器

注：本篇是根据教程学习记录的笔记，部分内容与教程是相同的，因为转载需要填链接，但是没有，所以填的原创，如果侵权会直接删除。

问题：

- 之前在学习list和dict相关的知识时，遇到了一个常见的问题：如何在遍历list或dict的时候正常删除？例如我们在遍历dict的时候删除，会报错：RuntimeError: dictionary changed size during iteration；而在遍历list的时候删除，会有部分元素删除不完全。由这个问题又引发了我对另一个问题的思考：我们通过for循环去遍历一个list或dict时，具体是如何for的呢？即for循环的本质是什么？
- 在查阅了相关资料后，我认识到这是一个和迭代器相关的问题，所以借此机会来详细认识一下Python中的for循环、可迭代对象、迭代器和生成器

### 1. 迭代

“迭代是重复反馈过程的活动，其目的通常是为了逼近所需目标或结果。”在Python中，可迭代对象、迭代器、for循环都是和“迭代”密切相关的知识点。

#### 1.1 可迭代对象Iterable

- 在Python中，称可以迭代的对象为**可迭代对象**。要判断一个类是否可迭代，只需要判断这个类是否为Iterable类的实例即可：

  ```python
  >>> from collections.abc import Iterable
  >>> isinstance([], Iterable)
  True
  >>> isinstance(123, Iterable)
  False
  ```

- 上述提供了一个判断对象是否为可迭代对象的方法，那么一个对象怎么才是可迭代对象呢——只需要该对象的类实现了\_\_iter\_\_()方法即可：

  ```python
  >>> class A:
      	pass
  >>> isinstance(A(), Iterable)
  False
  >>> class B:
      	def __iter__(self):
          	pass     
  >>> isinstance(B(), Iterable)
  True
  ```

  由此可见，只要一个类实现了\_\_iter\_\_()方法，那么这个类的实例对象就是可迭代对象。注意这里的\_\_iter\_\_()方法可以没有任何内容。

#### 1.2 迭代器Iterator

- 在Python中，通过Iterator类与迭代器相对应。相较于可迭代对象，迭代器只是多实现了一个\_\_next\_\_()方法:

  ```python
  >>> from collections.abc import Iterator
  >>> class C:
          def __iter__(self):
              pass
  
          def __next__(self):
              pass
  >>> isinstance(C(), Iterator)
  True
  ```

  显然，迭代器一定是可迭代对象（因为迭代器同时实现了\_\_iter\_\_()方法和\_\_next\_\_()方法），而可迭代对象不一定是迭代器。

- 我们来看一下内建类型中的可迭代对象是否为迭代器：

  ```python
  >>> isinstance(C(), Iterator)
  True
  >>> isinstance([], Iterable)
  True
  >>> isinstance([], Iterator)
  False
  >>> isinstance('123', Iterable)
  True
  >>> isinstance('123', Iterator)
  False
  >>> isinstance({}, Iterable)
  True
  >>> isinstance({}, Iterator)
  False
  ```

  由此可见，str、list、dict对象都是可迭代对象，但它们都不是迭代器。

- 至此，我们对可迭代对象和迭代器有了一个基本概念上的认识，也知道了有\_\_iter\_\_()和\_\_next\_\_()这两种方法。但是这两个魔法方法究竟是如何使用的呢？它们和for循环又有什么关系呢？

#### 1.3 for循环

##### 1.3.1 iter()方法和next()方法

- iter()方法和next()方法都是Python提供的内置方法。对对象使用iter()方法会调用对象的\_\_iter\_\_()方法，对对象使用next()方法会调用对象的\_\_next\_\_()方法。下面我们具体看一下它们之间的关系。

##### 1.3.2 iter()和\_\_iter\_\_()

- \_\_iter\_\_()方法的作用就是返回一个迭代器，一般我们可以通过内置函数iter()来调用对象的\_\_iter\_\_()方法

- 1.2中举的例子，只是简单的实现了\_\_iter\_\_()方法，但函数体直接被pass掉了，本质上是没有实现迭代功能的，现在我们来看一下\_\_iter\_\_()正常使用时的例子：

  ```python
  >>> class A:
      def __iter__(self):
          print('执行A类的__iter__()方法')
          return B()
  
  >>> class B:
      def __iter__(self):
          print('执行B类的__iter__()方法')
          return self
      def __next__(self):
          pass
  
      
  >>> a = A()
  >>> a1 = iter(a)
  执行A类的__iter__()方法
  >>> b = B()
  >>> b1 = iter(b)
  执行B类的__iter__()方法
  ```

  可以看到，对于类A，我们为它的\_\_iter\_\_()方法设置了返回值为B()，而B()就是一个迭代器；而对于类B，我们在它的\_\_iter\_\_()方法中直接返回了它的实例self，因为它的实例本身就是可迭代对象。当然这里我们也可以返回其他的迭代器，但是如果\_\_iter\_\_()方法返回的是一个非迭代器，那么当我们调用iter()方法时就会报错：
  
  ```python
  >>> class C:
          def __iter__(self):
              pass
  
  >>> iter(C())
  Traceback (most recent call last):
    File "<pyshell#4>", line 1, in <module>
      iter(C())
  TypeError: iter() returned non-iterator of type 'NoneType'
  
  
  >>> class D:
          def __iter__(self):
              return []
  
  >>> iter(D())
  Traceback (most recent call last):
    File "<pyshell#8>", line 1, in <module>
      iter(D())
  TypeError: iter() returned non-iterator of type 'list'
  ```

##### 1.3.3 next()和\_\_next\_\_()

- \_\_next\_\_()方法的作用是返回**遍历过程**中的下一个元素，如果没有下一个元素，则会抛出StopIteration异常，一般我们可以通过内置函数next()来调用对象的\_\_next\_\_()方法

- 下面我们以list对象为例，来看一下next是如何遍历的：

  ```python
  >>> l1 = [1, 2, 3]
  >>> next(l1)
  Traceback (most recent call last):
    File "<pyshell#1>", line 1, in <module>
      next(l1)
  TypeError: 'list' object is not an iterator
  ```

  可以看到，当我们直接对列表对象l1使用next()方法时，会报错'list' object is not an iterator，显然list对象并不是迭代器，也就是说它没有实现\_\_next\_\_()方法，那么我们怎么才能去”对一个列表对象使用next()“呢——根据我们前面介绍的\_\_iter\_\_()方法，我们知道它会返回一个迭代器，而迭代器是实现了\_\_next\_\_()方法的，所以我们可以先对list对象使用iter\_\_()，获取到它对应的迭代器，然后对这个迭代器使用next()就可以了：

  ```python
  >>> l1 = [1, 2, 3]
  >>> l1_iter = iter(l1)
  >>> type(l1_iter)
  <class 'list_iterator'>
  >>> next(l1_iter)
  1
  >>> next(l1_iter)
  2
  >>> next(l1_iter)
  3
  >>> next(l1_iter)
  Traceback (most recent call last):
    File "<pyshell#6>", line 1, in <module>
      next(l1_iter)
  StopIteration
  ```

- 思考：\_\_next\_\_()为什么要不停地去取出元素，并且在最后去抛出异常，而不是通过对象的长度相关信息来确定调用次数？

  个人认为是因为我们可以通过next()去手动调用对象的\_\_next\_\_()方法，而在next()中并没有判断对象的长度，所以需要在\_\_next\_\_()去处理

##### 1.3.4 自定义类实现\_\_iter\_\_()和\_\_next\_\_()

下面我们试着通过实现自定义一下list的迭代过程：

1. 首先我们定义一个类A，它是一个可迭代对象，\_\_iter\_\_()方法会返回一个迭代器B()，并且还拥有一个成员变量m_Lst：

   ```python
   >>> class A:
           def __init__(self, lst):
               self.m_Lst = lst
           def __iter__(self):
               return B(self.m_Lst)
   ```

2. 对于迭代器的类B，我们实现它的\_\_iter\_\_()方法和\_\_next\_\_()方法，注意在\_\_next\_\_()方法中我们需要抛出StopIteration异常。此外，它拥有两个成员变量self.m_Lst和self.m_Index用于迭代遍历：

   ```python
   >>> class B:
           def __init__(self, lst):
               self.m_Lst = lst
               self.m_Index= 0
           def __iter__(self):
               return self
           def __next__(self):
               try:
                   value = self.m_Lst[self.m_Index]
                   self.m_Index += 1
                   return value
               except IndexError:
                   raise StopIteration()
   ```

3. 至此，我们已经完成了迭代器的准备工作，下面我们来实践一下迭代吧，为了更好地展示这个过程，我们可以加上一些打印：

   ```python
   >>> class A:
           def __init__(self, lst):
               self.m_Lst = lst
           def __iter__(self):
               print('call A().__iter__()')
               return B(self.m_Lst)
           
   >>> class B:
           def __init__(self, lst):
               self.m_Lst = lst
               self.m_Index= 0
           def __iter__(self):
               print('call B().__iter__()')
               return self
           def __next__(self):
               print('call B().__next__()')
               try:
                   value = self.m_Lst[self.m_Index]
                   self.m_Index += 1
                   return value
               except IndexError:
                   print('call B().__next__() except IndexError')
                   raise StopIteration()
                   
   >>> l = [1, 2, 3]
   >>> a = A(l)
   >>> a_iter = iter(a)
   call A().__iter__()
   >>> next(a_iter)
   call B().__next__()
   1
   >>> next(a_iter)
   call B().__next__()
   2
   >>> next(a_iter)
   call B().__next__()
   3
   >>> next(a_iter)
   call B().__next__()
   call B().__next__() except IndexError
   Traceback (most recent call last):
     File "<pyshell#5>", line 11, in __next__
       value = self.m_Lst[self.m_Index]
   IndexError: list index out of range
   
   During handling of the above exception, another exception occurred:
   
   Traceback (most recent call last):
     File "<pyshell#12>", line 1, in <module>
       next(a_iter)
     File "<pyshell#5>", line 16, in __next__
       raise StopIteration()
   StopIteration
   ```

4. 可以看到，我们借助iter()和next()方法能够很好地将整个遍历的过程展示出来。至此，我们对可迭代对象、迭代器以及\_\_iter\_\_()和\_\_next\_\_()都有了一定的认识，那么，for循环和它们有什么关系呢？

##### 1.3.5 探究for循环

- for循环是我们使用频率最高的操作之一，我们一般会用它来遍历一个容器（列表、字典等），这些容器都有一个共同的特点——都是可迭代对象。那么对于我们自定义的类A，它的实例对象a应该也可以通过for循环来遍历：

  ```python
  >>> for i in a:
      	print(i)
  call A().__iter__()
  call B().__next__()
  1
  call B().__next__()
  2
  call B().__next__()
  3
  call B().__next__()
  call B().__next__() except IndexError
  
  >>> for i in a:
  	    pass
  call A().__iter__()
  call B().__next__()
  call B().__next__()
  call B().__next__()
  call B().__next__()
  call B().__next__() except IndexError
  ```

- 通过打印，我们可以清楚的看到：对一个可迭代对象使用for循环进行遍历时，for循环会调用该对象的\_\_iter\_\_()方法来获取到迭代器，然后循环调用该迭代器的\_\_next\_\_()方法，依次获取下一个元素，并且最后会捕获StopIteration异常（这里可以尝试在类B的\_\_next\_\_()方法最后只捕获IndexError而不抛出StopIteration，则for循环此时会无限循环）

- 既然我们提到了for循环会自动去捕获StopIteration异常，当没有捕获到StopIteration异常时会无限循环，那么我们是否可以用while循环来模拟一下这个过程呢？

  ```python
  >>> while True:
          try:
              i = next(a_iter)
              print(i)
          except StopIteration:
              print('except StopIteration')
              break
  
  call B().__next__()
  1
  call B().__next__()
  2
  call B().__next__()
  3
  call B().__next__()
  call B().__next__() except IndexError
  except StopIteration
  ```

  到这里，大家应该对for对可迭代对象遍历的过程有了一定的了解，想要更深入了解的话可以结合源码进一步学习（本次学习分享主要是结合实际代码对一些概念进行讲解，并未涉及到相应源码）。

#### 2 生成器

迭代器和生成器总是会被同时提起，那么它们之间有什么关联呢——生成器是一种特殊的迭代器。

#### 2.1 获取生成器

- 当一个函数体内使用yield关键字时，我们就称这个函数为生成器函数；当我们调用这个生成器函数时，Python会自动在返回的对象中添加\_\_iter\_\_()方法和\_\_next\_\_()方法，它返回的对象就是一个生成器。

- 代码示例：

  ```python
  >>> from collections.abc import Iterator
  >>> def generator():
          print('first')
          yield 1
          print('second')
          yield 2
          print('third')
          yield 3
  
  >>> gen = generator()
  >>> isinstance(gen, Iterator)
  True
  ```

#### 2.2 next(生成器)

- 既然生成器是一种特殊的迭代器，那么我们对它使用一下next()方法：

  ```python
  >>> next(gen)
  first
  1
  >>> next(gen)
  second
  2
  >>> next(gen)
  third
  3
  >>> next(gen)
  Traceback (most recent call last):
    File "<pyshell#19>", line 1, in <module>
      next(gen)
  StopIteration
  ```

  这里我想给这个generator()函数加一个return，最后会在抛出异常时打印这个返回值（这里我对Python异常相关的知识了解比较少，不太清楚这个问题，以后再补充吧）：

  ```python
  >>> from collections.abc import Iterator
  >>> def generator():
          print('first')
          yield 1
          print('second')
          yield 2
          print('third')
          yield 3
          return 'end'
  
  >>> gen = generator()
  >>> isinstance(gen, Iterator)
  True
  >>> next(gen)
  first
  1
  >>> next(gen)
  second
  2
  >>> next(gen)
  third
  3
  >>> next(gen)
  Traceback (most recent call last):
    File "<pyshell#7>", line 1, in <module>
      next(gen)
  StopIteration: end
  ```

- 可以看到，当我们对生成器使用next()方法时，生成器会执行到下一个yield为止，并且返回yield后面的值；当我们再次调用next(生成器)时，会继续向下执行，直到下一个yield语句；执行到最后再没有yield语句时，就会抛出StopIteration异常

#### 2.3 生成器和迭代器

- 通过上面的过程，我们知道了生成器本质上就是一种迭代器，但是除了yield的特殊外，生成器还有什么特殊点呢——惰性计算。

- 这里的惰性计算是指：当我们调用next(生成器)时，每次调用只会产生一个值，这样的好处就是，当遍历的元素量很大时，我们不需要将所有的元素一次获取，而是每次只取其中的一个元素，可以节省大量内存。（个人理解：这里注意和上面的迭代器的next()区别开，对于迭代器，虽然每次next()时，也只会返回一个值，但是本质上我们已经把所有的值存储在内存中了（比如类A和类B的self.m_Lst），但是对于生成器，内存中并不会将所有的值先存储起来，而是每次调用next()就获取一个值）

- 下面我们来看一个实际的例子：输出10000000以内的所有偶数（注意，如果实际业务环境下需要存储，那就根据实际情况来，这里只是针对两者的区别进行讨论）

  首先我们通过迭代器来实现：（这里直接使用列表）

  ```python
  >>> def iterator():
          lst = []
          index = 0
          while index <= 10000000:
              if index % 2 == 0:
                  print(index)
                  lst.append(index)
              index += 1
          return lst
  
  >>> result = iterator()
  ```

  然后通过生成器来实现：

  ```python
  >>> def generator():
          index = 0
          while index <= 10000000:
              if index % 2 == 0:
                  yield index
              index += 1
              
  >>> gen = generator()
  >>> next(gen)
  0
  >>> next(gen)
  2
  >>> next(gen)
  4
  >>> next(gen)
  6
  >>> next(gen)
  8
  ```

- 由于采取了惰性运算，生成器也有它的不足：对于列表对象、字典对象等可迭代对象，我们可以通过len()方法直接获取其长度，但是对于生成器对象，我们只知道当前元素，自然就不能获取到它的长度信息了。

- 下面我们总结一下生成器和迭代器的相同点和不同点：

  1. 生成器是一种特殊的迭代器；
  2. 迭代器会通过return来返回值，而生成器则是通过yield来返回值，对生成器使用next()方法，会在每一个yield语句处停下；
  3. 迭代器会存储所有的元素，但是生成器采用的是惰性计算，只知道当前元素。

#### 2.4 生成器解析式

- 列表解析式是我们常用的一种解析式：（类似的还有字典解析式、集合解析式）

  ```pyth
  >>> lst = [i for i in range(10) if i % 2 == 1]
  >>> lst
  [1, 3, 5, 7, 9]
  ```

- 而生成器解析式和列表解析式类似，我们只需要将[]更换为()即可：（把元组解析式给抢了，hh）

  ```python
  >>> gen = (i for i in range(10) if i % 2 == 1)
  >>> gen
  <generator object <genexpr> at 0x00000193E2945A80>
  >>> next(gen)
  1
  >>> next(gen)
  3
  >>> next(gen)
  5
  >>> next(gen)
  7
  >>> next(gen)
  9
  >>> next(gen)
  Traceback (most recent call last):
    File "<pyshell#11>", line 1, in <module>
      next(gen)
  StopIteration
  ```

- 至此，我们就有了生成器的两种创造方式：

  1. 生成器函数（yield）返回一个生成器
  2. 生成器解析式返回一个生成器

### 3 解决问题

- 最后回到我们最初的问题：如何在遍历list或dict的时候正常删除？

- 首先我们来探寻一下出错的原因，以list对象为例：

  ```python
  >>> lst = [1, 2, 3]
  >>> for i in lst:
      	print(i)
  	    lst.remove(i)
  
  1
  3
  ```

  可以看到，我们在遍历打印列表元素的同时删除当前元素，实际的输出和我们需要的输出并不一样。以下是个人理解（想更准确地解答这个问题可能需要进一步结合源码）：

  1. remove删除列表元素时，列表元素的索引会发生变化（这是因为Python底层列表是通过数组实现的，remove方法删除元素时需要挪动其他元素，具体分析我后续会补充相关源码学习笔记，这里先了解即可）

  2. 类比我们自定义实现的迭代器，可以看到我们会在\_\_next\_\_()方法中对索引进行递增：

     ```python
     >>> class A:
             def __init__(self, lst):
                 self.m_Lst = lst
             def __iter__(self):
                 print('call A().__iter__()')
                 return B(self.m_Lst)
             
     >>> class B:
             def __init__(self, lst):
                 self.m_Lst = lst
                 self.m_Index= 0
             def __iter__(self):
                 print('call B().__iter__()')
                 return self
             def __next__(self):
                 print('call B().__next__()')
                 try:
                     value = self.m_Lst[self.m_Index]
                     self.m_Index += 1
                     return value
                 except IndexError:
                     print('call B().__next__() except IndexError')
                     raise StopIteration()
     ```

     那么我们可以猜测：列表对象对应的迭代器，应该也是会有一个索引成员变量，用于在\_\_next\_\_()方法中进行定位（这里没看过源码，只是个人猜想）

  3. 当我们使用for循环遍历列表对象时，实际上是通过next()方法对其对应的迭代器进行操作，此时由于remove()方法的调用，导致列表元素的索引发生了改变（原来元素3的索引是2，删除元素2之后索引变为了1），所以在\_\_next\_\_()方法中，此时需要遍历的元素索引为1，而元素3顶替了这个位置，所以最后的输出为1，3。

- dict和list类似，不过在遍历时删除dict中的元素时会直接报错，具体原因大家也可以自行分析。

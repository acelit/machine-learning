廖雪峰python学习笔记--基本数据结构

1.list--[]列表，有序元素的集合，可以存储不同类型的数据，包括list本身。
list = ['a','b',3]
相关操作：
len--list长度
append(ele)--追加到尾部
insert(n,ele)--插入到给定位置n
pop(n)--pop()从尾部删除元素，和append相对应，返回被删除的元素，也可指定删除元素的位置n
sort()--针对统一类型的元素可进行排序操作，不是同一类型会报错。
注意：可以从头到尾也可以从尾到头访问元素。

2.tuple--()元组，和list类似，有序元素的集合，但是tuple的元素一旦初始化后就不能改变（基本数据类型和值不能改变，list类型不变但是值可以变），因此不能使用append(),insert()，pop()方法。因此，为了代码更安全，尽量使用tuple代替list。

>>> tp = (1,2,['a','b'])
>>> tp[2][0] = 'x'
>>> tp[2][1] = 'y'

tuple还有一个有意思的地方：定义单个元素的tuple时需要注意：
>>> tp = (1)
>>> tp
1
>>> tp = (1,)
>>> tp
(1,)
第一种方式是不对的(系统会认为是一个数1)，需要在元素后面再加一个逗号菜才是tuple类型的数据。

3.dict--{}字典,无序键-值（key-value）对，key必须是相异的不可变对象，可以是字符/数字，但是不能为list。
>>> d = {'a':10,'b':20,'c':30}
>>> d['a']
10

相关操作：
dist不具备append/insert操作，具有pop操作。
可以直接通过设置key-value来添加新的键值对，d['d'] = 40
由于dist中键值对是无序的，因此新添加的键值对位置不确定。
此外，可以通过 key in dict 或者dict.get(key)的操作来判断key是否存在。

和list比较，dict有以下几个特点：

    查找和插入的速度极快，不会随着key的增加而增加；
    需要占用大量的内存，内存浪费多。

而list相反：

    查找和插入的时间随着元素的增加而增加；
    占用空间小，浪费内存很少。

所以，dict是用空间来换取时间的一种方法。

4.set--{[]},集合，和dict一样是无序key，因此key也必须是不同的不可变类型的值。
>>> s1 = set([1,2,3])
>>> s1
{1, 2, 3}
注意：set的输入是一个list，但是并不意味值key值为list类型，可以通过add()操作增加一个元素，也可以通过update([])来增加多个元素，对于已经存在的元素不再添加，因此set可以用来去除重复元素（Hash）。

可以通过remove()来删除某个元素，如果不存在则引发 KeyError ，discard()和remove()类似，但是不存在该元素时不会有错误提示，pop()用于随机删除某个元素（不确定），clear()用于清除该集合。

不可以用下标或者key值来访问集合元素。

最后，set还能进行交(&)并(|)差(-)对称差(^)集合运算，功能很强大。


------------------------------------------------------------------
不可变对象性质(char不可变，list可变)
对于不变对象来说，调用对象自身的任意方法，也不会改变该对象自身的内容。相反，这些方法会创建新的对象并返回，这样，就保证了不可变对象本身永远是不可变的。
>>> a = ['c', 'b', 'a']
>>> a.sort()
>>> a
['a', 'b', 'c']

>>> a = 'abc'
>>> a.replace('a', 'A')
'Abc'
>>> a
'abc'

虽然tuple也是不可变对象，但是如果tuple中如果含有list元素，是不能作为dict/set的key的。
>>> dict = {(1,2,3):1,'b':2}
>>> dict
{'b': 2, (1, 2, 3): 1}


>>> dict = {(1,[2,3]):1,'b':2}
Traceback (most recent call last):
  File "/usr/lib/x86_64-linux-gnu/gedit/plugins/pythonconsole/console.py", line 377, in __run
    r = eval(command, self.namespace, self.namespace)
  File "<string>", line 1
    dict = {(1,[2,3]):1,'b':2}
         ^
SyntaxError: invalid syntax

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/usr/lib/x86_64-linux-gnu/gedit/plugins/pythonconsole/console.py", line 381, in __run
    exec(command, self.namespace)
  File "<string>", line 1, in <module>
TypeError: unhashable type: 'list'

从错误信息可知，tuple中含有list元素时是可变对象，不可作为key。



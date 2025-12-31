# Flask装饰器执行顺序

当一个函数被多个装饰器修饰时，需要注意装饰器的执行顺序问题。

装饰器的装饰顺序是由内向外，而执行顺序则是由外向内。（这与函数和函数调用的机制有关）

## 执行顺序

例子:

```python
def decorator_a(func):
  print('Get in decorator_a')
  def inner_a(*args, **kwargs):
    print('Get in inner_a')
    return func(*args, **kwargs)
  return inner_a
 
def decorator_b(func):
  print('Get in decorator_b')
  def inner_b(*args, **kwargs):
    print('Get in inner_b')
    return func(*args, **kwargs)
  return inner_b
 
@decorator_b
@decorator_a
def f(x):
  print('Get in f')
  return x * 2

f(1)
```

如果按照一般思维，可能会认为先执行decorator_a，再执行decorator_b，执行结果应该如下:

```python
Get in decorator_a
Get in inner_a
Get in decorator_b
Get in inner_b
Get in f
```

但实际执行结果如下:

```python
Get in decorator_a
Get in decorator_b
Get in inner_b
Get in inner_a
Get in f
```

## 函数和函数调用的区别

在解释上面的问题之前，我们可以先删除代码中的f(1)，再次运行代码，得到的结果如下：

```python
Get in decorator_a
Get in decorator_b
```

这引出了两个重要概念：函数和函数调用。

在上面的代码中，f称为函数，f(1)称为函数调用，后者是对前者传入参数进行计算后的结果。

f是一个函数对象，它本身并不会执行。f(1)才是对函数的调用，这时才会真正执行这个函数对象。

对于decorator_b函数而言，它返回的是函数对象inner_a。

## 装饰器函数在被装饰函数定义好后立即执行

当解释器执行到如下代码时，装饰器函数decorator_a已经被调用了，它以函数f为参数，返回了一个它内部生成的函数inner_a：

```python
@decorator_a
def f(x):
  pass

# 相当于
def f(x):
  pass
f = decorator_a(f)
```

传给f的参数会传递给inner_a。在调用inner_a时，会把收到的参数传递给inner_a里的func，即经过修饰后的f。

## 例子解释

在最开始的代码中，初始化时就已经依次调用了decorator_a和decorator_b，输出Get in decorator_a和Get in decorator_b。此时f实际上是decorator_b里的inner_b。因为f还没有被调用，所以inner_b没有执行。

当执行f(1)时，会导致inner_b被调用，输出Get in inner_b，然后在inner_b内部执行`return func`，此处的func实际上是被decorator_a修饰过的func，所以会调用inner_a，输出Get in inner_a并返回原来f的结果。

由于这种嵌套修饰的机制，在实际应用场景中会先验证是否已登录@login_required，再验证权限是否足够@permission_allowed。

## route装饰器必须在最外层

正确的写法如下：

```python
@app.route('/index', methods=['POST'])
@login_required
def index():
  pass
```

在Flask文档中有如下描述：

* To use the decorator, apply it as innermost decorator to a view function. When applying further decorators, always remember that the [`route()`](https://flask.palletsprojects.com/en/1.0.x/api/#flask.Flask.route) decorator is the outermost.

这是因为如果route装饰器在内层，那么原始的view函数就会被直接传递给add_url_rule，从而跳过登录验证，直接指向最原始的视图函数。

上述装饰器用法等同于：

```python
def index():
  pass
index = login_required(index)
app.add_url_rule('/index', endpoint='index', view_func=index)
```

将route装饰器写在最外层，才能正常使用登录验证、权限设置等功能。
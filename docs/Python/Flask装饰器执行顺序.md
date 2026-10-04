# Flask装饰器执行顺序

当一个函数被多个装饰器修饰时，需要注意装饰器的执行顺序。

装饰器的应用顺序是由内向外（即靠近函数定义的先应用），而调用执行顺序则是由外向内（即最外层装饰器的包装函数先执行）。

## 执行顺序

示例：

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

如果按照一般思维，可能会认为先执行 `decorator_a`，再执行 `decorator_b`，执行结果应该如下：

```text
Get in decorator_a
Get in inner_a
Get in decorator_b
Get in inner_b
Get in f
```

但实际执行结果如下：

```text
Get in decorator_a
Get in decorator_b
Get in inner_b
Get in inner_a
Get in f
```

## 函数与函数调用的区别

在解释上述现象之前，我们先移除代码中的 `f(1)` 调用，再次运行代码，得到的结果如下：

```text
Get in decorator_a
Get in decorator_b
```

这引出了两个重要概念：**函数对象**与**函数调用**。

在上述代码中，`f` 是一个函数对象，它本身不会被执行；而 `f(1)` 是对该函数的调用，此时才会真正执行函数体中的代码。

对于 `decorator_b` 而言，它接收的是经 `decorator_a` 包装后的函数，最终返回的是其内部函数 `inner_b`。

## 装饰器在被装饰函数定义后立即执行

当Python解释器执行到如下代码时，装饰器 `decorator_a` 会被立即调用。它以函数 `f` 为参数，返回其内部定义的函数 `inner_a`：

```python
@decorator_a
def f(x):
    pass

# 相当于
def f(x):
    pass
f = decorator_a(f)
```

传给被装饰函数的参数会传递给 `inner_a`。当 `inner_a` 被调用时，它会将收到的参数传递给内部的 `func`，即传入 `decorator_a` 的原始函数 `f`（未被包装的版本），`inner_a` 本身才是包装后的结果。

## 例子解释

在最开始的代码中，初始化时就已经依次调用了 `decorator_a` 和 `decorator_b`，输出 `Get in decorator_a` 和 `Get in decorator_b`。此时 `f` 实际上是 `decorator_b` 里的 `inner_b`。因为 `f` 还没有被调用，所以 `inner_b` 没有执行。

当执行 `f(1)` 时，会导致 `inner_b` 被调用，输出 `Get in inner_b`，然后在 `inner_b` 内部执行 `return func(*args, **kwargs)`，即调用 `func` 并返回其结果。此处的 `func` 实际上是被 `decorator_a` 修饰过的 `func`，所以会调用 `inner_a`，输出 `Get in inner_a` 并返回原来 `f` 的结果。

由于这种嵌套修饰的机制，在实际应用场景中会先验证是否已登录 `@login_required`，再验证权限是否足够 `@permission_allowed`。要得到这个顺序，需要把 `@login_required` 写在 `@permission_allowed` 上方，越靠上的装饰器，其包装函数越先执行。

## route装饰器必须在最外层

正确的写法如下：

```python
@app.route('/index', methods=['POST'])
@login_required
def index():
    pass
```

在Flask文档中有如下描述：

> To use the decorator, apply it as innermost decorator to a view function. When applying further decorators, always remember that the [`route()`](https://flask.palletsprojects.com/en/1.0.x/api/#flask.Flask.route) decorator is the outermost.

这是因为如果 `route` 装饰器在内层，那么原始的 `view` 函数就会被直接传递给 `add_url_rule`，从而跳过登录验证，直接指向最原始的视图函数。

上述装饰器用法等同于：

```python
def index():
    pass
index = login_required(index)
app.add_url_rule('/index', endpoint='index', view_func=index)
```

将 `route` 装饰器写在最外层，才能正常使用登录验证、权限设置等功能。

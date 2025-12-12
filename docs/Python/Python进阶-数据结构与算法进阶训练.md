# Python进阶-数据结构与算法进阶训练

训练数据筛选,排序,查找,统计,记录等方面的处理能力.

<!--more--> 

## 2.1 如何在列表,字典,集合中根据条件筛选数据

在实际开发中，我们经常需要根据特定条件从数据结构中筛选数据。Python提供了多种高效的方法来实现这一点。

```python
from random import randint

# 原始数据
a = [1, 2, 3, 4, 5, -6, 6, 7, 8, -2, -4]

# 方法1: 使用filter函数
# filter函数接受一个函数和一个可迭代对象，返回满足条件的元素
b = list(filter(lambda x: x > 0, a))
print("使用filter筛选正数:", b)

# 方法2: 使用列表推导式（推荐）
# 列表推导式通常比filter更高效，且代码更易读
c = [i for i in a if i > 0]
print("使用列表推导式筛选正数:", c)

# 字典推导式示例
# 生成一个学生成绩字典
d = {x: randint(60, 100) for x in range(1, 6)}
print("学生成绩字典:", d)

# 筛选出成绩大于90的学生
e = {k: v for k, v in d.items() if v > 90}
print("成绩大于90的学生:", e)

# 集合推导式示例
# 将列表转换为集合并筛选能被3整除的数
f = set(a)
# 注意：原代码中的错误已修正，应该是 x % 3 == 0 而不是 a % 3 == 0
g = {x for x in f if x % 3 == 0}
print("能被3整除的数:", g)
```

**性能优化提示**：
- 列表推导式通常比filter函数更快，因为它在C语言层面进行了优化
- 对于大数据集，考虑使用生成器表达式以节省内存

## 2.2 如何为元组中的每个元素命名,提高程序可读性

在处理元组数据时，使用索引访问元素会降低代码可读性。我们可以通过以下方法为元组元素命名：

```python
# 实际案例：学生信息系统中数据为固定格式:(名字,年龄,性别,邮箱...)
# 为了减少存储开销,对每个学生信息用元组表示:
student_data = ('Jim', 16, 'male', 'jim@qq.com')

# 方案1: 定义数值常量（不推荐）
# 这种方法容易出错且不易维护
NAME_IDX = 0
AGE_IDX = 1
SEX_IDX = 2
EMAIL_IDX = 3

print("方案1 - 使用索引常量:")
print(f"姓名: {student_data[NAME_IDX]}, 年龄: {student_data[AGE_IDX]}")

# 方案2: 使用collections.namedtuple（推荐）
# namedtuple是tuple的子类，为每个位置分配名称，提高代码可读性
from collections import namedtuple

# 创建一个Student类，具有name, age, sex, email四个字段
Student = namedtuple('Student', ['name', 'age', 'sex', 'email'])

# 创建Student实例
s = Student('Jim', 16, 'male', 'jim@qq.com')
print("\n方案2 - 使用namedtuple:")
print(f"学生信息: {s}")
print(f"姓名: {s.name}, 邮箱: {s.email}")
# 验证s确实是tuple的子类
print(f"s是tuple的子类: {isinstance(s, tuple)}")

# namedtuple的优势：
# 1. 不可变性：与tuple一样，namedtuple创建后不可修改
# 2. 内存效率：比定义完整类更节省内存
# 3. 可读性：通过字段名访问元素，代码更清晰
```

## 2.3 如何统计序列中元素的出现频度

统计元素出现频度是数据分析中的常见需求。Python的collections.Counter提供了高效的解决方案。

```python
from collections import Counter
from random import randint

# 示例1: 统计随机序列中元素的出现频度
data = [randint(1, 20) for x in range(20)]
print("随机数据:", data)

# 使用Counter统计频度
c = Counter(data)
print("元素频度统计:", c)

# 获取出现频度最高的3个元素
print("出现频度最高的3个元素:", c.most_common(3))

# 示例2: 统计文本中单词的出现频度
text = "Python is great Python is powerful Python is easy to learn"
words = text.split()
word_counter = Counter(words)
print("\n单词频度统计:", word_counter)
print("最常见的3个单词:", word_counter.most_common(3))

# Counter的其他有用方法
print("所有元素(展开):", list(c.elements()))  # 展开所有元素
print("总元素数:", sum(c.values()))  # 计算总元素数
```

## 2.4 如何根据字典中值的大小,对字典中的项排序

在处理字典数据时，经常需要根据值的大小进行排序。

```python
from random import randint

# 某班成绩以字典形式存储
d = {x: randint(60, 100) for x in 'abcdefghij'}
print("学生成绩:", d)

# 方法1: 利用zip将字典数据转化成元组进行排序
# zip函数将两个序列组合成元组序列
sorted_scores_zip = sorted(zip(d.values(), d.keys()))
print("\n方法1 - 使用zip排序(成绩升序):", sorted_scores_zip)

# 方法2: 传递sorted函数的key参数（推荐）
# 使用lambda函数指定按值排序
sorted_scores_key = sorted(d.items(), key=lambda x: x[1])
print("方法2 - 使用key参数排序(成绩升序):", sorted_scores_key)

# 降序排列
sorted_scores_desc = sorted(d.items(), key=lambda x: x[1], reverse=True)
print("按成绩降序排列:", sorted_scores_desc)

# 按键排序
sorted_by_keys = sorted(d.items(), key=lambda x: x[0])
print("按键排序:", sorted_by_keys)
```

## 2.5 如何快速找到多个字典中的公共键(key)?

在处理多个字典时，有时需要找出它们的公共键。

```python
from random import randint, sample
from functools import reduce

# 模拟三轮球员进球统计
players = 'abcdefghijklmnopqrstuvwxyz'
s1 = {x: randint(1, 4) for x in sample(players, randint(3, 6))}
s2 = {x: randint(1, 4) for x in sample(players, randint(3, 6))}
s3 = {x: randint(1, 4) for x in sample(players, randint(3, 6))}

print("第一轮进球统计:", s1)
print("第二轮进球统计:", s2)
print("第三轮进球统计:", s3)

# 方法1: 使用集合的交集操作
# dict.keys()返回字典键的视图，可以转换为集合
common_keys = set(s1.keys()) & set(s2.keys()) & set(s3.keys())
print("\n方法1 - 使用集合交集找到公共键:", common_keys)

# 方法2: 使用reduce函数（更优雅）
# reduce函数对序列中的元素进行累积操作
common_keys_reduce = reduce(lambda a, b: a & b, map(dict.keys, [s1, s2, s3]))
print("方法2 - 使用reduce找到公共键:", common_keys_reduce)

# 方法3: 使用集合的intersection方法
common_keys_intersection = set(s1.keys()).intersection(s2.keys(), s3.keys())
print("方法3 - 使用intersection方法:", common_keys_intersection)
```

## 2.6 如何让字典保持有序

在某些场景下，我们需要字典保持插入顺序。

```python
# Python 3.7+中，普通字典已经保持插入顺序
# 但在需要明确控制或在旧版本Python中，可以使用OrderedDict

from collections import OrderedDict
from random import randint

# 使用OrderedDict保持插入顺序
d = OrderedDict()
d['Jim'] = (1, 35)   # 第1名，用时35分钟
d['Leo'] = (2, 34)   # 第2名，用时34分钟
d['David'] = (3, 40) # 第3名，用时40分钟

print("OrderedDict中的选手成绩:")
for k in d:
    print(f"  {k}: {d[k]}")

print("OrderedDict内容:", d)

# 普通字典在Python 3.7+中也保持顺序
normal_dict = {}
normal_dict['Jim'] = (1, 35)
normal_dict['Leo'] = (2, 34)
normal_dict['David'] = (3, 40)

print("\n普通字典中的选手成绩:")
for k in normal_dict:
    print(f"  {k}: {normal_dict[k]}")

# OrderedDict的特殊方法
d.move_to_end('Jim')  # 将Jim移到末尾
print("\n将Jim移到末尾后:", d)

# popitem方法可以指定是否从末尾弹出
last_item = d.popitem(last=True)   # 弹出最后一个
first_item = d.popitem(last=False) # 弹出第一个
print(f"弹出的最后一项: {last_item}")
print(f"弹出的第一项: {first_item}")
print("剩余项:", d)
```

## 2.7 如何实现用户的历史记录功能(最多 n 条)?

在交互式程序中，保存用户历史记录是常见的需求。

```python
from random import randint
from collections import deque

# 生成一个随机数作为答案
N = randint(0, 100)
print("欢迎来到猜数字游戏！我已经想好了一个0-100之间的数字。")

# 使用deque实现固定长度的历史记录队列
# maxlen参数指定最大长度，超出时自动删除最早记录
history = deque([], 5)

def guess(k):
    """判断猜测结果"""
    if k == N:
        print('恭喜你，猜对了！')
        return True
    elif k < N:
        print('太小了，请再试试')
    else:
        print('太大了，请再试试')
    return False

# 游戏主循环
# 注意：在实际环境中运行时取消注释以下代码
"""
while True:
    line = input('请输入你猜的数字 (输入h查看历史记录, q退出): ')
    
    # 处理特殊命令
    if line == 'h':
        print(f'历史记录 (最近{len(history)}条): {list(history)}')
        continue
    elif line == 'q':
        print('游戏结束')
        break
    
    # 验证输入是否为数字
    if line.isdigit():
        k = int(line)
        # 添加到历史记录
        history.append(k)
        
        # 判断猜测结果
        if guess(k):
            print(f'答案就是 {N}')
            break
    else:
        print('请输入一个有效的数字')
"""

# deque的优势：
# 1. 双端操作：可以从两端高效地添加和删除元素
# 2. 固定长度：通过maxlen参数自动维护长度
# 3. 性能优异：在两端操作的时间复杂度为O(1)
```

## 新增章节：性能优化与时间复杂度分析

在实际开发中，了解数据结构和算法的时间复杂度对于编写高效代码至关重要。

### 常见操作的时间复杂度

```python
# 列表操作的时间复杂度
# 创建一个大列表用于测试
large_list = list(range(10000))

# O(1) - 常数时间操作
# 访问特定索引的元素
element = large_list[5000]  # O(1)

# O(n) - 线性时间操作
# 在列表末尾添加元素
large_list.append(9999)  # O(1) 平均情况

# 在列表开头插入元素（低效）
# large_list.insert(0, 0)  # O(n) - 需要移动所有后续元素

# O(n) - 查找元素
if 5000 in large_list:  # O(n) - 需要遍历列表
    pass

# 字典操作的时间复杂度（平均情况）
large_dict = {i: i**2 for i in range(10000)}

# O(1) - 常数时间操作
value = large_dict[5000]  # O(1) 平均情况
large_dict[10000] = 100000000  # O(1) 平均情况

# 集合操作的时间复杂度（平均情况）
large_set = set(range(10000))

# O(1) - 常数时间操作
large_set.add(10000)  # O(1) 平均情况
if 5000 in large_set:  # O(1) 平均情况
    pass

# deque操作的时间复杂度
from collections import deque
dq = deque(range(10000))

# O(1) - 两端操作
dq.appendleft(0)  # O(1) - 左端添加
dq.append(10000)  # O(1) - 右端添加
dq.popleft()      # O(1) - 左端删除
dq.pop()          # O(1) - 右端删除

# O(n) - 中间操作
# dq.insert(5000, 999)  # O(n) - 在中间插入
```

### 性能优化技巧

```python
import time
from collections import deque

# 技巧1: 选择合适的数据结构
# 场景：需要频繁在两端添加/删除元素

# 使用列表（低效）
def list_performance_test():
    lst = []
    start = time.time()
    for i in range(10000):
        lst.insert(0, i)  # 在开头插入，O(n)操作
    end = time.time()
    return end - start

# 使用deque（高效）
def deque_performance_test():
    dq = deque()
    start = time.time()
    for i in range(10000):
        dq.appendleft(i)  # 在左端添加，O(1)操作
    end = time.time()
    return end - start

# 技巧2: 利用推导式提高效率
# 传统方法
def traditional_method():
    result = []
    for i in range(1000):
        if i % 2 == 0:
            result.append(i * 2)
    return result

# 推导式方法（更高效且简洁）
def comprehension_method():
    return [i * 2 for i in range(1000) if i % 2 == 0]

# 技巧3: 使用适当的内置函数
from collections import Counter

# 统计元素频度的传统方法
def traditional_count(items):
    counts = {}
    for item in items:
        if item in counts:
            counts[item] += 1
        else:
            counts[item] = 1
    return counts

# 使用Counter（更高效）
def counter_method(items):
    return Counter(items)

# 技巧4: 避免重复计算
data = list(range(10000))

# 低效方法：在循环中重复计算
def inefficient_calculation():
    result = []
    for i in range(1000):
        # len(data)在每次迭代中都被调用
        if i < len(data) / 2:
            result.append(i)
    return result

# 高效方法：提前计算
def efficient_calculation():
    result = []
    half_length = len(data) / 2  # 提前计算
    for i in range(1000):
        if i < half_length:
            result.append(i)
    return result
```

## 总结

通过本文的学习，你应该掌握了以下关键技能：

1. **数据筛选技巧**：使用列表推导式、filter等方法高效筛选数据
2. **提高代码可读性**：使用namedtuple为元组元素命名
3. **频度统计**：利用Counter进行高效的元素频度统计
4. **字典排序**：掌握多种字典排序方法
5. **集合操作**：使用集合运算找出多个字典的公共键
6. **保持顺序**：使用OrderedDict或了解Python 3.7+字典的有序特性
7. **历史记录**：使用deque实现固定长度的历史记录功能
8. **性能优化**：理解时间复杂度，选择合适的数据结构

**关键要点**：
- 列表推导式通常比传统循环更高效且更易读
- namedtuple是在保持tuple内存效率的同时提高代码可读性的好方法
- Counter是进行频度统计的最佳工具
- deque在需要频繁两端操作的场景下比list更高效
- 了解不同操作的时间复杂度有助于选择合适的数据结构
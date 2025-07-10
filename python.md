## 函数

### 函数也是对象  
python 的函数也可以看作是一种对象，如
```py
def generatef(n) :
    def f(x) :
        return x * n
    return f

f = generatef(5)
print(f(10))  # 输出 50
```
类似于一种运行时编译

#### 修饰
本质仍是将函数作为对象操作
```py
def decorator(func) :
    def wrapper(*args, **kwargs):
        print("Before the function call")
        result = func(*args, **kwargs)
        print("After the function call")
        return result
    return wrapper

@decorator
def f(x,y) :
    print(x + y)

f(2,3)
```
等价于
```py
f = decorator(f)
```

如果修饰两次 
```py
@dec1
@dec2
def f
```
则等价于
```py
f = dec1(dec2(f))
```

## 虚拟环境
为了管理不同的版本 python 建议使用虚拟环境
```
python3 -m venv venv // 创建
source ./venv/bin/activate // 激活
deactivate // 退出
```
删除直接删除文件夹即可
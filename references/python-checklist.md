# Python 代码审查清单

## 正确性
- [ ] `is` 用于比较 None/True/False，而非 `==`
- [ ] 可变对象（list/dict/set）不作为函数默认参数
- [ ] `except:` 裸异常捕获是否合理（至少应写 `except Exception:`）
- [ ] `finally` 中是否有 `return`（会吞掉异常）
- [ ] 迭代时是否修改了正在遍历的容器
- [ ] 切片操作索引是否正确（`[start:stop:step]`）
- [ ] 浮点数比较使用 `math.isclose()` 而非 `==`
- [ ] 深拷贝 vs 浅拷贝（`copy.deepcopy` vs `copy.copy`）

## 安全
- [ ] SQL/NoSQL 查询使用参数化（`cursor.execute(sql, params)`）
- [ ] `eval()` / `exec()` / `__import__()` 是否有输入注入风险
- [ ] `pickle.load()` 是否从不可信来源加载
- [ ] `subprocess` 是否使用 `shell=True`（有命令注入风险）
- [ ] `os.system()` / `os.popen()` 使用
- [ ] 文件路径拼接使用 `pathlib` 而非字符串拼接（路径穿越风险）
- [ ] `yaml.load()` 使用 `SafeLoader`（`yaml.safe_load()`）
- [ ] 密码/密钥/Token 硬编码
- [ ] `assert` 用于安全检查（生产环境 `-O` 模式下 assert 被跳过）

## 性能
- [ ] 列表推导式替代 `for` 循环构建列表
- [ ] `in` 操作使用 `set` 而非 `list`（O(1) vs O(n)）
- [ ] 大文件使用迭代读取而非 `read()` 全量加载
- [ ] `f-strings` 优于 `+` 拼接字符串
- [ ] 生成器表达式替代列表推导式（内存友好）
- [ ] `@lru_cache` / `@cache` 缓存重复计算
- [ ] `join()` 优于 `+=` 拼接大量字符串
- [ ] `__slots__` 用于大量实例的类

## 可读性
- [ ] 函数不超过 50 行（超过需拆分）
- [ ] 类不超过 500 行（超过需拆分）
- [ ] 变量命名：`snake_case`，名称体现用途
- [ ] 常量使用大写 `CONSTANT_NAME`
- [ ] 类型注解（Type Hints）是否使用
- [ ] 魔法数字用命名常量替换
- [ ] `if __name__ == "__main__":` 保护入口代码
- [ ] 嵌套不超过 3 层（`if` 套 `if` 套 `if`）

## 可维护性
- [ ] DRY：重复代码是否抽取为函数/类
- [ ] 单函数单一职责
- [ ] 依赖注入代替硬编码依赖
- [ ] 配置使用环境变量或配置文件
- [ ] 日志使用 `logging` 而非 `print()`
- [ ] 类型注解是否完整（`mypy` 严格模式）
- [ ] 文档字符串（docstring）是否合理
- [ ] `__init__.py` 的 `__all__` 是否明确定义导出

## 常见反模式

```python
# ❌ 反模式：可变默认参数
def add(item, items=[]):
    items.append(item)
    return items

# ✅ 正确写法
def add(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

```python
# ❌ 反模式：except 裸捕获
try:
    result = risky_operation()
except:
    pass  # 吞掉了 KeyboardInterrupt、SystemExit 等

# ✅ 正确写法
try:
    result = risky_operation()
except Exception as e:
    log.error(f"操作失败: {e}")
    raise
```

```python
# ❌ 反模式：flatten 嵌套过深
if a:
    if b:
        if c:
            do_something()

# ✅ 正确写法：提前返回
if not a:
    return
if not b:
    return
if not c:
    return
do_something()
```

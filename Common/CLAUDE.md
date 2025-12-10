# Common 模块

> **导航**：[首页](../CLAUDE.md) > Common 模块

## 概述

Common 模块提供了整个框架的通用基础设施，包括枚举定义、异常处理、时间管理、缓存机制和工具函数等。是所有其他模块的基础依赖。

## 核心组件

### 1. CEnum.py - 枚举定义

**功能**：定义系统中使用的所有枚举类型

**核心枚举**：

```python
# 数据源类型
class DATA_SRC(Enum):
    BAO_STOCK = "baostock"      # BaoStock
    CCXT = "ccxt"               # 数字货币交易所
    CSV = "csv"                 # CSV文件
    TDX = "tdx"                 # 通达信
    CUSTOM = "custom"           # 自定义

# K线类型
class KL_TYPE(Enum):
    K_MIN = "1min"              # 1分钟
    K_5M = "5min"               # 5分钟
    K_15M = "15min"             # 15分钟
    K_30M = "30min"             # 30分钟
    K_60M = "60min"             # 60分钟
    K_DAY = "day"               # 日线
    K_WEEK = "week"             # 周线
    K_MONTH = "month"           # 月线

# 复权类型
class AUTYPE(Enum):
    QFQ = "qfq"                 # 前复权
    HFQ = "hfq"                 # 后复权
    NONE = "none"               # 不复权

# 分形标记
class FRACTAL_TYPE(Enum):
    TOP = "top"                 # 顶分形
    BOTTOM = "bottom"           # 底分形
    MIDDLE = "middle"           # 中分形

# 笔方向
class BI_DIR(Enum):
    UP = "up"                   # 向上笔
    DOWN = "down"               # 向下笔

# 线段方向
class SEG_DIR(Enum):
    UP = "up"                   # 向上线段
    DOWN = "down"               # 向下线段

# 买卖点类型
class BSP_TYPE(Enum):
    BUY1 = "buy1"               # 一买
    SELL1 = "sell1"             # 一卖
    BUY2 = "buy2"               # 二买
    SELL2 = "sell2"             # 二卖
    BUY3 = "buy3"               # 三买
    SELL3 = "sell3"             # 三卖

# 中枢类型
class ZS_TYPE(Enum):
    UP = "up"                   # 上涨中枢
    DOWN = "down"               # 下跌中枢
    CONFLUX = "conflux"         # 扩展中枢
```

### 2. ChanException.py - 异常处理

**功能**：定义系统专用的异常类型

**核心类**：`CChanException`

```python
class CChanException(Exception):
    def __init__(self, err_code: ErrCode, msg: str = "", detail: str = ""):
        self.err_code = err_code
        self.msg = msg
        self.detail = detail
        super().__init__(f"[{err_code.name}] {msg}")

class ErrCode(Enum):
    DATA_ERROR = (1001, "数据错误")
    PARAM_ERROR = (1002, "参数错误")
    CALC_ERROR = (1003, "计算错误")
    TIME_ERROR = (1004, "时间错误")
    SYMBOL_ERROR = (1005, "代码错误")
    NETWORK_ERROR = (1006, "网络错误")
    CONFIG_ERROR = (1007, "配置错误")
```

**使用示例**：
```python
from Common.ChanException import CChanException, ErrCode

def validate_kline_data(data):
    if not data:
        raise CChanException(ErrCode.DATA_ERROR, "K线数据为空")
```

### 3. CTime.py - 时间管理

**功能**：提供统一的时间处理类

**核心类**：`CTime`

```python
class CTime:
    def __init__(self, time_str=None, timestamp=None):
        # 支持多种时间格式初始化
        pass

    # 核心属性
    year: int
    month: int
    day: int
    hour: int
    minute: int
    second: int

    # 关键方法
    def to_str(self):                    # 转字符串
    def to_timestamp(self):              # 转时间戳
    def add_days(self, days):            # 日期加减
    def is_trade_day(self):              # 判断交易日
    def format(self, fmt):               # 格式化输出
```

**支持的时间格式**：
- "2023-01-01"
- "2023-01-01 09:30:00"
- "2023/01/01"
- 时间戳

### 4. func_util.py - 工具函数

**功能**：提供通用的工具函数

**核心函数**：

```python
def check_kltype_order(lv_list):
    """检查 K线类型顺序是否正确"""

def kltype_lte_day(kl_type):
    """判断 K线类型是否小于等于日线"""

def round_price(price, precision=2):
    """价格四舍五入"""

def calculate_change(last_close, current_price):
    """计算涨跌幅"""

def is_valid_price(price):
    """判断价格是否有效"""

def merge_time_dict(dict1, dict2):
    """合并时间序列字典"""
```

### 5. cache.py - 缓存机制

**功能**：提供数据缓存功能

**核心类**：`CCache`

```python
class CCache:
    def __init__(self, max_size=1000):
        self.cache = {}
        self.max_size = max_size

    def set(self, key, value, ttl=3600):    # 设置缓存
    def get(self, key):                     # 获取缓存
    def delete(self, key):                  # 删除缓存
    def clear(self):                        # 清空缓存
    def size(self):                         # 缓存大小
```

## 使用示例

### 时间处理
```python
from Common.CTime import CTime

# 创建时间对象
t1 = CTime("2023-01-01 09:30:00")
t2 = CTime(timestamp=1672531200)

# 时间运算
t3 = t1.add_days(30)
print(t3.to_str())  # "2023-01-31 09:30:00"

# 判断交易日
if t1.is_trade_day():
    print("是交易日")
```

### 异常处理
```python
from Common.ChanException import CChanException, ErrCode

try:
    # 业务代码
    process_data(data)
except CChanException as e:
    print(f"错误码: {e.err_code}")
    print(f"错误信息: {e.msg}")
    if e.detail:
        print(f"详细信息: {e.detail}")
```

### 缓存使用
```python
from Common.cache import CCache

# 创建缓存
cache = CCache(max_size=100)

# 设置缓存
cache.set("stock_000001", kline_data, ttl=1800)

# 获取缓存
data = cache.get("stock_000001")
if data is not None:
    print("从缓存获取数据")
else:
    print("缓存未命中，重新获取")
```

## 最佳实践

1. **枚举使用**：使用枚举而非字符串常量，提高代码可读性和类型安全
2. **异常处理**：每个模块都应该抛出 `CChanException`，包含明确的错误码
3. **时间处理**：统一使用 `CTime` 类处理时间，避免直接使用 datetime
4. **缓存策略**：对频繁访问的数据使用缓存，注意设置合理的 TTL
5. **工具函数**：通用逻辑提取到工具函数，避免代码重复

## 扩展指南

### 添加新的枚举
```python
# 在 CEnum.py 中添加
class NEW_ENUM(Enum):
    VALUE1 = "value1"
    VALUE2 = "value2"
```

### 添加新的异常类型
```python
# 在 ErrCode 中添加
NEW_ERROR = (2001, "新的错误类型")
```

### 添加新的工具函数
```python
# 在 func_util.py 中添加
def new_util_function(param):
    """新工具函数"""
    pass
```

## 注意事项

1. 所有时间都应转换为 `CTime` 对象处理
2. 异常信息应使用中文，便于理解
3. 枚举值使用小写，符合 Python 规范
4. 缓存键名要有明确的前缀，避免冲突
5. 工具函数应该保持 pure function 特性
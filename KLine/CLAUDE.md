# KLine 模块

> **导航**：[首页](../CLAUDE.md) > KLine 模块

## 概述

KLine 模块是整个缠论框架的数据基础，负责 K线数据的存储、管理和基础计算。提供了高效的 K线数据结构和相关操作方法。

## 核心组件

### 1. KLine_Unit.py - K线单元

**功能**：表示单根 K线的数据结构

**核心类**：`CKLine_Unit`

**属性**：
```python
class CKLine_Unit:
    time: CTime              # K线时间
    open: float              # 开盘价
    high: float              # 最高价
    low: float               # 最低价
    close: float             # 收盘价
    volume: float            # 成交量
    turnover: float          # 成交额
    kl_type: KL_TYPE         # K线类型
    code: str                # 代码

    # 缠论相关属性
    valid: bool              # 是否有效
    processed: bool          # 是否已处理
    combine: Combine_Item    # 合并信息
    fn: int                  # 分形标记
```

**关键方法**：
- `is_up()`: 判断是否为阳线
- `is_down()`: 判断是否为阴线
- `get_mid_price()`: 获取中位数价格
- `get_real_range()`: 获取真实波动范围

### 2. KLine_List.py - K线序列

**功能**：管理 K线序列，提供批量操作和迭代功能

**核心类**：`CKLine_List`

**主要功能**：
- K线数据的增删改查
- 时间序列切片操作
- 数据验证和清理
- 迭代器支持

**关键方法**：
```python
class CKLine_List:
    def __init__(self, kline_list=None):
        self.list = []  # CKLine_Unit 列表

    def add(self, kline_unit):  # 添加 K线
    def get_by_time(self, time):  # 根据时间获取
    def get_range(self, begin_idx, end_idx):  # 获取范围
    def is_valid(self):  # 验证数据有效性
    def __iter__(self):  # 迭代器支持
    def __getitem__(self, idx):  # 索引访问
```

### 3. KLine.py - K线处理

**功能**：提供 K线的基础处理和计算功能

**核心类**：`CKLine`

**主要功能**：
- K线数据初始化
- 基础技术指标计算
- 数据预处理
- 格式转换

**关键方法**：
```python
class CKLine:
    def __init__(self, kl_type, code):
        self.kl_type = kl_type
        self.code = code
        self.kline_list = CKLine_List()

    def load_from_api(self, api, begin_time, end_time):  # 从API加载
    def calculate_change(self):  # 计算涨跌幅
    def calculate_turnover_rate(self):  # 计算换手率
    def validate_data(self):  # 数据验证
```

### 4. TradeInfo.py - 交易信息

**功能**：存储交易相关的额外信息

**数据结构**：
```python
class TradeInfo:
    date: CTime             # 交易日期
    amount: float           # 成交额
    pe_ratio: float         # 市盈率
    pb_ratio: float         # 市净率
    total_market_value: float  # 总市值
   流通市场_value: float     # 流通市值
```

## K线类型定义

```python
class KL_TYPE(Enum):
    K_MIN = "1min"          # 1分钟
    K_5M = "5min"           # 5分钟
    K_15M = "15min"         # 15分钟
    K_30M = "30min"         # 30分钟
    K_60M = "60min"         # 60分钟
    K_DAY = "day"           # 日线
    K_WEEK = "week"         # 周线
    K_MONTH = "month"       # 月线
    K_QUARTER = "quarter"   # 季线
    K_YEAR = "year"         # 年线
```

## 数据流转过程

```mermaid
graph LR
    A[原始数据] --> B[数据清洗]
    B --> C[创建 CKLine_Unit]
    C --> D[构建 CKLine_List]
    D --> E[数据验证]
    E --> F[传递给合并器]
```

## 使用示例

### 创建 K线数据
```python
from KLine.KLine_Unit import CKLine_Unit
from KLine.KLine_List import CKLine_List
from Common.CTime import CTime
from Common.CEnum import KL_TYPE

# 创建单个 K线
kline = CKLine_Unit(
    time=CTime("2023-01-01 09:30:00"),
    open=10.0,
    high=10.5,
    low=9.8,
    close=10.2,
    volume=1000000,
    kl_type=KL_TYPE.K_DAY,
    code="000001"
)

# 创建 K线列表
kline_list = CKLine_List()
kline_list.add(kline)

# 遍历 K线
for kline in kline_list:
    print(f"时间: {kline.time}, 收盘: {kline.close}")
```

### 从数据源加载
```python
from KLine.KLine import CKLine
from DataAPI.BaoStockAPI import CBaoStockAPI

# 创建 K线对象
kline_obj = CKLine(KL_TYPE.K_DAY, "000001")

# 从 API 加载数据
api = CBaoStockAPI()
kline_obj.load_from_api(api, "2023-01-01", "2023-12-31")

# 获取 K线列表
kline_data = kline_obj.kline_list
```

## 性能优化

1. **批量处理**：使用批量方法减少循环次数
2. **索引缓存**：对频繁访问的数据建立索引
3. **内存管理**：大数据集使用生成器避免内存溢出
4. **数据压缩**：历史数据可考虑压缩存储

## 扩展功能

### 自定义指标
```python
# 在 CKLine_Unit 中添加
def calculate_custom_indicator(self):
    # 计算自定义指标
    return self.custom_value
```

### 数据增强
```python
# 在 CKLine_List 中添加
def add_derived_data(self):
    # 添加衍生数据
    pass
```

## 注意事项

1. K线数据的时间必须是连续的
2. 注意处理停牌、节假日等特殊情况
3. 不同数据源的数据精度可能不同
4. 大数据量操作时注意内存使用
5. 确保 K线数据的完整性，避免跳空
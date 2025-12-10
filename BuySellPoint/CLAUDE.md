# BuySellPoint 模块

> **导航**：[首页](../CLAUDE.md) > BuySellPoint 模块

## 概述

BuySellPoint 模块实现了缠论买卖点的核心算法，包括一买、二买、三买和一卖、二卖、三卖等各类买卖点的识别和计算。支持区间套策略和自定义买卖点开发。

## 核心组件

### 1. BS_Point.py - 买卖点基础类

**功能**：定义买卖点的基础结构和接口

**核心类**：`CBS_Point`

**属性**：
```python
class CBS_Point:
    point_type: BSP_TYPE      # 买卖点类型
    time: CTime              # 时间
    price: float             # 价格
    level: KL_TYPE           # 级别
    confident: float         # 置信度
    reason: str              # 原因说明
    related_zs: List[CTZS]   # 相关中枢
    trigger_kline: CKLine_Unit  # 触发K线
```

**买卖点类型**：
```python
class BSP_TYPE(Enum):
    # 买点
    BUY1 = "buy1"            # 一买 - 背驰买点
    BUY2 = "buy2"            # 二买 - 中枢买点
    BUY3 = "buy3"            # 三买 - 突破买点

    # 卖点
    SELL1 = "sell1"          # 一卖 - 背驰卖点
    SELL2 = "sell2"          # 二卖 - 中枢卖点
    SELL3 = "sell3"          # 三卖 - 突破卖点
```

### 2. BSPointList.py - 买卖点序列

**功能**：管理买卖点序列，提供查询和分析功能

**核心类**：`CBSPointList`

**主要功能**：
- 买卖点的添加和排序
- 按类型和时间过滤
- 买卖点匹配和配对
- 统计分析

**关键方法**：
```python
class CBSPointList:
    def __init__(self):
        self.buy_list = []      # 买点列表
        self.sell_list = []     # 卖点列表

    def add_bsp(self, bsp):                 # 添加买卖点
    def get_by_type(self, bsp_type):        # 按类型获取
    def get_by_time_range(self, begin, end): # 按时间范围获取
    def match_buy_sell(self):                # 配对买卖点
    def calculate_success_rate(self):        # 计算成功率
    def get_latest(self, bsp_type=None):     # 获取最新买卖点
```

### 3. BSPointConfig.py - 买卖点配置

**功能**：配置买卖点计算参数

**核心配置项**：
```python
class CBSPointConfig:
    # 通用配置
    min_confidence = 0.6          # 最小置信度
    enable_filter = True          # 启用过滤
    max_points_per_day = 5        # 每日最大买卖点数

    # 一买配置
    buy1_macd_period = 12         # MACD周期
    buy1_divergence_threshold = 0.1  # 背驰阈值

    # 二买配置
    buy2_zs_times = 3             # 中枢振荡次数
    buy2_support_strength = 0.5   # 支撑强度

    # 三买配置
    buy3_break_rate = 0.02        # 突破幅度
    buy3_volume_ratio = 1.5       # 成交量比率
```

## 买卖点算法详解

### 一买/一卖（背驰买卖点）

**识别条件**：
1. 形成趋势（至少两个中枢）
2. 价格创新高/新低
3. MACD出现背驰（绿柱缩短/红柱缩短）

**算法流程**：
```python
def detect_buy1():
    # 1. 识别趋势
    trend = identify_trend()

    # 2. 检查背驰
    if check_macd_divergence():
        # 3. 确认买点
        return create_buy_point()
```

### 二买/二卖（中枢买卖点）

**识别条件**：
1. 中枢扩展或盘整
2. 在中枢边界附近
3. 出现转折信号

**算法流程**：
```python
def detect_buy2():
    # 1. 识别中枢
    zs = identify_zs()

    # 2. 判断位置
    if is_near_zs_boundary():
        # 3. 确认买点
        return create_buy_point()
```

### 三买/三卖（突破买卖点）

**识别条件**：
1. 突破中枢
2. 突破幅度达到要求
3. 成交量放大确认

**算法流程**：
```python
def detect_buy3():
    # 1. 检查突破
    if check_zs_break():
        # 2. 验证强度
        if validate_break_strength():
            # 3. 确认买点
            return create_buy_point()
```

## 区间套策略

### 策略原理

区间套是缠论中重要的操作策略，通过多个级别的买卖点共振来提高成功率。

**核心思想**：
- 大级别定方向
- 小级别找时机
- 多级别共振确认

### 实现方式

```python
class IntervalStrategy:
    def __init__(self, levels):
        self.levels = levels  # [K_DAY, K_60M, K_15M]

    def detect_signal(self):
        signals = {}

        # 各级别信号
        for level in self.levels:
            signals[level] = self.detect_level_signal(level)

        # 信号共振判断
        if self.check_resonance(signals):
            return self.create_combined_signal()
```

## 使用示例

### 基础使用
```python
from BuySellPoint.BS_Point import CBS_Point
from BuySellPoint.BSPointList import CBSPointList
from Common.CEnum import BSP_TYPE

# 创建买卖点列表
bsp_list = CBSPointList()

# 添加买点
buy1 = CBS_Point(
    point_type=BSP_TYPE.BUY1,
    time=CTime("2023-01-01 09:30:00"),
    price=10.5,
    level=KL_TYPE.K_DAY,
    confident=0.8,
    reason="MACD背驰"
)
bsp_list.add_bsp(buy1)

# 获取所有买点
buy_points = bsp_list.get_by_type([BSP_TYPE.BUY1, BSP_TYPE.BUY2, BSP_TYPE.BUY3])
```

### 自定义买卖点策略
```python
from BuySellPoint.BS_Point import CBS_Point

class CustomBuyPoint(CBS_Point):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.custom_indicator = kwargs.get('custom_indicator')

    def validate(self):
        # 自定义验证逻辑
        return self.custom_indicator > threshold
```

## 配置示例

```python
from BuySellPoint.BSPointConfig import CBSPointConfig

# 创建配置
config = CBSPointConfig()

# 调整参数
config.buy1_macd_period = 26
config.buy3_break_rate = 0.03
config.min_confidence = 0.7

# 应用配置
bsp_detector = BuyPointDetector(config)
```

## 最佳实践

1. **参数优化**：根据不同品种和时间周期调整买卖点参数
2. **多级验证**：使用区间套策略提高信号质量
3. **风险控制**：设置合理的止损和止盈
4. **信号过滤**：过滤掉低置信度的买卖点
5. **回测验证**：对策略进行充分的历史回测

## 扩展开发

### 添加新的买卖点类型
```python
# 1. 扩展 BSP_TYPE 枚举
class BSP_TYPE(Enum):
    # ... 现有类型
    CUSTOM_BUY = "custom_buy"  # 自定义买点

# 2. 继承 CBS_Point 类
class CustomBuyPoint(CBS_Point):
    def detect(self):
        # 实现检测逻辑
        pass
```

### 添加新的过滤器
```python
class BSPFilter:
    def filter(self, bsp_list):
        # 过滤逻辑
        return filtered_list
```

## 注意事项

1. 买卖点具有滞后性，需要结合实时行情判断
2. 不是所有买卖点都应该操作，需要筛选高概率信号
3. 注意买卖点的级别，不同级别的操作策略不同
4. 市场环境变化时，及时调整买卖点参数
5. 任何买卖点都不是100%准确，需要做好风险管理
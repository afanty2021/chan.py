# Combiner 模块

> **导航**：[首页](../CLAUDE.md) > Combiner 模块

## 概述

Combiner（合并器）模块是缠论分析的第一步，负责对原始K线进行分形合并处理。通过识别和合并包含关系，生成标准化的K线序列，为后续的笔、线段分析奠定基础。

## 核心组件

### 1. KLine_Combiner.py - K线合并器

**功能**：实现K线的合并算法

**核心类**：`CKLine_Combiner`

**主要功能**：
- 识别K线包含关系
- 执行K线合并
- 处理特殊情况
- 维护合并历史

**关键方法**：
```python
class CKLine_Combiner:
    def __init__(self, config=None):
        self.config = config or CCombinerConfig()
        self.combined_klines = []
        self.combine_history = []

    def process_klines(self, kline_list):        # 处理K线序列
    def check_inclusion(self, k1, k2):           # 检查包含关系
    def combine_klines(self, outer, inner):      # 合并K线
    def handle_special_cases(self, klines):      # 处理特殊情况
    def get_combined_result(self):                # 获取合并结果
```

### 2. Combine_Item.py - 合并项

**功能**：记录K线合并的信息

**核心类**：`CCombine_Item`

**属性**：
```python
class CCombine_Item:
    combine_type: COMBINE_TYPE        # 合并类型
    original_klines: List[CKLine_Unit]  # 原始K线列表
    combined_kline: CKLine_Unit       # 合并后的K线
    combine_time: CTime              # 合并时间
    reason: str                      # 合并原因

class COMBINE_TYPE(Enum):
    UP_INCLUDE = "up_include"        # 向上包含
    DOWN_INCLUDE = "down_include"    # 向下包含
    EQUAL = "equal"                  # 相等处理
    SPECIAL = "special"              # 特殊处理
```

## 合并算法详解

### 包含关系判断

**向上包含**：
1. 后一根K线的高点 >= 前一根K线的高点
2. 后一根K线的低点 <= 前一根K线的低点
3. 两根K线都是阳线

**向下包含**：
1. 后一根K线的高点 >= 前一根K线的高点
2. 后一根K线的低点 <= 前一根K线的低点
3. 两根K线都是阴线

**算法实现**：
```python
def check_inclusion(k1, k2):
    """检查K1是否包含K2"""
    # 检查高低点关系
    if not (k1.high >= k2.high and k1.low <= k2.low):
        return None

    # 判断方向
    if k1.close > k1.open and k2.close > k2.open:
        # 都是阳线，向上包含
        return COMBINE_TYPE.UP_INCLUDE
    elif k1.close < k1.open and k2.close < k2.open:
        # 都是阴线，向下包含
        return COMBINE_TYPE.DOWN_INCLUDE
    else:
        # 特殊情况处理
        return COMBINE_TYPE.SPECIAL
```

### K线合并规则

**向上包含合并**：
- 高点取两根K线高点的较大值
- 低点取两根K线低点的较小值
- 收盘价取两根K线收盘价的较大值

**向下包含合并**：
- 高点取两根K线高点的较大值
- 低点取两根K线低点的较小值
- 收盘价取两根K线收盘价的较小值

**特殊情况处理**：
1. 一阴一阳的包含关系
2. 十字星处理
3. 跳空缺口处理
4. 极端价格处理

**合并实现**：
```python
def combine_klines(outer, inner, combine_type):
    """合并两根K线"""
    combined = CKLine_Unit()

    # 基础属性
    combined.time = inner.time  # 使用后一根K线的时间
    combined.open = outer.open  # 使用第一根K线的开盘价

    # 合并高低点
    combined.high = max(outer.high, inner.high)
    combined.low = min(outer.low, inner.low)

    # 根据合并类型确定收盘价
    if combine_type == COMBINE_TYPE.UP_INCLUDE:
        combined.close = max(outer.close, inner.close)
    elif combine_type == COMBINE_TYPE.DOWN_INCLUDE:
        combined.close = min(outer.close, inner.close)
    else:
        # 特殊处理
        combined.close = (outer.close + inner.close) / 2

    # 合并成交量
    combined.volume = outer.volume + inner.volume

    # 合并成交额
    combined.turnover = outer.turnover + inner.turnover

    return combined
```

### 完整合并流程

```python
def process_klines(kline_list):
    """处理完整的K线序列"""
    if len(kline_list) < 2:
        return kline_list

    combined_list = []
    i = 0

    while i < len(kline_list) - 1:
        k1 = kline_list[i]
        k2 = kline_list[i + 1]

        # 检查包含关系
        combine_type = check_inclusion(k1, k2)

        if combine_type:
            # 需要合并
            combined = combine_klines(k1, k2, combine_type)

            # 记录合并信息
            combine_item = CCombine_Item(
                combine_type=combine_type,
                original_klines=[k1, k2],
                combined_kline=combined
            )
            combine_history.append(combine_item)

            # 继续检查是否与下一根K线合并
            if i + 2 < len(kline_list):
                k3 = kline_list[i + 2]
                # 用合并后的K线继续检查
                if check_inclusion(combined, k3):
                    # 递归处理
                    combined_list.extend(process_klines([combined, k3] + kline_list[i+3:]))
                    return combined_list

            combined_list.append(combined)
            i += 2
        else:
            # 不需要合并
            combined_list.append(k1)
            i += 1

    # 添加最后一根K线
    if i < len(kline_list):
        combined_list.append(kline_list[-1])

    return combined_list
```

## 使用示例

### 基础使用
```python
from Combiner.KLine_Combiner import CKLine_Combiner
from Combiner.Combine_Item import CCombine_Item

# 创建合并器
combiner = CKLine_Combiner()

# 获取原始K线数据
original_klines = get_original_klines()

# 执行合并
combined_klines = combiner.process_klines(original_klines)

# 获取合并结果
result = combiner.get_combined_result()

print(f"原始K线数: {len(original_klines)}")
print(f"合并后K线数: {len(combined_klines)}")
print(f"合并次数: {len(combiner.combine_history)}")
```

### 分析合并效果
```python
def analyze_combine_effects(combiner):
    """分析合并效果"""
    original_count = 0
    combined_count = 0
    combine_types = {}

    for item in combiner.combine_history:
        original_count += len(item.original_klines)
        combined_count += 1

        combine_type = item.combine_type
        combine_types[combine_type] = combine_types.get(combine_type, 0) + 1

    print("合并统计:")
    print(f"  原始K线数: {original_count}")
    print(f"  合并后K线数: {combined_count}")
    print(f"  压缩率: {(1 - combined_count/original_count)*100:.2f}%")
    print("  合并类型分布:")
    for ctype, count in combine_types.items():
        print(f"    {ctype}: {count}")
```

### 自定义合并规则
```python
class CustomCombiner(CKLine_Combiner):
    def __init__(self, config=None):
        super().__init__(config)
        self.custom_rules = []

    def add_custom_rule(self, rule_func):
        """添加自定义合并规则"""
        self.custom_rules.append(rule_func)

    def check_custom_rules(self, k1, k2):
        """检查自定义规则"""
        for rule in self.custom_rules:
            result = rule(k1, k2)
            if result:
                return result
        return None

    def process_klines(self, kline_list):
        """使用自定义规则处理"""
        # 实现自定义处理逻辑
        pass

# 自定义规则示例
def volume_combine_rule(k1, k2):
    """基于成交量的合并规则"""
    if k2.volume > k1.volume * 3:  # 成交量异常放大
        return COMBINE_TYPE.SPECIAL
    return None

# 使用自定义合并器
custom_combiner = CustomCombiner()
custom_combiner.add_custom_rule(volume_combine_rule)
```

## 配置参数

### CCombinerConfig 配置项
```python
class CCombinerConfig:
    # 基础配置
    enable_combine = True          # 启用合并
    strict_mode = False           # 严格模式

    # 特殊处理
    handle_gap = True             # 处理跳空
    handle_equal = True           # 处理相等
    handle_doji = True            # 处理十字星

    # 阈值配置
    min_combine_ratio = 0.01      # 最小合并比例
    max_combine_count = 5         # 最大连续合并数

    # 时间过滤
    min_time_span = 1             # 最小时间间隔
    ignore_weekend = True         # 忽略周末
```

## 性能优化

### 批量处理
```python
def batch_process_klines(kline_groups, combiner):
    """批量处理多组K线"""
    results = []

    for group in kline_groups:
        combined = combiner.process_klines(group)
        results.append(combined)

    return results
```

### 增量处理
```python
class IncrementalCombiner(CKLine_Combiner):
    def __init__(self, config=None):
        super().__init__(config)
        self.last_kline = None
        self.pending_combine = None

    def add_new_kline(self, kline):
        """增量添加新K线"""
        if self.last_kline:
            # 检查是否与最后一根K线合并
            combine_type = self.check_inclusion(self.last_kline, kline)

            if combine_type:
                # 合并
                combined = self.combine_klines(self.last_kline, kline, combine_type)
                self.combined_klines[-1] = combined
                self.last_kline = combined
            else:
                # 不合并，添加新K线
                self.combined_klines.append(kline)
                self.last_kline = kline
        else:
            # 第一根K线
            self.combined_klines.append(kline)
            self.last_kline = kline
```

## 扩展功能

### 合并质量评估
```python
def evaluate_combine_quality(original, combined):
    """评估合并质量"""
    metrics = {
        'compression_ratio': len(combined) / len(original),
        'price_preservation': calculate_price_similarity(original, combined),
        'volume_preservation': calculate_volume_similarity(original, combined),
        'pattern_preservation': check_pattern_similarity(original, combined)
    }
    return metrics

def calculate_price_similarity(original, combined):
    """计算价格相似度"""
    # 对齐时间序列
    # 计算价格差异
    # 返回相似度分数
    pass
```

### 可视化合并过程
```python
def visualize_combine_process(original_klines, combined_klines):
    """可视化合并过程"""
    import matplotlib.pyplot as plt

    fig, axes = plt.subplots(2, 1, figsize=(20, 10))

    # 原始K线
    plot_candlestick(axes[0], original_klines)
    axes[0].set_title("原始K线")

    # 合并后K线
    plot_candlestick(axes[1], combined_klines)
    axes[1].set_title("合并后K线")

    # 标记合并位置
    for item in combine_history:
        for kline in item.original_klines:
            axes[0].axvline(x=kline.time, color='red', alpha=0.3)

    plt.tight_layout()
    return fig
```

## 最佳实践

1. **参数调整**：根据市场特性调整合并规则
2. **数据验证**：合并后验证数据合理性
3. **性能监控**：监控合并对性能的影响
4. **特殊处理**：对异常行情制定特殊规则
5. **结果分析**：定期分析合并效果

## 注意事项

1. 合并会减少数据量，可能丢失细节
2. 不同合并规则会产生不同结果
3. 实时数据处理需要考虑延迟
4. 历史数据回测时保持一致性
5. 合并后的数据需要重新计算指标
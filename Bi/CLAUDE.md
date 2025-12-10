# Bi 模块

> **导航**：[首页](../CLAUDE.md) > Bi 模块

## 概述

Bi（笔）模块是缠论分析的基础组件，负责计算和管理笔。笔是缠论中描述价格波动的基本单元，由相邻的顶分形和底分形连接而成。本模块提供了完整的笔识别、分类和管理功能。

## 核心组件

### 1. Bi.py - 笔基础类

**功能**：定义笔的基础结构和行为

**核心类**：`CBi`

**属性**：
```python
class CBi:
    idx: int                 # 笔的索引
    direction: BI_DIR       # 笔的方向（上/下）
    start: CFractal         # 起始分形
    end: CFractal           # 结束分形
    level: KL_TYPE          # 级别
    broken: bool            # 是否被破坏
    break_time: CTime       # 破坏时间
    price: float            # 笔的价格（高点或低点）
    time: CTime             # 笔的时间
    kline_count: int        # 包含的K线数量
```

**笔方向**：
```python
class BI_DIR(Enum):
    UP = "up"               # 向上笔（从底分形到顶分形）
    DOWN = "down"           # 向下笔（从顶分形到底分形）
```

**关键方法**：
```python
class CBi:
    def get_length(self):               # 获取笔的长度（价格）
    def get_duration(self):             # 获取笔的时间跨度
    def get_slope(self):                # 获取笔的斜率
    def get_volume(self):               # 获取笔的成交量
    def is_broken_by(self, price):      # 判断是否被破坏
    def get_mid_price(self):            # 获取中点价格
```

### 2. BiList.py - 笔列表

**功能**：管理笔序列，提供查询和分析功能

**核心类**：`CBiList`

**主要功能**：
- 笔的添加和排序
- 笔的验证和修正
- 笔的统计和分析
- 笔的模式识别

**关键方法**：
```python
class CBiList:
    def __init__(self):
        self.list = []              # 笔列表
        self.level = KL_TYPE.K_DAY  # 级别

    def add_bi(self, bi):            # 添加笔
    def validate_bis(self):         # 验证笔序列
    def find_pattern(self):         # 寻找模式
    def get_statistics(self):        # 获取统计信息
    def get_by_direction(self, dir): # 按方向获取
    def merge_adjacent(self):       # 合并相邻笔
```

### 3. BiConfig.py - 笔配置

**功能**：配置笔的计算参数

**核心配置项**：
```python
class CBiConfig:
    # 分形配置
    fractal_left = 1              # 左侧K线数
    fractal_right = 1             # 右侧K线数
    min_fractal_gap = 0.01        # 最小分形间距

    # 笔配置
    min_bi_height = 0.02          # 最小笔高度
    min_bi_duration = 3           # 最小笔持续时间
    enable_merge = True           # 允许合并小笔
    merge_ratio = 0.382           # 合并比例阈值

    # 验证配置
    enable_strict_mode = False    # 严格模式
    check_overlap = True          # 检查重叠
    filter_noise = True           # 过滤噪音
```

## 笔算法详解

### 分形识别

**顶分形**：
1. 中间K线高点是局部最高点
2. 左右各有至少一根K线
3. 满足最小间距要求

**底分形**：
1. 中间K线低点是局部最低点
2. 左右各有至少一根K线
3. 满足最小间距要求

**算法实现**：
```python
def identify_fractal(klines, left=1, right=1):
    fractals = []

    for i in range(left, len(klines) - right):
        center = klines[i]

        # 检查顶分形
        is_top = True
        for j in range(i - left, i + right + 1):
            if j != i and klines[j].high >= center.high:
                is_top = False
                break
        if is_top:
            fractals.append(CFractal(i, FRACTAL_TYPE.TOP, center.high, center.time))

        # 检查底分形
        is_bottom = True
        for j in range(i - left, i + right + 1):
            if j != i and klines[j].low <= center.low:
                is_bottom = False
                break
        if is_bottom:
            fractals.append(CFractal(i, FRACTAL_TYPE.BOTTOM, center.low, center.time))

    return fractals
```

### 笔连接

**连接规则**：
1. 顶分形和底分形交替连接
2. 笔不能完全包含在前一笔内
3. 满足最小高度和时间要求

**算法流程**：
```python
def connect_fractals(fractals, config):
    bi_list = []

    # 1. 按时间排序
    fractals.sort(key=lambda x: x.time)

    # 2. 寻找有效笔
    for i in range(len(fractals) - 1):
        f1 = fractals[i]
        f2 = fractals[i + 1]

        # 检查分形类型
        if f1.type == f2.type:
            continue

        # 创建笔
        bi = create_bi(f1, f2)

        # 验证笔
        if validate_bi(bi, config):
            bi_list.append(bi)

    # 3. 优化笔序列
    bi_list = optimize_bi_list(bi_list, config)

    return bi_list
```

### 笔的破坏

**破坏条件**：
- 向上笔：后续价格跌破笔的起点
- 向下笔：后续价格涨破笔的起点
- 破坏需要得到确认

**判断逻辑**：
```python
def check_bi_break(bi, current_price, config):
    if bi.direction == BI_DIR.UP:
        # 向上笔被向下跌破
        if current_price < bi.start.price * (1 - config.break_threshold):
            return True, BI_BREAK_TYPE.DOWN_BREAK
    else:
        # 向下笔被向上突破
        if current_price > bi.start.price * (1 + config.break_threshold):
            return True, BI_BREAK_TYPE.UP_BREAK
    return False, None
```

## 使用示例

### 基础使用
```python
from Bi.Bi import CBi
from Bi.BiList import CBiList
from Bi.BiConfig import CBiConfig
from Common.CEnum import BI_DIR

# 创建配置
config = CBiConfig()

# 创建笔列表
bi_list = CBiList()

# 创建笔
bi = CBi(
    idx=1,
    direction=BI_DIR.UP,
    start=bottom_fractal,
    end=top_fractal,
    level=KL_TYPE.K_DAY
)

# 添加到列表
bi_list.add_bi(bi)

# 获取统计信息
stats = bi_list.get_statistics()
print(f"总笔数: {stats['total_count']}")
print(f"上涨笔: {stats['up_count']}")
print(f"下跌笔: {stats['down_count']}")
```

### 笔分析
```python
# 分析笔的特征
for bi in bi_list.list:
    print(f"笔 {bi.idx}:")
    print(f"  方向: {bi.direction}")
    print(f"  长度: {bi.get_length():.2f}")
    print(f"  时间: {bi.get_duration()} 天")
    print(f"  斜率: {bi.get_slope():.4f}")
    print(f"  成交量: {bi.get_volume():,}")
```

### 笔的模式识别
```python
def identify_patterns(bi_list):
    patterns = []

    for i in range(len(bi_list.list) - 2):
        bi1 = bi_list.list[i]
        bi2 = bi_list.list[i + 1]
        bi3 = bi_list.list[i + 2]

        # 识别三笔形态
        if bi1.direction == bi3.direction and bi1.direction != bi2.direction:
            pattern = {
                'type': 'three_bi',
                'start_idx': bi1.idx,
                'end_idx': bi3.idx,
                'direction': bi1.direction
            }
            patterns.append(pattern)

    return patterns

patterns = identify_patterns(bi_list)
print(f"识别到 {len(patterns)} 个三笔形态")
```

## 笔的特征分析

### 基础特征
- **长度**：价格差值
- **时间**：持续时间
- **斜率**：长度/时间
- **成交量**：累计成交量
- **振幅**：(最大值-最小值)/平均值

### 高级特征
```python
class BiFeatureExtractor:
    def extract_features(self, bi):
        features = {
            'length': bi.get_length(),
            'duration': bi.get_duration(),
            'slope': bi.get_slope(),
            'volume': bi.get_volume(),
            'amplitude': self.calculate_amplitude(bi),
            'momentum': self.calculate_momentum(bi),
            'consistency': self.calculate_consistency(bi)
        }
        return features

    def calculate_momentum(self, bi):
        """计算动量"""
        if bi.direction == BI_DIR.UP:
            return bi.end.price / bi.start.price - 1
        else:
            return bi.start.price / bi.end.price - 1

    def calculate_consistency(self, bi):
        """计算一致性（收盘价与极值的关系）"""
        # 实现一致性计算逻辑
        pass
```

## 级别关系

### 笔的级别
- 不同时间周期产生不同级别的笔
- 高级别笔由低级别笔组合而成
- 支持多级别联立分析

### 级别转换
```python
def upgrade_bi_level(bi_list, target_level):
    """将低级别笔组合成高级别笔"""
    upgraded_bis = []

    # 根据级别比例组合
    combine_ratio = get_level_ratio(target_level)

    for i in range(0, len(bi_list.list), combine_ratio):
        group = bi_list.list[i:i + combine_ratio]
        if len(group) >= combine_ratio:
            new_bi = combine_bi_group(group, target_level)
            upgraded_bis.append(new_bi)

    return upgraded_bis
```

## 配置优化

### 不同市场的配置

**A股配置**：
```python
config = CBiConfig(
    fractal_left=1,
    fractal_right=1,
    min_bi_height=0.02,
    min_bi_duration=3
)
```

**数字货币配置**：
```python
config = CBiConfig(
    fractal_left=1,
    fractal_right=1,
    min_bi_height=0.005,
    min_bi_duration=1
)
```

**期货配置**：
```python
config = CBiConfig(
    fractal_left=1,
    fractal_right=1,
    min_bi_height=0.01,
    min_bi_duration=2
)
```

## 性能优化

### 增量更新
```python
class IncrementalBiCalculator:
    def __init__(self, config):
        self.config = config
        self.last_klines = []
        self.last_fractals = []
        self.last_bis = []

    def update(self, new_klines):
        """增量更新笔"""
        # 1. 合并新K线
        all_klines = self.last_klines + new_klines

        # 2. 只对新增部分计算分形
        new_fractals = self.calculate_new_fractals(new_klines)

        # 3. 更新笔
        updated_bis = self.update_bi_list(new_fractals)

        # 4. 保存状态
        self.last_klines = all_klines[-100:]  # 保留最近100根
        self.last_fractals = updated_bis

        return updated_bis
```

## 扩展开发

### 自定义笔算法
```python
class CustomBiCalculator:
    def __init__(self, custom_config):
        self.config = custom_config

    def calculate_bi(self, klines):
        """自定义笔计算算法"""
        # 实现自定义逻辑
        pass

    def custom_validation(self, bi):
        """自定义验证规则"""
        if self.check_special_pattern(bi):
            return True
        return False
```

### 特殊笔识别
```python
class SpecialBiDetector:
    def detect_gap_bi(self, bi_list):
        """检测跳空笔"""
        gap_bis = []
        for i in range(1, len(bi_list)):
            prev_bi = bi_list[i-1]
            curr_bi = bi_list[i]

            if self.is_gap(prev_bi, curr_bi):
                gap_bis.append(curr_bi)
        return gap_bis

    def is_gap(self, bi1, bi2):
        """判断是否跳空"""
        # 实现跳空判断逻辑
        pass
```

## 最佳实践

1. **参数调整**：根据品种特性调整笔的参数
2. **验证机制**：使用验证确保笔的合理性
3. **级别选择**：选择合适的分析级别
4. **噪音过滤**：过滤掉过小的笔
5. **多级验证**：使用多级别验证笔的有效性

## 注意事项

1. 笔的确认具有滞后性
2. 不同参数会产生不同的笔
3. 极端行情可能产生异常笔
4. 需要结合其他元素综合分析
5. 实时计算时注意性能问题
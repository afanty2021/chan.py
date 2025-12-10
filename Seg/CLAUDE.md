# Seg 模块

> **导航**：[首页](../CLAUDE.md) > Seg 模块

## 概述

Seg 模块实现缠论中的线段计算，提供了多种线段算法实现，包括标准缠论线段、通用线段算法、自定义线段等。线段是缠论中重要的组成部分，用于描述趋势的中等结构。

## 核心组件

### 1. Seg.py - 线段基础类

**功能**：定义线段的基础结构和行为

**核心类**：`CSeg`

**属性**：
```python
class CSeg:
    seg_type: SEG_TYPE        # 线段类型
    direction: SEG_DIR        # 线段方向（上/下）
    start: CKLine_Unit        # 起点
    end: CKLine_Unit          # 终点
    bi_list: List[CBi]        # 包含的笔列表
    level: KL_TYPE            # 级别
    broken: bool              # 是否被破坏
    break_time: CTime         # 破坏时间
```

**线段方向**：
```python
class SEG_DIR(Enum):
    UP = "up"                 # 向上线段
    DOWN = "down"             # 向下线段
```

### 2. SegListComm.py - 通用线段算法

**功能**：实现通用的线段计算算法

**核心类**：`CSegListComm`

**算法特点**：
- 灵活的线段定义规则
- 支持多种破坏条件
- 可配置的参数

**关键方法**：
```python
class CSegListComm:
    def __init__(self, config):
        self.config = config
        self.seg_list = []

    def add_bi(self, bi):                     # 添加笔
    def check_seg_break(self):                # 检查线段破坏
    def merge_small_seg(self):                # 合并小线段
    def validate_seg(self):                   # 验证线段
```

### 3. SegListChan.py - 缠论标准线段

**功能**：实现缠论的标准线段算法

**核心类**：`CSegListChan`

**缠论线段特征**：
1. 线段必须被反向线段破坏才算结束
2. 破坏需要满足特定条件
3. 支持线段的包含关系处理

**算法流程**：
```python
def process_chan_seg():
    # 1. 获取笔序列
    bi_list = get_bi_list()

    # 2. 初始化线段
    if len(bi_list) >= 3:
        init_seg(bi_list[:3])

    # 3. 逐笔处理
    for bi in bi_list[3:]:
        process_new_bi(bi)

    # 4. 处理线段包含关系
    handle_seg_inclusion()
```

### 4. SegListDef.py - 自定义线段

**功能**：提供可自定义的线段算法

**核心类**：`CSegListDef`

**自定义选项**：
- 线段长度阈值
- 破坏幅度阈值
- 时间过滤条件
- 特殊形态识别

### 5. SegListDYH.py - 大智慧线段算法

**功能**：实现类似大智慧的线段算法

**特点**：
- 更严格的线段定义
- 特殊的破坏判断
- 适合特定市场环境

### 6. Eigen.py - 线段特征

**功能**：提取线段的特征信息

**特征类型**：
- 长度特征
- 时间特征
- 角度特征
- 成交量特征
- 波动率特征

### 7. EigenFX.py - 线段特征分析

**功能**：对线段特征进行深度分析

**分析内容**：
- 特征统计
- 相似性比较
- 模式识别
- 预测模型

## 线段算法详解

### 标准缠论线段

**定义规则**：
1. 线段至少包含三笔
2. 线段起点和终点必须是反向笔的极值
3. 线段内部可以有波动

**破坏条件**：
1. 反向线段突破起点
2. 反向线段满足最小幅度要求
3. 破坏得到确认

### 通用线段算法

**特点**：
- 更灵活的参数设置
- 支持多种破坏模式
- 适应不同市场

```python
class GeneralSegConfig:
    min_bi_count = 3               # 最少笔数
    break_ratio = 0.618            # 破坏比例
    min_break_time = 3             # 最少破坏时间
    enable_merge = True            # 允许合并
```

## 使用示例

### 基础使用
```python
from Seg.SegListChan import CSegListChan
from Seg.SegConfig import CSegConfig
from Common.CEnum import KL_TYPE

# 创建配置
config = CSegConfig()

# 创建线段计算器
seg_calculator = CSegListChan(config)

# 添加笔数据
for bi in bi_list:
    seg_calculator.add_bi(bi)

# 获取线段列表
seg_list = seg_calculator.seg_list

# 遍历线段
for seg in seg_list:
    print(f"线段: {seg.direction}, 长度: {seg.get_length()}")
```

### 自定义线段算法
```python
from Seg.SegListDef import CSegListDef

# 自定义配置
config = {
    'min_bi_count': 5,
    'break_ratio': 0.5,
    'filter_small_seg': True
}

# 创建自定义线段计算器
seg_calculator = CSegListDef(config)
```

### 线段特征提取
```python
from Seg.Eigen import CSegEigen

# 创建特征提取器
eigen = CSegEigen()

# 提取线段特征
features = eigen.extract_features(seg)

print(f"长度: {features.length}")
print(f"时间跨度: {features.time_span}")
print(f"平均斜率: {features.avg_slope}")
```

## 配置参数

### SegConfig.py 配置项

```python
class CSegConfig:
    # 基础配置
    seg_algo = "chan"              # 算法类型
    min_bi_count = 3               # 最小笔数
    level = KL_TYPE.K_DAY          # 默认级别

    # 破坏条件
    break_mode = "normal"          # 破坏模式
    break_threshold = 0.01         # 破坏阈值
    confirm_candles = 3            # 确认K线数

    # 过滤条件
    min_seg_length = 0.02          # 最小线段长度
    min_seg_time = 5               # 最小时间跨度
    filter_noise = True            # 过滤噪音

    # 特殊处理
    handle_gap = True              # 处理跳空
    merge_similar = True           # 合并相似线段
```

## 算法对比

| 算法类型 | 特点 | 适用场景 |
|---------|------|---------|
| Chan | 标准缠论 | 传统缠论分析 |
| Comm | 通用灵活 | 多种市场环境 |
| Def | 高度可定制 | 特殊需求 |
| DYH | 大智慧风格 | 特定指标 |

## 性能优化

1. **增量计算**：只处理新增数据，避免全量计算
2. **缓存机制**：缓存中间结果，提高计算效率
3. **批量处理**：批量添加数据，减少处理次数
4. **参数调优**：根据数据特点调整参数

## 扩展开发

### 自定义线段算法
```python
from Seg.Seg import CSeg

class CustomSegAlgorithm(CSeg):
    def __init__(self, config):
        super().__init__()
        self.config = config

    def process_bi(self, bi):
        # 实现自定义算法
        pass

    def check_break_condition(self):
        # 自定义破坏条件
        pass
```

### 添加新的特征
```python
class CustomSegEigen:
    def extract_custom_feature(self, seg):
        # 提取自定义特征
        return feature_value
```

## 注意事项

1. 不同算法的线段结果可能不同
2. 线段确认具有滞后性
3. 需要根据市场特点选择合适的算法
4. 参数设置对结果影响较大
5. 建议结合其他指标综合判断
# ZS 模块

> **导航**：[首页](../CLAUDE.md) > ZS 模块

## 概述

ZS（中枢）模块是缠论框架的核心组件之一，负责计算和分析中枢。中枢是缠论中的重要概念，表示价格在某个区间内的盘整和振荡。本模块提供了完整的中枢识别、分析和扩展功能。

## 核心组件

### 1. ZS.py - 中枢基础类

**功能**：定义中枢的基础结构和行为

**核心类**：`CZS`

**属性**：
```python
class CZS:
    zs_type: ZS_TYPE          # 中枢类型
    level: KL_TYPE           # 级别
    high: float              # 中枢上沿
    low: float               # 中枢下沿
    center: float            # 中枢中心
    start_time: CTime        # 开始时间
    end_time: CTime          # 结束时间
    seg_list: List[CSeg]     # 包含的线段列表
    bi_list: List[CBi]       # 包含的笔列表
    turnover: float          # 成交量
    amplitude: float         # 振荡幅度

    # 扩展属性
    extension: bool          # 是否扩展
    merge_with: List[CZS]    # 合并的中枢列表
```

**中枢类型**：
```python
class ZS_TYPE(Enum):
    UP = "up"                # 上涨中枢（下上下）
    DOWN = "down"            # 下跌中枢（上下上）
    CONFLUX = "conflux"      # 扩展中枢
```

### 2. ZSList.py - 中枢序列

**功能**：管理中枢序列，提供查询和分析功能

**核心类**：`CZSList`

**主要功能**：
- 中枢的添加和排序
- 中枢级别的维护
- 中枢的合并和扩展
- 中枢统计分析

**关键方法**：
```python
class CZSList:
    def __init__(self):
        self.zs_list = []          # 中枢列表
        self.level = KL_TYPE.K_DAY

    def add_zs(self, zs):            # 添加中枢
    def merge_zs(self, zs1, zs2):   # 合并中枢
    def get_by_time(self, time):    # 按时间获取
    def get_active_zs(self):        # 获取活跃中枢
    def calculate_overlap(self):    # 计算重叠
    def get_zs_statistics(self):    # 获取统计信息
```

### 3. ZSConfig.py - 中枢配置

**功能**：配置中枢计算参数

**核心配置项**：
```python
class CZSConfig:
    # 基础配置
    min_seg_count = 3           # 最小线段数
    min_zs_span = 3             # 最小时间跨度
    overlap_threshold = 0.01    # 重叠阈值

    # 扩展配置
    enable_extension = True     # 允许扩展
    extension_ratio = 0.382     # 扩展比例
    max_extension = 3           # 最大扩展次数

    # 合并配置
    enable_merge = True         # 允许合并
    merge_distance = 0.02       # 合并距离阈值
    merge_time_gap = 5          # 合并时间间隔
```

## 中枢算法详解

### 标准中枢识别

**中枢定义**：
- 上涨中枢：由至少三线段构成，形态为下-上-下
- 下跌中枢：由至少三线段构成，形态为上-下-上
- 中枢区间：三线段价格区间的重叠部分

**算法流程**：
```python
def identify_zs():
    # 1. 获取线段序列
    seg_list = get_seg_list()

    # 2. 寻找中枢形态
    for i in range(len(seg_list) - 2):
        seg1 = seg_list[i]
        seg2 = seg_list[i + 1]
        seg3 = seg_list[i + 2]

        # 3. 判断是否形成中枢
        if check_zs_form(seg1, seg2, seg3):
            zs = create_zs([seg1, seg2, seg3])
            zs_list.add_zs(zs)
```

### 中枢扩展

**扩展条件**：
1. 新线段与中枢有重叠
2. 扩展次数未超限
3. 满足时间条件

**扩展流程**：
```python
def extend_zs(zs, new_seg):
    # 1. 检查重叠
    if check_overlap(zs, new_seg):
        # 2. 添加到中枢
        zs.add_seg(new_seg)
        zs.extension = True

        # 3. 重新计算中枢区间
        zs.recalculate_range()
```

### 中枢合并

**合并条件**：
1. 两个中枢有重叠
2. 时间间隔较小
3. 级别相同

**合并流程**：
```python
def merge_zs(zs1, zs2):
    # 1. 检查合并条件
    if can_merge(zs1, zs2):
        # 2. 创建合并后的中枢
        new_zs = CZS()
        new_zs.merge_from(zs1, zs2)

        # 3. 更新中枢列表
        zs_list.replace([zs1, zs2], new_zs)
```

## 使用示例

### 基础使用
```python
from ZS.ZS import CZS
from ZS.ZSList import CZSList
from ZS.ZSConfig import CZSConfig
from Common.CEnum import ZS_TYPE, KL_TYPE

# 创建配置
config = CZSConfig()

# 创建中枢列表
zs_list = CZSList()

# 创建中枢
zs = CZS(
    zs_type=ZS_TYPE.UP,
    level=KL_TYPE.K_DAY,
    high=12.5,
    low=10.8,
    start_time=CTime("2023-01-01"),
    end_time=CTime("2023-01-15")
)

# 添加到列表
zs_list.add_zs(zs)
```

### 中枢分析
```python
# 获取活跃中枢
active_zs = zs_list.get_active_zs()

# 中枢统计
stats = zs_list.get_zs_statistics()
print(f"总中枢数: {stats['total_count']}")
print(f"平均持续时间: {stats['avg_duration']}")

# 中枢强度分析
strength = zs.calculate_strength()
print(f"中枢强度: {strength}")
```

### 自定义中枢算法
```python
class CustomZS(CZS):
    def __init__(self, config):
        super().__init__()
        self.config = config

    def custom_validation(self):
        # 自定义验证逻辑
        if self.amplitude < self.config.min_amplitude:
            return False
        return True
```

## 中枢特征分析

### 基础特征
- **中枢高度**：high - low
- **中枢宽度**：时间跨度
- **中枢密度**：成交量/高度
- **振荡频率**：单位时间内振荡次数

### 衍生特征
```python
class ZSEigen:
    def extract_features(self, zs):
        features = {
            'height': zs.get_height(),
            'width': zs.get_time_span(),
            'center': zs.center,
            'strength': zs.calculate_strength(),
            'stability': zs.calculate_stability(),
            'breakout_potential': zs.calculate_breakout_potential()
        }
        return features
```

## 级别关系

### 中枢级别
- 同级别线段构成同级别中枢
- 高级别中枢由低级别中枢组成
- 支持多级别联立分析

### 级别转换
```python
def upgrade_zs_level(zs_list, target_level):
    # 将低级别中枢合并成高级别中枢
    upgraded_zs = []
    for zs_group in group_zs(zs_list):
        new_zs = merge_zs_group(zs_group)
        new_zs.level = target_level
        upgraded_zs.append(new_zs)
    return upgraded_zs
```

## 配置优化

### 不同市场的配置建议

**A股配置**：
```python
zs_config = CZSConfig(
    min_seg_count=3,
    overlap_threshold=0.01,
    enable_extension=True,
    extension_ratio=0.382
)
```

**数字货币配置**：
```python
zs_config = CZSConfig(
    min_seg_count=5,
    overlap_threshold=0.005,
    enable_extension=False,
    merge_distance=0.01
)
```

## 最佳实践

1. **参数调整**：根据不同品种选择合适的中枢参数
2. **级别选择**：分析时使用多个级别联立
3. **扩展控制**：合理控制中枢扩展次数
4. **合并策略**：根据分析目的决定是否合并
5. **动态更新**：实时更新中枢状态

## 扩展开发

### 添加新的中枢类型
```python
class SpecialZS(CZS):
    def __init__(self):
        super().__init__()
        self.special_feature = None

    def calculate_special_indicator(self):
        # 计算特殊指标
        pass
```

### 添加新的分析功能
```python
class ZSAnalyzer:
    def __init__(self, zs_list):
        self.zs_list = zs_list

    def predict_direction(self):
        # 预测突破方向
        pass

    def calculate_support_resistance(self):
        # 计算支撑阻力
        pass
```

## 注意事项

1. 中枢确认具有滞后性
2. 不同参数设置会产生不同的中枢
3. 扩展和合并会影响分析结果
4. 需要结合其他指标综合判断
5. 实际交易时需要考虑中枢的可靠性
# Math 模块

> **导航**：[首页](../CLAUDE.md) > Math 模块

## 概述

Math 模块提供了丰富的技术指标计算功能，包括趋势指标、动量指标、波动率指标等。这些技术指标与缠论分析结合使用，可以提高分析的准确性和可靠性。

## 核心组件

### 1. MACD.py - 平滑异同移动平均线

**功能**：计算 MACD 指标，用于判断趋势和背驰

**核心类**：`CMACD`

**参数配置**：
```python
class MACDConfig:
    fast_period = 12          # 快线周期
    slow_period = 26          # 慢线周期
    signal_period = 9         # 信号线周期
}

class CMACD:
    def __init__(self, config=None):
        self.config = config or MACDConfig()
        self.macd_line = []       # MACD线
        self.signal_line = []     # 信号线
        self.histogram = []       # 柱状图
```

**计算方法**：
```python
def calculate_macd(close_prices):
    # 1. 计算EMA
    ema_fast = calculate_ema(close_prices, fast_period)
    ema_slow = calculate_ema(close_prices, slow_period)

    # 2. 计算DIF
    dif = ema_fast - ema_slow

    # 3. 计算DEA
    dea = calculate_ema(dif, signal_period)

    # 4. 计算MACD柱
    histogram = (dif - dea) * 2

    return dif, dea, histogram
```

**背驰判断**：
```python
def check_divergence(price, macd, n=3):
    # 1. 识别价格新高/新低
    price_highs = find_highs(price, n)
    macd_highs = find_highs(macd, n)

    # 2. 判断背驰
    if len(price_highs) >= 2 and len(macd_highs) >= 2:
        if price_highs[-1] > price_highs[-2] and \
           macd_highs[-1] < macd_highs[-2]:
            return "top_divergence"
        elif price_highs[-1] < price_highs[-2] and \
             macd_highs[-1] > macd_highs[-2]:
            return "bottom_divergence"
    return None
```

### 2. KDJ.py - 随机指标

**功能**：计算 KDJ 指标，用于判断超买超卖

**核心类**：`CKDJ`

**参数配置**：
```python
class KDJConfig:
    n_period = 9            # 周期
    k_period = 3            # K值平滑
    d_period = 3            # D值平滑
    j_period = 3            # J值平滑
}

class CKDJ:
    def __init__(self, config=None):
        self.config = config or KDJConfig()
        self.k_line = []         # K值
        self.d_line = []         # D值
        self.j_line = []         # J值
```

**计算方法**：
```python
def calculate_kdj(high, low, close):
    # 1. 计算RSV
    rsv = calculate_rsv(high, low, close, n_period)

    # 2. 计算K值
    k = calculate_ema(rsv, k_period) * 100

    # 3. 计算D值
    d = calculate_ema(k, d_period)

    # 4. 计算J值
    j = 3 * k - 2 * d

    return k, d, j
```

### 3. RSI.py - 相对强弱指数

**功能**：计算 RSI 指标，衡量价格变动的速度和变化幅度

**核心类**：`CRSI`

**参数配置**：
```python
class RSIConfig:
    period = 14               # 计算周期
}

def calculate_rsi(prices, period):
    # 1. 计算价格变动
    changes = prices.diff()

    # 2. 分离涨跌
    gains = changes.where(changes > 0, 0)
    losses = -changes.where(changes < 0, 0)

    # 3. 计算平均涨跌幅
    avg_gains = gains.rolling(window=period).mean()
    avg_losses = losses.rolling(window=period).mean()

    # 4. 计算RSI
    rs = avg_gains / avg_losses
    rsi = 100 - (100 / (1 + rs))

    return rsi
```

### 4. BOLL.py - 布林带

**功能**：计算布林带，用于判断价格通道和波动率

**核心类**：`CBOLL`

**参数配置**：
```python
class BOLLConfig:
    period = 20              # 周期
    std_dev = 2              # 标准差倍数
}

def calculate_boll(prices, period, std_dev):
    # 1. 计算中轨（MA）
    middle = prices.rolling(window=period).mean()

    # 2. 计算标准差
    std = prices.rolling(window=period).std()

    # 3. 计算上下轨
    upper = middle + std * std_dev
    lower = middle - std * std_dev

    return upper, middle, lower
```

### 5. TrendLine.py - 趋势线

**功能**：识别和绘制趋势线

**核心类**：`CTrendLine`

**趋势线类型**：
```python
class TrendLineType(Enum):
    SUPPORT = "support"       # 支撑线
    RESISTANCE = "resistance" # 阻力线
    CHANNEL = "channel"       # 通道线
```

**算法实现**：
```python
def find_trend_line(highs, lows, min_points=3, max_deviation=0.02):
    # 1. 寻找潜在的转折点
    pivots = find_pivots(highs, lows)

    # 2. 尝试连接趋势线
    trend_lines = []
    for p1_idx in range(len(pivots)):
        for p2_idx in range(p1_idx + 1, len(pivots)):
            line = create_line(pivots[p1_idx], pivots[p2_idx])

            # 3. 验证趋势线
            if validate_trend_line(line, pivots, max_deviation):
                trend_lines.append(line)

    # 4. 选择最佳趋势线
    return select_best_lines(trend_lines, min_points)
```

### 6. Demark.py - Demark 指标

**功能**：实现 Demark 系列指标

**指标类型**：
- TD Sequential
- TD Combo
- TD D-Wave
- TD Trend Factor

**TD Sequential 实现**：
```python
def calculate_td_sequential(close):
    # 1. 计算TD买序列
    td_buy_seq = calculate_td_sequence(
        close,
        compare_func=lambda x, y: x < y,  # 收盘价低于4天前
        min_count=9
    )

    # 2. 计算TD卖序列
    td_sell_seq = calculate_td_sequence(
        close,
        compare_func=lambda x, y: x > y,  # 收盘价高于4天前
        min_count=9
    )

    # 3. 寻找TD翻转
    td_flip = find_td_flip(close)

    return td_buy_seq, td_sell_seq, td_flip
```

## 使用示例

### MACD 使用
```python
from Math.MACD import CMACD
from Math.MACD import MACDConfig

# 创建配置
config = MACDConfig(
    fast_period=12,
    slow_period=26,
    signal_period=9
)

# 计算MACD
macd = CMACD(config)
macd.calculate(close_prices)

# 获取结果
dif, dea, histogram = macd.get_values()

# 判断背驰
divergence = macd.check_divergence(close_prices)
```

### 多指标组合使用
```python
from Math.KDJ import CKDJ
from Math.RSI import CRSI
from Math.BOLL import CBOLL

# 计算多个指标
kdj = CKDJ()
kdj.calculate(high, low, close)

rsi = CRSI()
rsi.calculate(close)

boll = CBOLL()
boll.calculate(close)

# 综合判断
def generate_signal():
    signals = []

    # 超买超卖判断
    if kdj.j > 100:
        signals.append("OVERBOUGHT")
    elif kdj.j < 0:
        signals.append("OVERSOLD")

    # RSI 极端值
    if rsi.value > 70:
        signals.append("RSI_OVERBOUGHT")
    elif rsi.value < 30:
        signals.append("RSI_OVERSOLD")

    # 布林带突破
    if close[-1] > boll.upper[-1]:
        signals.append("BOLL_BREAK_UP")
    elif close[-1] < boll.lower[-1]:
        signals.append("BOLL_BREAK_DOWN")

    return signals
```

## 性能优化

### 向量化计算
```python
import numpy as np

def optimized_macd(close, fast=12, slow=26, signal=9):
    # 使用 numpy 加速计算
    close = np.array(close)
    ema_fast = ema(close, fast)
    ema_slow = ema(close, slow)
    dif = ema_fast - ema_slow
    dea = ema(dif, signal)
    histogram = (dif - dea) * 2
    return dif, dea, histogram

def ema(data, period):
    alpha = 2 / (period + 1)
    ema_data = np.zeros_like(data)
    ema_data[0] = data[0]
    for i in range(1, len(data)):
        ema_data[i] = alpha * data[i] + (1 - alpha) * ema_data[i-1]
    return ema_data
```

### 缓存机制
```python
class CachedIndicator:
    def __init__(self):
        self.cache = {}

    def calculate(self, data, indicator_type, params):
        # 生成缓存键
        cache_key = f"{indicator_type}_{hash(tuple(params))}_{len(data)}"

        # 检查缓存
        if cache_key in self.cache:
            return self.cache[cache_key]

        # 计算新值
        result = self._calculate(data, indicator_type, params)

        # 存入缓存
        self.cache[cache_key] = result
        return result
```

## 自定义指标开发

### 创建自定义指标
```python
class CustomIndicator:
    def __init__(self, params):
        self.params = params

    def calculate(self, data):
        # 实现自定义算法
        result = []
        for i in range(len(data)):
            # 计算逻辑
            value = self._calculate_single(data[:i+1])
            result.append(value)
        return result

    def _calculate_single(self, data_slice):
        # 单点计算逻辑
        pass

    def get_signal(self, values):
        # 生成交易信号
        if values[-1] > self.threshold:
            return "BUY"
        elif values[-1] < -self.threshold:
            return "SELL"
        return "HOLD"
```

## 最佳实践

1. **参数优化**：使用历史数据优化指标参数
2. **指标组合**：多个指标结合使用，提高准确性
3. **级别选择**：不同时间周期选择合适参数
4. **信号过滤**：过滤掉噪音信号
5. **实时更新**：使用增量计算提高效率

## 注意事项

1. 指标具有滞后性，需要综合判断
2. 不同市场环境需要不同参数
3. 极端行情下指标可能失效
4. 注意指标的适用范围
5. 与缠论结合时，指标作为辅助判断
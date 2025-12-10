# Debug 模块

> **导航**：[首页](../CLAUDE.md) > Debug 模块

## 概述

Debug 模块提供了策略示例和调试工具，帮助开发者理解和学习如何使用缠论框架进行策略开发。虽然公开版本只包含基础示例，但展示了框架的基本用法和最佳实践。

## 策略示例

### 1. strategy_demo.py - 基础策略示例

**功能**：展示基础的缠论分析流程

**主要功能**：
- 数据加载和预处理
- 缠论分析初始化
- 基础买卖点识别
- 简单的策略逻辑

**代码结构**：
```python
from Chan import CChan
from ChanConfig import CChanConfig
from Common.CEnum import DATA_SRC, KL_TYPE

def demo_basic_strategy():
    """基础策略演示"""
    # 1. 配置
    config = CChanConfig()

    # 2. 初始化
    chan = CChan(
        code="000001",
        begin_time="2020-01-01",
        end_time="2023-12-31",
        data_src=DATA_SRC.BAO_STOCK,
        lv_list=[KL_TYPE.K_DAY, KL_TYPE.K_60M],
        config=config
    )

    # 3. 获取分析结果
    bi_list = chan.bi_list
    seg_list = chan.seg_list
    zs_list = chan.zs_list
    bsp_list = chan.bsp_list

    # 4. 简单策略逻辑
    signals = generate_simple_signals(bi_list, bsp_list)

    # 5. 输出结果
    print_results(signals)

def generate_simple_signals(bi_list, bsp_list):
    """生成简单信号"""
    signals = []
    position = 0  # 0: 空仓, 1: 持仓

    for bsp in bsp_list:
        if bsp.point_type in [BSP_TYPE.BUY1, BSP_TYPE.BUY2, BSP_TYPE.BUY3]:
            if position == 0:
                signals.append({
                    'time': bsp.time,
                    'action': 'BUY',
                    'price': bsp.price,
                    'type': bsp.point_type
                })
                position = 1

        elif bsp.point_type in [BSP_TYPE.SELL1, BSP_TYPE.SELL2, BSP_TYPE.SELL3]:
            if position == 1:
                signals.append({
                    'time': bsp.time,
                    'action': 'SELL',
                    'price': bsp.price,
                    'type': bsp.point_type
                })
                position = 0

    return signals
```

### 2. strategy_demo2.py - 多级别策略

**功能**：展示多级别联立分析

**特点**：
- 使用日线和60分钟线
- 区间套策略实现
- 多级别信号确认

**核心逻辑**：
```python
def demo_multi_level_strategy():
    """多级别策略演示"""
    # 配置多级别
    lv_list = [KL_TYPE.K_DAY, KL_TYPE.K_60M]

    # 初始化
    chan = CChan(
        code="000001",
        lv_list=lv_list,
        config=get_multi_level_config()
    )

    # 获取各级别数据
    day_chan = chan.lv_data[KL_TYPE.K_DAY]
    hour_chan = chan.lv_data[KL_TYPE.K_60M]

    # 区间套策略
    signals = interval_trading_strategy(day_chan, hour_chan)

    return signals

def interval_trading_strategy(day_chan, hour_chan):
    """区间套交易策略"""
    signals = []

    # 1. 大级别定方向
    day_trend = get_major_trend(day_chan)

    # 2. 小级别找时机
    for hour_bsp in hour_chan.bsp_list:
        # 检查是否与大级别同向
        if is_signal_aligned(hour_bsp, day_trend):
            signals.append({
                'time': hour_bsp.time,
                'action': 'BUY' if hour_bsp.is_buy() else 'SELL',
                'level': '60M',
                'aligned_with_daily': True
            })

    return signals
```

### 3. strategy_demo3.py - 技术指标结合

**功能**：展示缠论与技术指标结合使用

**特点**：
- MACD背驰确认
- 成交量分析
- 多指标过滤

**实现示例**：
```python
def demo_indicator_combined_strategy():
    """指标结合策略演示"""
    chan = CChan(
        code="000001",
        config=get_indicator_config()
    )

    # 计算技术指标
    macd_data = calculate_macd(chan.kline_list)
    volume_data = calculate_volume_indicators(chan.kline_list)

    # 生成信号
    signals = generate_combined_signals(
        chan.bsp_list,
        macd_data,
        volume_data
    )

    return signals

def generate_combined_signals(bsp_list, macd_data, volume_data):
    """生成组合信号"""
    signals = []

    for bsp in bsp_list:
        # 获取对应时间的指标值
        macd_value = get_indicator_at_time(macd_data, bsp.time)
        volume_ratio = get_volume_ratio(volume_data, bsp.time)

        # 信号确认
        if bsp.is_buy():
            # 买点需要MACD配合
            if macd_value['histogram'] > 0 and volume_ratio > 1.2:
                signals.append({
                    'time': bsp.time,
                    'action': 'BUY',
                    'confidence': 'HIGH',
                    'macd_confirmed': True,
                    'volume_confirmed': True
                })

        elif bsp.is_sell():
            # 卖点需要MACD绿柱
            if macd_value['histogram'] < 0:
                signals.append({
                    'time': bsp.time,
                    'action': 'SELL',
                    'confidence': 'HIGH',
                    'macd_confirmed': True
                })

    return signals
```

### 4. strategy_demo4.py - 回测框架

**功能**：展示简单的回测实现

**特点**：
- 交易成本计算
- 绩效指标计算
- 风险控制

**回测框架**：
```python
class SimpleBacktest:
    def __init__(self, initial_capital=100000):
        self.initial_capital = initial_capital
        self.capital = initial_capital
        self.position = 0
        self.trades = []
        self.daily_values = []

    def execute_trade(self, signal):
        """执行交易"""
        action = signal['action']
        price = signal['price']
        time = signal['time']

        if action == 'BUY' and self.position == 0:
            # 买入
            shares = self.capital / price
            cost = shares * price * (1 + 0.001)  # 0.1% 手续费

            self.position = shares
            self.capital = 0

            self.trades.append({
                'time': time,
                'action': 'BUY',
                'price': price,
                'shares': shares,
                'cost': cost
            })

        elif action == 'SELL' and self.position > 0:
            # 卖出
            proceeds = self.position * price * (1 - 0.001)  # 0.1% 手续费

            self.capital = proceeds
            shares = self.position
            self.position = 0

            self.trades.append({
                'time': time,
                'action': 'SELL',
                'price': price,
                'shares': shares,
                'proceeds': proceeds
            })

    def calculate_performance(self):
        """计算绩效指标"""
        total_return = (self.capital - self.initial_capital) / self.initial_capital

        # 计算其他指标
        win_trades = [t for t in self.trades if t['action'] == 'SELL' and t['proceeds'] > t['cost']]
        win_rate = len(win_trades) / len([t for t in self.trades if t['action'] == 'SELL'])

        return {
            'total_return': total_return,
            'win_rate': win_rate,
            'total_trades': len(self.trades) // 2
        }

def run_backtest_demo():
    """运行回测演示"""
    # 生成信号
    signals = generate_signals()

    # 执行回测
    backtest = SimpleBacktest()
    for signal in signals:
        backtest.execute_trade(signal)

    # 计算绩效
    performance = backtest.calculate_performance()
    print(f"总收益率: {performance['total_return']:.2%}")
    print(f"胜率: {performance['win_rate']:.2%}")
```

## 调试工具

### 1. 数据可视化
```python
def visualize_chan_analysis(chan_obj):
    """可视化缠论分析结果"""
    import matplotlib.pyplot as plt

    fig, axes = plt.subplots(3, 1, figsize=(20, 12))

    # K线图
    plot_kline(chan_obj.kline_list, axes[0])
    plot_bi(chan_obj.bi_list, axes[0])
    plot_seg(chan_obj.seg_list, axes[0])
    plot_zs(chan_obj.zs_list, axes[0])

    # 买卖点
    plot_bsp(chan_obj.bsp_list, axes[0])

    # 成交量
    plot_volume(chan_obj.kline_list, axes[1])

    # MACD
    plot_macd(chan_obj.macd_data, axes[2])

    plt.tight_layout()
    plt.show()
```

### 2. 数据导出
```python
def export_analysis_data(chan_obj, filename):
    """导出分析数据"""
    import pandas as pd

    # 准备数据
    data = {
        'time': [k.time for k in chan_obj.kline_list],
        'open': [k.open for k in chan_obj.kline_list],
        'high': [k.high for k in chan_obj.kline_list],
        'low': [k.low for k in chan_obj.kline_list],
        'close': [k.close for k in chan_obj.kline_list],
    }

    # 添加笔标记
    bi_markers = [0] * len(chan_obj.kline_list)
    for bi in chan_obj.bi_list:
        idx = find_kline_index(chan_obj.kline_list, bi.start.time)
        if idx >= 0:
            bi_markers[idx] = 1 if bi.direction == BI_DIR.UP else -1
    data['bi_marker'] = bi_markers

    # 保存到CSV
    df = pd.DataFrame(data)
    df.to_csv(filename, index=False)
```

### 3. 日志记录
```python
import logging

def setup_logger():
    """设置日志"""
    logging.basicConfig(
        level=logging.INFO,
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        handlers=[
            logging.FileHandler('chan_analysis.log'),
            logging.StreamHandler()
        ]
    )
    return logging.getLogger('ChanDemo')

def log_analysis_summary(chan_obj):
    """记录分析摘要"""
    logger = logging.getLogger('ChanDemo')

    logger.info(f"分析代码: {chan_obj.code}")
    logger.info(f"K线数量: {len(chan_obj.kline_list)}")
    logger.info(f"笔数量: {len(chan_obj.bi_list)}")
    logger.info(f"线段数量: {len(chan_obj.seg_list)}")
    logger.info(f"中枢数量: {len(chan_obj.zs_list)}")
    logger.info(f"买卖点数量: {len(chan_obj.bsp_list)}")
```

## 使用指南

### 运行示例
```bash
# 基础策略
python Debug/strategy_demo.py

# 多级别策略
python Debug/strategy_demo2.py

# 指标结合策略
python Debug/strategy_demo3.py

# 回测演示
python Debug/strategy_demo4.py
```

### 自定义策略开发
1. 复制示例文件作为模板
2. 修改配置参数
3. 实现自定义信号逻辑
4. 添加过滤条件
5. 执行回测验证

## 最佳实践

1. **从简单开始**：先运行基础示例理解框架
2. **逐步增加复杂度**：慢慢添加新的功能
3. **充分测试**：在历史数据上充分验证
4. **风险控制**：始终考虑止损和仓位管理
5. **持续优化**：根据实际表现调整策略

## 注意事项

1. 示例代码仅供学习，实盘需要更多优化
2. 注意交易成本和滑点的影响
3. 不同市场需要不同的参数
4. 策略需要定期评估和调整
5. 严格遵守风险管理原则

*注意：Debug 模块提供了基础示例，完整版包含更多策略和工具。*
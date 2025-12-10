# Plot 模块

> **导航**：[首页](../CLAUDE.md) > Plot 模块

## 概述

Plot 模块提供了强大的图表绘制功能，专门用于可视化缠论分析结果。支持绘制 K线、笔、线段、中枢、买卖点等多种元素，并支持交互式图表和动画演示。

## 核心组件

### 1. PlotDriver.py - 绘图驱动器

**功能**：核心绘图引擎，负责生成各种图表

**核心类**：`CPlotDriver`

**主要功能**：
- K线图绘制
- 缠论元素叠加显示
- 技术指标绘制
- 多子图布局
- 交互功能支持

**关键方法**：
```python
class CPlotDriver:
    def __init__(self, figsize=(20, 12), style="chan"):
        self.figsize = figsize
        self.style = style
        self.axes = []
        self.data = {}

    def plot_kline(self, kline_data, ax=None):           # 绘制K线
    def plot_bi(self, bi_list, ax=None):                 # 绘制笔
    def plot_seg(self, seg_list, ax=None):               # 绘制线段
    def plot_zs(self, zs_list, ax=None):                 # 绘制中枢
    def plot_bsp(self, bsp_list, ax=None):               # 绘制买卖点
    def plot_indicator(self, data, indicator, ax=None):   # 绘制指标
    def save_plot(self, filename):                        # 保存图表
    def show_plot(self):                                  # 显示图表
```

**图表布局**：
```python
def create_figure_layout():
    fig = plt.figure(figsize=(20, 12))

    # 主图 - K线和缠论元素
    ax_main = plt.subplot2grid((4, 1), (0, 0), rowspan=3)

    # 子图1 - MACD
    ax_macd = plt.subplot2grid((4, 1), (3, 0))

    # 子图2 - 成交量（可选）
    ax_volume = plt.subplot2grid((4, 1), (3, 0), sharex=ax_main)

    return fig, [ax_main, ax_macd, ax_volume]
```

### 2. PlotMeta.py - 绘图元数据

**功能**：管理绘图的样式、颜色、配置等元数据

**核心配置**：
```python
class CPlotMeta:
    # 颜色配置
    colors = {
        'background': '#FFFFFF',
        'grid': '#E0E0E0',
        'text': '#333333',
        'up_candle': '#FF4444',
        'down_candle': '#44FF44',
        'bi_up': '#FF0000',
        'bi_down': '#00FF00',
        'seg_up': '#0000FF',
        'seg_down': '#FF00FF',
        'zs': '#FFFF00',
        'buy_point': '#00FFFF',
        'sell_point': '#FF00FF'
    }

    # 样式配置
    styles = {
        'bi_width': 2,
        'seg_width': 3,
        'zs_alpha': 0.3,
        'point_size': 50,
        'grid_alpha': 0.3
    }

    # 字体配置
    fonts = {
        'title': {'size': 16, 'weight': 'bold'},
        'label': {'size': 12},
        'legend': {'size': 10}
    }
```

### 3. AnimatePlotDriver.py - 动画绘图

**功能**：创建动态图表，展示缠论分析过程

**核心类**：`CAnimatePlotDriver`

**动画类型**：
- 逐步显示分析过程
- 买卖点出现动画
- 中枢演化过程
- 实时数据更新

**实现示例**：
```python
def create_analysis_animation(kline_data, step_data):
    fig, ax = plt.subplots(figsize=(20, 10))

    def animate(frame):
        ax.clear()

        # 逐步显示K线
        current_data = kline_data[:frame+1]
        plot_kline(current_data, ax)

        # 逐步显示分析结果
        if frame in step_data:
            for element in step_data[frame]:
                if element['type'] == 'bi':
                    plot_bi(element['data'], ax)
                elif element['type'] == 'seg':
                    plot_seg(element['data'], ax)

        return ax.artists

    anim = FuncAnimation(
        fig, animate,
        frames=len(kline_data),
        interval=100,
        blit=False
    )
    return anim
```

## 使用示例

### 基础绘图
```python
from Plot.PlotDriver import CPlotDriver
from Plot.PlotMeta import CPlotMeta

# 创建绘图驱动器
plotter = CPlotDriver(figsize=(20, 12))

# 准备数据
kline_data = get_kline_data()
bi_list = get_bi_list()
seg_list = get_seg_list()
zs_list = get_zs_list()
bsp_list = get_bsp_list()

# 绘制K线
plotter.plot_kline(kline_data)

# 绘制缠论元素
plotter.plot_bi(bi_list)
plotter.plot_seg(seg_list)
plotter.plot_zs(zs_list)
plotter.plot_bsp(bsp_list)

# 显示图表
plotter.show_plot()
```

### 自定义样式
```python
# 创建自定义样式
custom_meta = CPlotMeta()
custom_meta.colors['background'] = '#1E1E1E'
custom_meta.colors['text'] = '#FFFFFF'
custom_meta.styles['grid_alpha'] = 0.1

# 应用自定义样式
plotter = CPlotDriver(style="custom", plot_meta=custom_meta)
```

### 多指标组合
```python
def plot_comprehensive_analysis(chan_obj):
    fig, axes = plt.subplots(4, 1, figsize=(20, 16))

    # 主图 - K线和缠论
    plot_kline(chan_obj.kline_list, axes[0])
    plot_bi(chan_obj.bi_list, axes[0])
    plot_seg(chan_obj.seg_list, axes[0])
    plot_zs(chan_obj.zs_list, axes[0])
    plot_bsp(chan_obj.bsp_list, axes[0])
    axes[0].set_title("缠论分析", fontsize=16)

    # MACD
    plot_macd(chan_obj.macd_data, axes[1])
    axes[1].set_title("MACD", fontsize=14)

    # KDJ
    plot_kdj(chan_obj.kdj_data, axes[2])
    axes[2].set_title("KDJ", fontsize=14)

    # 成交量
    plot_volume(chan_obj.volume_data, axes[3])
    axes[3].set_title("成交量", fontsize=14)

    plt.tight_layout()
    return fig
```

### 交互式图表
```python
import plotly.graph_objects as go
from plotly.subplots import make_subplots

def create_interactive_plot(chan_obj):
    # 创建子图
    fig = make_subplots(
        rows=3, cols=1,
        shared_xaxes=True,
        vertical_spacing=0.05,
        row_heights=[0.6, 0.2, 0.2]
    )

    # K线图
    fig.add_trace(
        go.Candlestick(
            x=chan_obj.time_list,
            open=chan_obj.open_list,
            high=chan_obj.high_list,
            low=chan_obj.low_list,
            close=chan_obj.close_list,
            name="K线"
        ),
        row=1, col=1
    )

    # 笔
    for bi in chan_obj.bi_list:
        fig.add_trace(
            go.Scatter(
                x=[bi.start.time, bi.end.time],
                y=[bi.start.price, bi.end.price],
                mode='lines',
                line=dict(color='red' if bi.dir == UP else 'green', width=2),
                name=f"笔_{bi.idx}"
            ),
            row=1, col=1
        )

    # 设置布局
    fig.update_layout(
        title=f"{chan_obj.code} 缠论分析",
        xaxis_rangeslider_visible=False
    )

    return fig
```

## 图表元素详解

### K线样式
```python
def plot_candlestick(ax, open, high, low, close, **kwargs):
    # 上涨 - 红色空心
    # 下跌 - 绿色实心
    colors = ['red' if c >= o else 'green' for o, c in zip(open, close)]

    for i in range(len(open)):
        if close[i] >= open[i]:
            # 阳线
            ax.plot([i, i], [low[i], high[i]], color='red', linewidth=1)
            ax.add_patch(
                Rectangle(
                    (i-0.3, open[i]),
                    0.6, close[i]-open[i],
                    fill=False,
                    edgecolor='red'
                )
            )
        else:
            # 阴线
            ax.plot([i, i], [low[i], high[i]], color='green', linewidth=1)
            ax.add_patch(
                Rectangle(
                    (i-0.3, close[i]),
                    0.6, open[i]-close[i],
                    fill=True,
                    facecolor='green'
                )
            )
```

### 买卖点标记
```python
def plot_buypoint(ax, time, price, label=None):
    ax.scatter(
        time, price,
        marker='^',
        s=100,
        color='red',
        zorder=5,
        label=label
    )
    if label:
        ax.annotate(
            label,
            (time, price),
            xytext=(10, 10),
            textcoords='offset points',
            bbox=dict(boxstyle='round,pad=0.3', facecolor='yellow', alpha=0.7)
        )

def plot_sellpoint(ax, time, price, label=None):
    ax.scatter(
        time, price,
        marker='v',
        s=100,
        color='green',
        zorder=5,
        label=label
    )
```

### 中枢绘制
```python
def plot_pivot(ax, zs):
    # 绘制中枢矩形
    rect = Rectangle(
        (zs.start_time, zs.low),
        zs.end_time - zs.start_time,
        zs.high - zs.low,
        facecolor='yellow',
        alpha=0.3,
        edgecolor='orange',
        linewidth=2
    )
    ax.add_patch(rect)

    # 绘制中枢中心线
    ax.plot(
        [zs.start_time, zs.end_time],
        [zs.center, zs.center],
        '--',
        color='orange',
        linewidth=1
    )
```

## 配置文件支持

### plot_config.json
```json
{
    "layout": {
        "figure_size": [20, 12],
        "subplots": ["main", "macd", "volume"],
        "grid": {
            "enabled": true,
            "alpha": 0.3
        }
    },
    "colors": {
        "background": "#FFFFFF",
        "candle_up": "#FF0000",
        "candle_down": "#00FF00",
        "volume_up": "#FFAAAA",
        "volume_down": "#AAFFAA"
    },
    "elements": {
        "bi": {
            "width": 2,
            "show_labels": true
        },
        "seg": {
            "width": 3,
            "show_labels": true
        },
        "zs": {
            "alpha": 0.3,
            "show_center": true
        },
        "bsp": {
            "size": 50,
            "show_text": true
        }
    }
}
```

## 性能优化

### 大数据量处理
```python
def plot_large_dataset(data, chunk_size=1000):
    """分块绘制大数据集"""
    fig, ax = plt.subplots(figsize=(20, 10))

    # 设置初始显示范围
    ax.set_xlim(0, chunk_size)

    def on_scroll(event):
        """滚动事件处理"""
        if event.button == 'up':
            new_xlim = ax.get_xlim()
            ax.set_xlim(new_xlim[0] + chunk_size, new_xlim[1] + chunk_size)
        elif event.button == 'down':
            new_xlim = ax.get_xlim()
            ax.set_xlim(max(0, new_xlim[0] - chunk_size),
                       max(chunk_size, new_xlim[1] - chunk_size))
        fig.canvas.draw_idle()

    fig.canvas.mpl_connect('scroll_event', on_scroll)

    # 绘制初始数据
    plot_kline(data[:chunk_size], ax)

    return fig, ax
```

## 扩展功能

### 自定义绘图元素
```python
class CustomPlotElement:
    def __init__(self, data, style):
        self.data = data
        self.style = style

    def draw(self, ax):
        """自定义绘制逻辑"""
        raise NotImplementedError

class FibonacciLines(CustomPlotElement):
    def draw(self, ax):
        """绘制斐波那契线"""
        high = max(self.data)
        low = min(self.data)
        diff = high - low

        ratios = [0, 0.236, 0.382, 0.5, 0.618, 0.786, 1]
        for ratio in ratios:
            value = high - diff * ratio
            ax.axhline(
                y=value,
                color='blue',
                alpha=0.5,
                linestyle='--',
                label=f'{ratio*100}%'
            )
```

## 最佳实践

1. **合理布局**：根据分析需求设计合适的图表布局
2. **颜色搭配**：使用对比明显的颜色区分不同元素
3. **标签清晰**：重要元素添加标签说明
4. **交互设计**：提供缩放、平移等交互功能
5. **性能考虑**：大数据量时使用优化技术

## 注意事项

1. 图表仅供参考，实际决策需要综合分析
2. 注意不同市场的交易时间
3. 复权处理会影响图表显示
4. 导出图片时注意分辨率设置
5. 实时图表需要考虑性能问题
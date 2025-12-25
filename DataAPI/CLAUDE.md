# DataAPI 模块

> **导航**：[首页](../CLAUDE.md) > DataAPI 模块

## 概述

DataAPI 模块负责统一管理多种数据源的接入，为缠论分析提供标准化的数据接口。支持股票、期货、数字货币等多种金融数据源。

## 核心组件

### 1. CommonStockAPI.py - 通用股票数据接口

**功能**：定义数据接入的统一接口规范

**核心类**：
- `CCommonStockApi`: 所有数据源的基础类
  - 提供统一的 K线数据获取接口
  - 支持复权处理（前复权/后复权）
  - 时间范围过滤

**关键方法**：
```python
def get_kline(self, code, begin_time, end_time, kline_type, autype)
```

### 2. BaoStockAPI.py - BaoStock 数据源

**功能**：集成 baostock 数据源，提供A股历史数据

**特点**：
- 免费A股数据
- 支持日K、周K、月K等多种周期
- 自动处理停牌、退市等特殊情况
- 数据质量验证

**使用示例**：
```python
from DataAPI.BaoStockAPI import CBaoStockAPI

api = CBaoStockAPI()
kline_data = api.get_kline(
    code="sz.000001",
    begin_time="2020-01-01",
    end_time="2023-12-31",
    kline_type="d",
    autype="qfq"
)
```

### 3. ccxt.py - 数字货币交易所接口

**功能**：集成 CCXT 库，支持全球数百家数字货币交易所

**特点**：
- 支持 Binance、OKX、Huobi 等主流交易所
- 实时和历史数据获取
- 统一的数据格式转换
- API 限频处理

**支持的交易所**：
- Binance (币安)
- OKX (OKEx)
- Huobi (火币)
- Coinbase
- Kraken
- 等 100+ 交易所

### 4. csvAPI.py - CSV 文件数据源

**功能**：从本地 CSV 文件读取 K线数据

**特点**：
- 灵活的列映射配置
- 自动日期格式识别
- 数据类型转换
- 缺失值处理

**CSV 格式要求**：
```csv
date,time,open,high,low,close,volume
2023-01-01,09:30:00,10.0,10.5,9.8,10.2,1000000
```

### 5. AkshareAPI.py - Akshare 数据源

**功能**：集成 akshare 库，提供 A股和指数历史数据

**特点**：
- 免费开源的 A股数据接口
- 支持日线、周线、月线多种周期
- 支持前复权、后复权、不复权
- 同时支持个股和指数数据
- 无需登录认证

**使用示例**：
```python
from DataAPI.AkshareAPI import CAkshare
from Common.CEnum import DATA_SRC, KL_TYPE, AUTYPE

# 创建 CChan 时使用 Akshare 数据源
chan = CChan(
    code="000001",           # 股票代码
    begin_time="2020-01-01",
    end_time="2023-12-31",
    data_src=DATA_SRC.AKSHARE,
    lv_list=[KL_TYPE.K_DAY],
    autype=AUTYPE.QFQ        # 前复权
)
```

**支持的周期**：
- `K_DAY`: 日线
- `K_WEEK`: 周线
- `K_MON`: 月线

**指数识别**：
- 代码格式如 `sh000001`、`sz399001` 自动识别为指数
- 指数代码通常以 `000` 或 `399` 开头

## 数据源枚举

在 `Common/CEnum.py` 中定义：

```python
class DATA_SRC(Enum):
    BAO_STOCK = "baostock"      # BaoStock A股数据
    AKSHARE = "akshare"         # Akshare A股数据
    CCXT = "ccxt"               # 数字货币交易所
    CSV = "csv"                 # CSV 文件
    TDX = "tdx"                 # 通达信
    CUSTOM = "custom"           # 自定义数据源
```

## 数据格式规范

### K线数据结构
```python
{
    "time": CTime,              # 时间
    "open": float,              # 开盘价
    "high": float,              # 最高价
    "low": float,               # 最低价
    "close": float,             # 收盘价
    "volume": float,            # 成交量
    "turnover": float,          # 成交额（可选）
}
```

### 复权类型
```python
class AUTYPE(Enum):
    QFQ = "qfq"                 # 前复权
    HFQ = "hfq"                 # 后复权
    NONE = "none"               # 不复权
```

## 扩展新的数据源

### 步骤
1. 继承 `CCommonStockApi` 基类
2. 实现 `get_kline` 方法
3. 处理数据格式转换
4. 添加错误处理和重试机制
5. 更新 `DATA_SRC` 枚举

### 模板代码
```python
from DataAPI.CommonStockAPI import CCommonStockApi

class CCustomAPI(CCommonStockApi):
    def __init__(self, **kwargs):
        super().__init__()
        # 初始化参数

    def get_kline(self, code, begin_time, end_time, kline_type, autype):
        # 1. 获取原始数据
        raw_data = self._fetch_data(code, begin_time, end_time)

        # 2. 数据清洗和转换
        cleaned_data = self._clean_data(raw_data)

        # 3. 复权处理（如需要）
        if autype != AUTYPE.NONE:
            cleaned_data = self._adjust_price(cleaned_data, autype)

        return cleaned_data
```

## 最佳实践

1. **数据缓存**：频繁使用的数据应考虑缓存
2. **批量请求**：尽量使用批量接口减少请求次数
3. **错误处理**：实现完善的异常处理和重试机制
4. **数据验证**：对获取的数据进行质量检查
5. **限频控制**：遵守各数据源的 API 限制

## 注意事项

- 不同数据源的时间格式可能不同，需要统一转换
- 注意处理节假日和停牌情况
- 数字货币数据 24/7 交易，需要特殊处理
- 部分数据源需要认证，请妥善保管 API 密钥
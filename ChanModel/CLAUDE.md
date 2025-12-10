# ChanModel 模块

> **导航**：[首页](../CLAUDE.md) > ChanModel 模块

## 概述

ChanModel 模块负责机器学习相关的功能，主要是特征提取。虽然公开版本只包含了基础的特征定义，但为完整的机器学习框架预留了接口和扩展能力。

## 核心组件

### 1. Features.py - 特征提取

**功能**：定义和提取用于机器学习的特征

**特征类型**：

#### 基础价格特征
```python
class PriceFeatures:
    @staticmethod
    def extract_basic_features(kline_data):
        """提取基础价格特征"""
        features = {
            'open': kline_data.open,
            'high': kline_data.high,
            'low': kline_data.low,
            'close': kline_data.close,
            'volume': kline_data.volume,
            'turnover': kline_data.turnover,

            # 衍生特征
            'price_change': kline_data.close - kline_data.open,
            'price_change_pct': (kline_data.close - kline_data.open) / kline_data.open,
            'high_low_ratio': kline_data.high / kline_data.low,
            'close_position': (kline_data.close - kline_data.low) / (kline_data.high - kline_data.low),
            'volume_price_ratio': kline_data.volume / kline_data.close,
        }
        return features
```

#### 技术指标特征
```python
class IndicatorFeatures:
    @staticmethod
    def extract_ma_features(prices, windows=[5, 10, 20, 60]):
        """提取移动平均线特征"""
        features = {}
        for window in windows:
            ma = prices.rolling(window).mean()
            features[f'ma_{window}'] = ma
            features[f'price_ma_{window}_ratio'] = prices / ma
            features[f'ma_{window}_slope'] = ma.diff()
        return features

    @staticmethod
    def extract_volatility_features(prices, window=20):
        """提取波动率特征"""
        returns = prices.pct_change()
        features = {
            'volatility': returns.rolling(window).std(),
            'atr': calculate_atr(prices, window),
            'volatility_ratio': returns.rolling(window).std() / returns.rolling(window*2).std(),
        }
        return features
```

#### 缠论特征
```python
class ChanFeatures:
    @staticmethod
    def extract_bi_features(bi_list, current_time):
        """提取笔相关特征"""
        features = {}

        # 最近N笔的特征
        recent_bis = get_recent_bis(bi_list, current_time, n=5)

        for i, bi in enumerate(recent_bis):
            prefix = f'bi_{i}' if i > 0 else 'current_bi'
            features[f'{prefix}_length'] = bi.get_length()
            features[f'{prefix}_duration'] = bi.get_duration()
            features[f'{prefix}_slope'] = bi.get_slope()
            features[f'{prefix}_volume'] = bi.get_volume()
            features[f'{prefix}_direction'] = 1 if bi.direction == BI_DIR.UP else -1

        return features

    @staticmethod
    def extract_zs_features(zs_list, current_time):
        """提取中枢相关特征"""
        features = {}

        # 当前活跃中枢
        active_zs = find_active_zs(zs_list, current_time)
        if active_zs:
            features['zs_count'] = len(active_zs)
            features['zs_avg_strength'] = np.mean([zs.get_strength() for zs in active_zs])
            features['zs_distance_to_center'] = calculate_distance_to_center(active_zs, current_time)
        else:
            features['zs_count'] = 0
            features['zs_avg_strength'] = 0

        return features
```

## 特征工程框架

虽然公开版本简化了，但框架预留了完整的特征工程接口：

```python
class FeatureExtractor:
    def __init__(self, config):
        self.config = config
        self.extractors = {
            'price': PriceFeatures(),
            'indicator': IndicatorFeatures(),
            'chan': ChanFeatures(),
        }

    def extract_all_features(self, data, timestamp):
        """提取所有特征"""
        all_features = {}

        for name, extractor in self.extractors.items():
            if self.config.get(f'use_{name}_features', True):
                features = extractor.extract(data, timestamp)
                all_features.update(features)

        return all_features

    def transform(self, raw_data):
        """将原始数据转换为特征矩阵"""
        features_list = []

        for i, row in raw_data.iterrows():
            features = self.extract_all_features(raw_data[:i+1], row['time'])
            features_list.append(features)

        return pd.DataFrame(features_list)
```

## 使用示例

### 基础特征提取
```python
from ChanModel.Features import PriceFeatures, IndicatorFeatures

# 准备K线数据
kline_data = get_kline_data()

# 提取基础特征
price_features = PriceFeatures.extract_basic_features(kline_data)

# 提取技术指标特征
ma_features = IndicatorFeatures.extract_ma_features(kline_data.close)
volatility_features = IndicatorFeatures.extract_volatility_features(kline_data.close)

# 合并特征
all_features = {**price_features, **ma_features, **volatility_features}
```

### 缠论特征提取
```python
from ChanModel.Features import ChanFeatures

# 获取缠论分析结果
bi_list = chan_obj.bi_list
zs_list = chan_obj.zs_list
bsp_list = chan_obj.bsp_list

# 提取笔特征
bi_features = ChanFeatures.extract_bi_features(bi_list, current_time)

# 提取中枢特征
zs_features = ChanFeatures.extract_zs_features(zs_list, current_time)

# 提取买卖点特征
bsp_features = ChanFeatures.extract_bsp_features(bsp_list, current_time)
```

### 特征选择
```python
def select_features(features_df, target):
    """特征选择"""
    from sklearn.feature_selection import SelectKBest, f_regression

    # 移除缺失值
    features_df = features_df.fillna(0)

    # 特征选择
    selector = SelectKBest(score_func=f_regression, k=50)
    selected_features = selector.fit_transform(features_df, target)

    # 获取选中的特征名
    selected_names = features_df.columns[selector.get_support()]

    return pd.DataFrame(selected_features, columns=selected_names)
```

## 模型集成接口

框架预留了模型集成的接口：

```python
class ModelPredictor:
    def __init__(self, model_path=None):
        self.model = None
        self.feature_extractor = FeatureExtractor(config)

    def load_model(self, model_path):
        """加载训练好的模型"""
        # 预留模型加载接口
        pass

    def predict(self, data, current_time):
        """预测"""
        # 1. 提取特征
        features = self.feature_extractor.extract_all_features(data, current_time)

        # 2. 模型预测
        if self.model:
            prediction = self.model.predict([features])
            probability = self.model.predict_proba([features])
        else:
            prediction = None
            probability = None

        return {
            'features': features,
            'prediction': prediction,
            'probability': probability
        }
```

## 配置管理

### 特征配置
```json
{
    "features": {
        "price_features": {
            "enabled": true,
            "basic": ["open", "high", "low", "close", "volume"],
            "derived": ["price_change", "price_change_pct", "high_low_ratio"]
        },
        "indicator_features": {
            "enabled": true,
            "ma_windows": [5, 10, 20, 60],
            "rsi_period": 14,
            "macd_params": [12, 26, 9]
        },
        "chan_features": {
            "enabled": true,
            "bi_history": 5,
            "zs_history": 3,
            "bsp_history": 2
        }
    },
    "model": {
        "type": "classifier",
        "algorithms": ["random_forest", "xgboost", "neural_network"],
        "cross_validation": {
            "folds": 5,
            "shuffle": true,
            "random_state": 42
        }
    }
}
```

## 扩展开发

### 添加新的特征提取器
```python
class CustomFeatures:
    @staticmethod
    def extract_sentiment_features(data, news_data):
        """提取市场情绪特征"""
        # 整合新闻情绪分析
        sentiment_scores = analyze_news_sentiment(news_data)

        return {
            'news_sentiment': sentiment_scores.get('overall', 0),
            'news_count': len(news_data),
            'sentiment_change': sentiment_scores.get('change', 0)
        }

    @staticmethod
    def extract_macro_features(data, macro_data):
        """提取宏观经济特征"""
        return {
            'interest_rate': macro_data.get('interest_rate'),
            'inflation_rate': macro_data.get('inflation_rate'),
            'gdp_growth': macro_data.get('gdp_growth')
        }

# 注册到特征提取器
feature_extractor.register_extractor('custom', CustomFeatures())
```

### 实现模型训练接口
```python
class ModelTrainer:
    def __init__(self, config):
        self.config = config
        self.models = {}

    def prepare_data(self, raw_data, labels):
        """准备训练数据"""
        # 1. 特征提取
        feature_extractor = FeatureExtractor(self.config)
        features = feature_extractor.transform(raw_data)

        # 2. 数据清洗
        features = features.fillna(0)
        features = features.replace([np.inf, -np.inf], 0)

        # 3. 特征标准化
        from sklearn.preprocessing import StandardScaler
        scaler = StandardScaler()
        features_scaled = scaler.fit_transform(features)

        return features_scaled, labels, scaler

    def train_models(self, X, y):
        """训练多个模型"""
        from sklearn.ensemble import RandomForestClassifier
        from xgboost import XGBClassifier

        # 随机森林
        rf = RandomForestClassifier(n_estimators=100, random_state=42)
        rf_score = cross_val_score(rf, X, y, cv=5).mean()
        self.models['random_forest'] = {'model': rf, 'score': rf_score}

        # XGBoost
        xgb = XGBClassifier(random_state=42)
        xgb_score = cross_val_score(xgb, X, y, cv=5).mean()
        self.models['xgboost'] = {'model': xgb, 'score': xgb_score}

        return self.models
```

## 最佳实践

1. **特征选择**：使用相关性分析选择最有效的特征
2. **数据标准化**：对不同量纲的特征进行标准化
3. **时间序列处理**：避免未来数据泄露
4. **交叉验证**：使用时间序列交叉验证
5. **特征重要性**：分析特征重要性，优化特征集

## 注意事项

1. 特征工程是模型成功的关键
2. 避免过拟合，使用正则化
3. 注意特征的稳定性和可解释性
4. 考虑特征的计算成本
5. 定期更新特征和模型

*注意：当前公开版本仅包含基础框架，完整的机器学习功能需要完整版支持。*
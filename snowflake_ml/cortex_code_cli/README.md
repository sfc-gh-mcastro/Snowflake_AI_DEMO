# Stock Price Prediction & Algorithmic Trading

A Snowflake ML notebook implementing an intelligent stock trading system using XGBoost for price prediction and automated portfolio management.

## 📋 Problem Statement

Implement an algorithmic trading system that:
- Predicts stock price movements
- Makes optimal BUY/SELL decisions
- Maximizes profit starting from $100
- Handles constraints (cash availability, share ownership)
- Scoring: `5 × ln(final_money)`

## 🏗️ Architecture

```
┌─────────────────┐
│  Stock Prices   │
│  (5-day history)│
└────────┬────────┘
         │
         ▼
┌─────────────────────────────┐
│  Feature Engineering        │
│  - Moving Averages          │
│  - Momentum Indicators      │
│  - Volatility Measures      │
│  - Technical Indicators     │
└────────┬────────────────────┘
         │
         ▼
┌─────────────────────────────┐
│  XGBoost ML Model           │
│  - Predict next-day returns │
│  - Confidence scores        │
└────────┬────────────────────┘
         │
         ▼
┌─────────────────────────────┐
│  Trading Strategy           │
│  - Risk Management          │
│  - Position Sizing          │
│  - Buy/Sell Decisions       │
└────────┬────────────────────┘
         │
         ▼
┌─────────────────────────────┐
│  Portfolio Management       │
│  - Execute Transactions     │
│  - Track Holdings           │
│  - State Persistence        │
└─────────────────────────────┘
```

## 🚀 Quick Start

### 1. Deploy to Snowflake

#### Option A: Via Snowflake UI
1. Go to Snowflake → **Notebooks**
2. Click **+ Notebook**
3. Select **Run on container**
4. Choose **Runtime version**: CPU (or GPU for faster training)
5. Select **Compute pool**: `SYSTEM_COMPUTE_POOL_CPU`
6. Upload `stock_prediction.ipynb`

#### Option B: Via Cortex Code CLI
```bash
# From project directory
snow notebook execute stock_prediction.ipynb \
  --database ML \
  --schema PROJECTS \
  --compute-pool SYSTEM_COMPUTE_POOL_CPU
```

### 2. Run the Notebook

Execute all cells sequentially:
1. **Setup** (Cell 1): Initialize environment
2. **Data Structures** (Cells 2-3): Define portfolio and features
3. **ML Model** (Cell 4): XGBoost predictor
4. **Strategy** (Cells 5-6): Trading logic and state management
5. **Main Function** (Cell 7): `printTransactions()`
6. **Testing** (Cell 8): Validate with sample data

### 3. Test with Sample Input

```python
# Sample input from problem description
printTransactions(
    m=90.0,
    k=2,
    d=400,
    name=['iStreet', 'HR'],
    owned=[10, 0],
    prices=[
        [4.54, 5.53, 6.56, 5.54, 7.60],
        [30.54, 27.53, 24.42, 20.11, 17.50]
    ]
)
```

**Expected Output:**
```
2
iStreet SELL 10
HR BUY 5
```

## 📊 Components

### 1. Portfolio Management
- **Cash tracking**: Available vs. pending (from sales)
- **Holdings**: Track shares owned per stock
- **Transactions**: Complete history with P&L
- **Constraints**: Enforce cash availability rules

### 2. Feature Engineering
Technical indicators calculated from 5-day price history:
- **Moving Averages**: 3-day, 5-day
- **Momentum**: 3-day, 5-day price momentum
- **Volatility**: Standard deviation of returns
- **RSI**: Relative Strength Index (simplified)
- **Trend**: Price vs. MA comparison

### 3. ML Model: XGBoost
- **Input**: Technical features from price history
- **Output**: Predicted next-day return
- **Training**: Incremental learning as data accumulates
- **Validation**: RMSE and directional accuracy
- **Persistence**: Model saved between trading days

### 4. Trading Strategy
**Buy Rules:**
- Predicted return > 2% (configurable)
- Sufficient cash available
- Position size < 35% of portfolio
- Maintain 15% cash reserve

**Sell Rules:**
- Predicted decline > 2%
- Take profit at 15% gain
- Last day: liquidate all positions
- Stop loss triggers

**Risk Management:**
- Maximum position size per stock
- Diversification (max 3-4 stocks)
- Cash reserves for opportunities
- Position sizing based on confidence

### 5. State Persistence
Maintains state across trading days:
- `portfolio.json`: Cash, holdings, transactions
- `price_history.json`: Extended price data (100 days)
- `model.pkl`: Trained XGBoost model

## 🎯 Strategy Details

### Phase 1: Model Training (Days 1-10)
- Collect price data
- Build training dataset from historical windows
- Train XGBoost when sufficient data available (>30 examples)
- Use simple momentum strategy until model trained

### Phase 2: Active Trading (Days 11-399)
- Predict next-day returns for all stocks
- Rank stocks by predicted return
- Sell declining positions
- Buy top 3 promising stocks
- Maintain risk limits

### Phase 3: Liquidation (Day 400)
- Sell all positions at market price
- Maximize final cash value
- Calculate final score

## 📈 Performance Metrics

The system tracks:
- **Total Portfolio Value**: Cash + stock holdings
- **Return**: % gain/loss from $100 initial
- **Score**: `5 × ln(total_value)` (optimization target)
- **Win Rate**: % of profitable trades
- **Directional Accuracy**: Model prediction correctness

## 🔧 Configuration

Adjust strategy parameters in `TradingStrategy` class:

```python
self.buy_threshold = 0.02        # Buy if predicted return > 2%
self.sell_threshold = -0.02      # Sell if predicted decline > 2%
self.max_position_pct = 0.35     # Max 35% per stock
self.min_cash_reserve = 0.15     # Keep 15% cash
self.take_profit_threshold = 0.15 # Take profit at 15%
```

## 🧪 Testing

### Unit Testing
```python
# Test individual components
test_features = calculate_technical_features([4.54, 5.53, 6.56, 5.54, 7.60])
print(test_features)
```

### Integration Testing
```python
# Test with sample input
test_sample_input()
```

### Simulation
```python
# Simulate 10 trading days with synthetic data
simulate_trading_days(10)
```

### Reset State
```python
# Clear all saved state to start fresh
reset_state()
```

## 📁 File Structure

```
snowflake-ml-stock-prediction/
├── stock_prediction.ipynb      # Main notebook
├── README.md                   # This file
├── requirements.txt            # Python dependencies
└── test_data.json             # Sample test inputs
```

## 🔑 Key Features

✅ **Machine Learning**: XGBoost regression for price prediction
✅ **Feature Engineering**: 15+ technical indicators
✅ **Risk Management**: Position sizing, stop losses, diversification
✅ **State Persistence**: Continuous learning across days
✅ **Constraint Handling**: Cash availability, share ownership
✅ **Performance Tracking**: Complete transaction history
✅ **Snowflake Integration**: Runs on Container Runtime

## 🚧 Advanced Usage

### Connect to Snowflake Tables

Replace manual input with data from Snowflake tables:

```python
# Load stock data from Snowflake
df = session.table("MARKET_DATA.STOCKS.DAILY_PRICES") \
    .filter(col("date") == current_date()) \
    .select("symbol", "price", "date")

# Transform to expected format
stock_data = process_snowflake_data(df)

# Call trading function
printTransactions(m, k, d, names, owned, prices)
```

### Register Model in Model Registry

```python
from snowflake.ml.registry import Registry

reg = Registry(session=session, database_name="ML", schema_name="MODELS")

mv = reg.log_model(
    predictor.model,
    model_name="stock_price_predictor",
    version_name="v1",
    sample_input_data=X_train.head(10),
    task=task.Task.TABULAR_REGRESSION,
    target_platforms=["SNOWPARK_CONTAINER_SERVICES"],
    metrics={"rmse": validation_rmse, "accuracy": direction_accuracy}
)
```

### Set Up ML Observability

```python
from snowflake.ml.monitoring import ModelMonitor

monitor = ModelMonitor(session)

# Log predictions
monitor.log_predictions(
    model_name="stock_price_predictor",
    model_version="v1",
    prediction_data=predictions_df
)

# Log actuals (next day)
monitor.log_actuals(
    model_name="stock_price_predictor",
    model_version="v1",
    actual_data=actuals_df
)
```

## 🎓 Learning Resources

- [Snowflake ML Overview](https://docs.snowflake.com/en/developer-guide/snowflake-ml/overview)
- [Model Registry Guide](https://docs.snowflake.com/en/developer-guide/snowflake-ml/model-registry/overview)
- [Notebooks on Container Runtime](https://docs.snowflake.com/en/developer-guide/snowflake-ml/notebooks-on-spcs)
- [XGBoost Documentation](https://xgboost.readthedocs.io/)

## 📝 Game Rules Summary

### Input Format
```
m k d
name_1 owned_1 price_1_t-4 price_1_t-3 price_1_t-2 price_1_t-1 price_1_t
name_2 owned_2 price_2_t-4 price_2_t-3 price_2_t-2 price_2_t-1 price_2_t
...
```

### Output Format
```
N
stock_1 ACTION shares_1
stock_2 ACTION shares_2
...
```

### Constraints
- Money from sales available **next day only**
- Can only sell shares you own
- Can only buy with available cash
- Final day: all stocks sold automatically

### Scoring
```
score = 5 × ln(final_total_value)
```

## 🤝 Contributing

To improve the trading strategy:
1. Tune hyperparameters in `TradingStrategy`
2. Add new technical indicators in `calculate_technical_features()`
3. Experiment with different ML models (LSTM, ensemble)
4. Implement advanced features (sentiment analysis, news data)

## 📄 License

This project is for educational purposes as part of Snowflake ML skill development.

## 🆘 Troubleshooting

**Model not training?**
- Check if enough data accumulated (need 30+ examples)
- Verify price_history.json exists and has data

**Transactions failing?**
- Verify cash constraints (can't spend sale proceeds same day)
- Check share ownership (can't sell more than owned)

**Poor performance?**
- Adjust strategy thresholds (`buy_threshold`, `sell_threshold`)
- Tune model hyperparameters in `XGBRegressor`
- Add more features to `calculate_technical_features()`

**State issues?**
- Run `reset_state()` to clear all saved files
- Check STATE_DIR permissions (`/tmp/stock_trading_state`)

---

**Built with Snowflake ML** | **Powered by XGBoost** | **Optimized for Container Runtime**

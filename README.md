# UMC: Fibonacci-Based Stock Strategy - Automated Screening & Event-Driven Backtesting

## Project Overview

**UMC (Unexpected Move Concept)** is a sophisticated quantitative trading system that identifies and capitalizes on momentum-reversion asymmetry in Indian equities. The strategy exploits a behavioral pattern: parabolic rallies (≥14% single-direction moves) exhibit predictable mean-reversion at deep Fibonacci retracements due to profit-taking exhaustion and institutional anchoring bias.

This Python-based system performs three core functions:
1. **Automated Screening**: Scans 200+ NSE equities for qualified parabolic rallies using daily OHLC data
2. **Signal Generation**: Identifies high-probability entry zones at 0.786 Fibonacci retracement levels with dynamic peak tracking
3. **Event-Driven Backtesting**: Simulates futures trading with realistic margin mechanics, capital constraints, and FCFS (First-Come-First-Served) position allocation

The strategy has been validated across 7+ years of historical data, achieving **56.8% win rate**, **1.49 profit factor**, and **positive P&L skew** with statistical significance (binomial p < 0.001).

---

## Trading Strategy Mechanics

### Entry Criteria
- **Streak Detection**: Identifies consecutive green candles with ≥14% low-to-high rally
- **Starting Low**: Minimum of breakout candle's low or previous candle's low (ensures accurate retracement calculation)
- **Dynamic Peak Tracking**: Monitors post-streak price action for 6 months, updating peak as new highs form
- **Entry Trigger**: Price touches 0.786 Fibonacci retracement level (±0.75% tolerance band)

### Risk Management
- **Stop Loss**: Placed at starting low minus 0.2% buffer (1.0 retracement level)
- **Target**: 1.5× risk distance above entry (asymmetric 1:1.5 risk-reward)
- **Time Exit**: Forced exit after 60 trading days (~2 months) if neither target nor stop loss hit
- **Position Sizing**: Fixed ₹500,000 notional per trade for P&L calculations

### Futures Simulation (Capital Constraints)
- **Starting Capital**: ₹600,000
- **Margin Requirement**: ₹150,000 per futures lot
- **Max Concurrent Positions**: Dynamic based on available capital (cash ÷ margin per lot)
- **Allocation Logic**: FCFS—setups compete chronologically; if margin unavailable, setup is skipped
- **Exit Handling**: Margin released upon trade exit, enabling new positions

---

## Key Features

### 1. **Comprehensive Universe Coverage**
- Scans **217 F&O-eligible NSE stocks** plus Nifty 50/Bank Nifty indices
- Handles delisted stocks, data gaps, and symbol transformations automatically
- Fetches 7+ years of daily OHLC data via `yfinance` API

### 2. **Sophisticated Signal Logic**
- **Latest-Upmove-Only Rule**: When a new parabolic rally forms, it supersedes older streaks for the same stock (prevents stale signals)
- **Dynamic Retracement Calculation**: Recalculates 0.786 level daily as peak updates during 6-month monitoring window
- **Multi-Price Touch Detection**: Entry triggered if Low, High, Open, or Close touches retracement band

### 3. **Realistic Backtesting Engine**
- **Event-Driven Architecture**: Processes entry/exit events chronologically with proper margin locking/release
- **Same-Day Entry/Exit Handling**: Correctly accounts for intraday setups where entry and exit occur on same date
- **Transaction Cost Modeling**: 0.15% per trade (configurable)
- **Walk-Forward Validation**: Out-of-sample win rate tracking to prevent overfitting

### 4. **Production-Grade Outputs**
- **Excel Export**: `Executed_Trades_Futures_Sim.xlsx` with 15+ columns (Entry/Exit dates, prices, P&L, days held, exit reason)
- **Verbose Logging**: Real-time progress tracking for data fetch, setup detection, and trade execution
- **Performance Metrics Dashboard**: Win rate, profit factor, expectancy, Sharpe ratio, Sortino ratio, max drawdown

---

## Validated Performance Metrics

### Backtest Results (7-Year Period, 2018-2025)
| Metric | Value |
|--------|-------|
| **Total Setups Identified** | 1,049 |
| **Executed Trades** | 154 (capital-constrained) |
| **Win Rate** | 56.8% |
| **Profit Factor** | 1.49 |
| **CAGR** | 72.2% |
| **Sortino Ratio** | 2.2 |
| **Average Holding Period** | 12.8 days |
| **Out-of-Sample Win Rate** | 55.7% (walk-forward) |
| **P&L Distribution** | Positive skew (p < 0.001) |

### Key Insights
- **Edge Persistence**: Out-of-sample validation confirms strategy generalizes beyond training data
- **Capital Efficiency**: Despite 1,049 theoretical setups, only 154 executed due to margin constraints—demonstrates realistic position sizing
- **Asymmetric Returns**: Profit factor 1.49 indicates average winner is 49% larger than average loser
- **Statistical Significance**: Binomial test confirms win rate > 50% is not due to chance (p < 0.001)

---

## Technical Architecture

### Core Components

#### 1. **Data Pipeline** (`fetch_stock_data_for_period`)
- Fetches daily OHLC data via `yfinance.Ticker.history()`
- Handles timezone normalization and missing data
- Implements error handling for delisted/suspended stocks

#### 2. **Streak Detection** (`analyze_green_candles_period`)
- Identifies consecutive green candles (Close > Open)
- Calculates low-to-high % change for each streak
- Filters for ≥14% threshold
- Handles open-ended streaks (rally continuing through monitoring window)

#### 3. **Fibonacci Calculator** (`calculate_fib_and_find_entry`)
- Computes starting low (min of breakout low and prior candle low)
- Tracks dynamic peak during 6-month monitoring window
- Calculates 0.786 retracement level with ±0.75% tolerance band
- Returns first valid touch with entry price, stop loss, and target

#### 4. **Trade Simulator** (`execute_trade`)
- Simulates forward price action up to 60 days from entry
- Checks daily High against target and Low against stop loss
- Returns exit date, exit price, P&L, and exit reason
- Handles time-based exit at Close if neither level hit

#### 5. **Event-Driven Engine** (main loop)
- Builds chronological event stream (entry/exit pairs for each setup)
- Sorts events by date (exits processed before entries on same day)
- Tracks available capital and locked margin in real-time
- Implements FCFS allocation: skips setups if no margin available
- Records executed trades with actual entry/exit dates and realized P&L

---

## Installation & Usage

### Prerequisites
```bash
pip install yfinance pandas numpy openpyxl python-dateutil
```

### Configuration
Edit top of script to customize:
```python
BACKTEST_YEARS = 7              # Historical period to analyze
START_CAPITAL = 600_000         # Starting trading capital (₹)
MARGIN_PER_LOT = 150_000        # Margin per futures lot (₹)
POSITION_SIZE = 500_000         # Notional per trade for P&L calc (₹)
MAX_HOLD_DAYS = 60              # Max holding period (trading days)
MONITOR_MONTHS = 6              # Post-streak monitoring window
VERBOSE = True                  # Enable/disable progress logs
```

### Running the Backtest
```bash
python fibonacci_backtest.py
```

### Output Files
1. **`Executed_Trades_Futures_Sim.xlsx`**: Detailed trade log with 15+ columns
   - Stock, Entry/Exit Dates, Prices, P&L, Quantity, Days Held, Exit Reason
   - Trade setup metadata (starting low, final peak, Fib levels)

2. **Console Output**: Real-time simulation log
   - Data fetch progress (200+ stocks)
   - Setup detection confirmations
   - Event-driven trade execution log
   - Performance metrics dashboard

---

## Strategy Rationale & Market Behavior

### Why 0.786 Retracement Works
1. **Profit-Taking Exhaustion**: After 14%+ rallies, early buyers lock profits, creating selling pressure
2. **Institutional Anchoring**: Large players use Fibonacci levels for support/resistance, creating self-fulfilling behavior
3. **Mean-Reversion Asymmetry**: Sharp rallies overshoot fundamentals, triggering rational retracement
4. **Deep Value Zone**: 0.786 represents 78.6% retracement—deep enough to filter noise, shallow enough to maintain trend structure

### Walk-Forward Validation Results
- **In-Sample (Training)**: 56.8% win rate across 700+ setups
- **Out-of-Sample (Validation)**: 55.7% win rate across 349 setups
- **Conclusion**: Edge persists in unseen data, indicating genuine market inefficiency vs. overfitting

---

## Risk Disclosures & Limitations

### Known Constraints
1. **Slippage Not Modeled**: Entry assumes exact 0.786 touch; real execution may vary ±0.5-1%
2. **Liquidity Assumptions**: All stocks treated equally; low-volume F&O stocks may have wider spreads
3. **Backtest Bias**: Survivorship bias (only stocks with 7+ years data); look-ahead bias mitigated via chronological simulation
4. **Black Swan Events**: COVID-19 crash (March 2020) may skew metrics; consider stress-testing excluded periods

### Recommended Enhancements
- [ ] Add Monte Carlo simulation for confidence intervals on CAGR/Sharpe
- [ ] Implement Kelly Criterion for dynamic position sizing
- [ ] Integrate bid-ask spread modeling from NSE order book data
- [ ] Add volatility filters (exclude setups during high VIX regimes)
- [ ] Build real-time alert system via Telegram/Discord API

---

## Technologies Used

### Core Stack
- **Python 3.8+**: Main programming language
- **yfinance**: Yahoo Finance API wrapper for historical OHLC data
- **pandas**: Time-series manipulation and DataFrame operations
- **numpy**: Numerical computations for retracement calculations
- **openpyxl**: Excel file generation for trade logs

### Data Sources
- **Yahoo Finance**: Daily OHLC + volume for NSE equities (2018-2025)
- **NSE F&O List**: 217 futures-eligible stocks (manually curated)

---

## Project Structure
```
UMC-Fibonacci-Strategy/
│
├── fibonacci_backtest.py          # Main backtesting engine
├── Executed_Trades_Futures_Sim.xlsx   # Output: executed trades log
├── README.md                      # This file
└── requirements.txt               # Python dependencies
```

---

## Future Roadmap

### Phase 1: Real-Time Deployment
- [ ] Integrate with broker APIs (Zerodha Kite, Angel One SmartAPI)
- [ ] Build websocket-based live price monitoring
- [ ] Implement automated order placement with GTT (Good Till Triggered)

### Phase 2: Machine Learning Integration
- [ ] Train XGBoost classifier on setup features (volatility, volume, sector)
- [ ] Predict probability of target hit vs. stop loss hit
- [ ] Optimize entry timing within 0.75% tolerance band using ML

### Phase 3: Portfolio Optimization
- [ ] Add sector diversification constraints (max 2 positions per sector)
- [ ] Implement correlation matrix to avoid crowded trades
- [ ] Build dynamic capital allocation based on setup quality scores

---

## License

This project is released under the **MIT License**. See `LICENSE` file for details.

---

## Contributing

Contributions are welcome! Please open an issue or submit a pull request with:
- Bug fixes or performance improvements
- Enhanced documentation or usage examples
- New features (e.g., alternative retracement levels, multi-timeframe analysis)

---

## Contact

**Author**: Darshit Sarda  
**Email**: ds8286@nyu.edu  
**LinkedIn**: [linkedin.com/in/darshitsarda](https://linkedin.com/in/darshitsarda)  
**GitHub**: [github.com/DarshitSarda](https://github.com/DarshitSarda)

---

## Disclaimer

**This strategy is for educational and research purposes only.** Past performance does not guarantee future results. Trading involves substantial risk of loss. Always conduct your own due diligence and consult a financial advisor before trading with real capital. The author assumes no liability for trading losses incurred using this system.

---

## Acknowledgments

- **NSE India**: For providing F&O-eligible stock list
- **Yahoo Finance**: For historical price data API
- **NYU Tandon MFE Program**: For quantitative finance education and mentorship

---

**⭐ If you find this project useful, please star the repository!**

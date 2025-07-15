# Aave Credit Scoring Model

## 🚀 Goal
Assign credit scores (0–1000) to DeFi wallets based on their interaction behavior with the Aave V2 protocol using ML.

## 📈 Features Engineered
- Transaction frequency & type counts (deposit, borrow, repay, etc.)
- Total/avg amounts
- USD value estimations
- Repay-to-borrow ratios
- Liquidation rates
- Asset and protocol diversity

## 🧠 Model Used
- **Random Forest Regressor**
- R²: 0.925
- Top features: `repay_to_borrow_ratio`, `total_usd_value`, `borrow_count`

## 🧪 Scoring Strategy
Scores are scaled between 0–1000. Higher = responsible wallet. Lower = risky or bot-like.

## 📂 Output Files
- `wallet_scores.csv`
- `feature_importance.csv`
- `comprehensive_analysis.png`

## 📊 Behavior Summary
- Excellent wallets: 57%
- Risky wallets (liquidated): 2.9%
- Low repay ratio: 21.6%


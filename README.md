# WALLET-RISK-SCORINGS
# Problem Statement
Given a list of 100 wallet addresses, the task is to analyze their on-chain transaction behavior with the Compound V2 protocol and develop a risk scoring model that assigns a score between 0 and 1000 to each wallet. This score should reflect how safe or risky the wallet is, based on their historical lending activity.

# Deliverables
wallet_scores.csv: CSV with final risk scores.

wallet_scoring.ipynb: Jupyter Notebook with end-to-end code.

# Project Architecture
text
Copy
Edit
wallet-risk-scoring/
├── data/
│   └── wallets_list.csv  ← Input wallets (100 Ethereum addresses)
├── notebook/
│   └── wallet_scoring.ipynb  ← Codebase (Jupyter)
├── output/
│   └── wallet_scores.csv  ← Final scores
├── README.md  ← This file

#  Methodology

1. Data Collection
We queried The Graph’s Compound V2 Subgraph using GraphQL to collect transaction history per wallet.

The following interactions were retrieved:

deposits

borrows

repays

withdrawals

liquidates

Each wallet’s data is fetched and parsed using Python’s requests library with retry logic and delays to avoid rate-limiting.

2. Feature Engineering
Based on the transaction history, the following raw and derived features were computed:

* Raw Features
Feature	Description
num_deposits	Count of deposit transactions
num_borrows	Count of borrow transactions
num_repays	Count of repayment transactions
num_withdrawals	Count of withdrawals
num_liquidations	Count of liquidations
total_deposited	Cumulative value deposited
total_borrowed	Cumulative value borrowed
total_repaid	Total value repaid
total_withdrawn	Total value withdrawn

* Derived Metrics
Feature	Logic
repayment_ratio	total_repaid / total_borrowed (1.0 if no borrow)
net_position	total_deposited - total_borrowed
supply_borrow_ratio	total_deposited / (total_borrowed + 1) to avoid division by zero

3. Normalization
To ensure consistency and comparability across features, we applied MinMax Scaling using sklearn.preprocessing.MinMaxScaler. Normalized features:

repayment_ratio

supply_borrow_ratio

net_position

num_liquidations

4. Scoring Logic
We defined a composite risk score using a weighted aggregation:

python
Copy
Edit
score = (
    0.30 * repayment_ratio +
    0.25 * supply_borrow_ratio +
    0.25 * (1 - num_liquidations) +  # Penalize liquidations
    0.20 * net_position
) * 1000
repayment_ratio: Higher ratio → safer user.

supply_borrow_ratio: A conservative lender profile is better.

num_liquidations: Fewer or no liquidations are ideal.

net_position: Positive net indicates better health.

The score is scaled to the 0–1000 range, rounded to integers.

# Visualizations
To interpret and validate our scoring:

Histogram of Scores: View overall distribution.

Barplot of Top Wallets: Top 10 and full bar distribution.

Scatter Plots: Feature correlation with score.

Heatmap: Feature-feature correlation.

Trendlines: Linear regression for insight into relationships.

# Observations
Some wallets had zero or minimal activity → scored low due to insufficient data.

High scorers exhibited:

High repayment ratios

Few or no liquidations

Strong net positions

Liquidations were strongly anti-correlated with score.

# Scalability & Improvements
Could support real-time risk scoring if integrated with on-chain indexers like Dune, Covalent, or custom node providers.

Incorporate time-series trends or token-specific behavior (DAI vs USDC vs ETH).

Use ML models (e.g., clustering or classification) for more sophisticated modeling.

# Requirements
Python 3.9+

# Packages:

pandas, numpy, requests, matplotlib, seaborn, sklearn


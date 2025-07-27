# WALLET-RISK-SCORINGS
# Problem Statement
Given a list of 100 wallet addresses, the task is to analyze their on-chain transaction behavior with the Compound V2 protocol and develop a risk scoring model that assigns a score between 0 and 1000 to each wallet. This score should reflect how safe or risky the wallet is, based on their historical lending activity.

# Deliverables
wallet_scores.csv: CSV with final risk scores.

wallet_scoring.ipynb: Jupyter Notebook with end-to-end code.

# Project Architecture
┌────────────────────────┐
│  Wallet Address List   │
│    (100 Wallets)       │
└────────────┬───────────┘
             │
             ▼
┌────────────────────────┐
│  Data Collector Layer  │
│ (GraphQL API via       │
│  TheGraph - CompoundV2)│
└────────────┬───────────┘
             │
             ▼
┌────────────────────────┐
│  Transaction Parser     │
│ (Deposits, Borrows,     │
│  Repays, Withdrawals,   │
│  Liquidations)          │
└────────────┬───────────┘
             │
             ▼
┌────────────────────────────┐
│ Feature Engineering Module │
│ - Raw Tx Counts            │
│ - Total Amounts            │
│ - Derived Ratios           │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│  Normalization & Scoring   │
│  (MinMaxScaler + Formula)  │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│  Wallet Scores CSV Output  │
│  [wallet_id, score]        │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│   Visualization Layer      │
│ - Histograms               │
│ - Barplots                 │
│ - Correlation Heatmap      │
└────────────────────────────┘

# Processing Flow (Detailed)

                +-------------------------------+
                |     Wallets Input List        |
                +---------------+---------------+
                                |
                                v
          +----------------------------+       
          |  GraphQL Query (The Graph) |  <-- Compound V2 Subgraph
          +-------------+--------------+
                        |
                        v
            +------------------------+
            | Extract Transactions   |
            | - Deposits             |
            | - Borrows              |
            | - Repays               |
            | - Withdrawals          |
            | - Liquidations         |
            +-----------+------------+
                        |
                        v
              +---------------------+
              | Feature Engineering |
              | - Ratios (e.g.,     |
              |   repayment_ratio)  |
              | - Aggregates        |
              +----------+----------+
                         |
                         v
           +-----------------------------+
           | MinMaxScaler Normalization  |
           +----------+------------------+
                      |
                      v
       +-------------------------------+
       | Weighted Scoring (0–1000)     |
       | - Composite risk formula      |
       +---------------+---------------+
                       |
                       v
           +-------------------------+
           | wallet_scores.csv       |
           | [wallet_id, score]      |
           +-------------------------+
                       |
                       v
       +-------------------------------------+
       | Visualizations & Insights           |
       | - Score histogram                   |
       | - Top wallets barplot               |
       | - Feature-score scatterplots        |
       | - Correlation heatmap               |
       +-------------------------------------+

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

# Uses of Wallet Risk Scoring System
The wallet risk scoring system has multiple practical applications in decentralized finance (DeFi), blockchain analytics, and fraud prevention:

 1. Credit Risk Assessment
Lenders and protocols like Compound, Aave, or MakerDAO can use risk scores to automatically assess the credibility of borrowers before approving loans.

Helps avoid defaults and ensures healthier liquidity management.

2. DeFi Insurance Platforms
Insurance DAOs can use wallet scores to adjust premiums based on historical behavior (e.g., penalize frequent liquidations).

 3. Protocol Governance
Governance token holders can use scores to vote on blacklisting high-risk wallets, or to weight voting based on creditworthiness.

 4. Risk Analytics Tools
Analytics dashboards (e.g., Dune, Nansen, DeBank) can integrate scores to offer a user-friendly risk indicator for wallets.

 5. DeFi Portfolio Management
Robo-advisors or asset managers can filter or diversify DeFi exposure based on the wallet risk scores of liquidity providers or pool participants.

# Future Work and Improvements
This is just a foundational model. It can be significantly extended and improved with the following ideas:

 1. Real-Time Risk Monitoring
Move from static historical scoring to real-time, on-chain wallet scoring using event listeners and streaming data (e.g., The Graph Live Queries).

 2. Incorporate More Features
Include token balances, collateral-to-debt ratios, token price impact, and yield farming behaviors for richer profiling.

 3. Time-Decayed Behavior Weighting
Recent actions may be more predictive than older ones. Implement time-weighted scoring (e.g., exponential decay on older transactions).

 4. Cross-Protocol Scoring
Aggregate data not just from Compound, but also from Aave, MakerDAO, Curve, Uniswap lending, etc. → Cross-platform risk intelligence.

 5. Machine Learning Integration
Use supervised learning models like Random Forest or XGBoost to predict risk scores from engineered features instead of linear formulas.

 6. Frontend Visualization Dashboard
Build a full web app using Streamlit, Dash, or React to allow live filtering, trend visualizations, and downloadable reports for wallet risk scores.

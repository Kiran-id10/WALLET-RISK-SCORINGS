# Introduction
In decentralized finance (DeFi), assessing wallet creditworthiness is crucial for mitigating risks in lending protocols like Compound V2 and V3. This project aims to compute a risk score (0–1000) for each wallet, based on historical interaction behavior on-chain, especially with borrow, repay, and liquidation patterns.

 # Data Collection Method
Source: The Graph API for the Compound V2 protocol (subgraph: graphprotocol/compound-v2).

Wallets: 100 wallet addresses were provided via PDF/Spreadsheet.

Query: Each wallet was queried for:

deposits

borrows

repays

withdrawals

liquidates

Each transaction type included:

amount

blockTimestamp (for potential time-based analysis)

 # Feature Engineering
From the raw transaction lists, we computed both simple and derived features:

Feature	Description
num_deposits, num_borrows, etc.	Count of each interaction type
total_deposited, total_borrowed, etc.	Sum of amounts
repayment_ratio	Total repaid / total borrowed (1.0 if nothing borrowed)
net_position	Deposits - Borrows
supply_borrow_ratio	Deposits / (Borrows + 1) to avoid div-by-zero
num_liquidations	Count of liquidation events (negative indicator)

These features offer insight into the wallet's borrowing discipline, collateral handling, and default risk.

 # Scoring Methodology
1. Normalization:
Used MinMaxScaler on selected features:

repayment_ratio, net_position, supply_borrow_ratio, num_liquidations

2. Weighting and Scoring Formula:
python
Copy
Edit
score = (
    0.30 * repayment_ratio +
    0.25 * supply_borrow_ratio +
    0.25 * (1 - num_liquidations) +
    0.20 * net_position
) * 1000
Repayment Ratio (30%): Key signal of reliability.

Supply-Borrow Ratio (25%): Over-collateralization tendency.

Liquidations (25%): Penalized. Fewer = better.

Net Position (20%): Shows long-term risk-taking or conservative behavior.

 5. Visual Analysis
 Score Distribution
Histogram showed a slight right-skew, indicating most wallets have moderate risk scores, with few exceptionally safe or risky profiles.

 Top Wallets
Bar chart of top 10 wallets revealed dominant players with near-perfect repayment and high supply ratios.

 Correlation Plots
Strong positive trend: repayment_ratio vs score

Slight nonlinear: net_position vs score

Inverse trend: num_liquidations vs score

 Heatmap
Most features correlated positively with score except num_liquidations, validating its inverse impact.

# Architecture Diagram
On-Chain Risk Scoring Pipeline

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/a0c36b1d-6df4-4102-8b28-f97d81b6af81" />

[ Wallet Addresses ] 
        │
        ▼
[GraphQL Query via The Graph → Compound V2 Subgraph]
        │
        ▼
[Raw Transaction Data]
        │
        ▼
[Feature Engineering (Python)]
        │
        ▼
[MinMax Scaling → Weighted Scoring]
        │
        ├──> [Score CSV]
        ├──> [Visualizations (Matplotlib/Seaborn)]
        └──> [Optional Streamlit Dashboard]

        The image is a flowchart diagram representing the Wallet Risk Scoring System Architecture. Here's a breakdown of each step in the diagram and its purpose:

 1. Input: Wallet Address List
Description: A list of 100 wallet addresses provided via PDF or spreadsheet (e.g., Google Sheets).

Purpose: These are the on-chain Ethereum wallet IDs to be analyzed for risk scoring.

 2. Data Collection (GraphQL API Call to Compound V2/V3 Subgraph)
Description: Each wallet is queried via The Graph’s Compound V2 subgraph using GraphQL.

Data Pulled:

deposits, borrows, repays, withdrawals, liquidates

Purpose: Collect raw on-chain transaction data for risk evaluation.

 3. Transaction Parsing & Preprocessing
Description: The data fetched from each wallet is parsed into structured transaction categories.

Purpose: Normalize data format and convert JSON responses into a processable format.

 4. Feature Engineering
Raw Metrics:

Number and total of deposits, borrows, repays, withdrawals, liquidations

Derived Features:

repayment_ratio, net_position, supply_borrow_ratio

Purpose: Create numerical indicators representing user behavior and financial responsibility.

 5. Feature Normalization
Technique: MinMaxScaler used to scale values between 0 and 1

Columns Normalized:

Repayment ratio, net position, supply-borrow ratio, number of liquidations

Purpose: Standardize all features for fair comparison across wallets.

6. Risk Scoring Model
Composite Formula:

Weighted combination of engineered features

Scores scaled from 0 to 1000

Purpose: Assign a single risk score indicating the wallet’s credit behavior.

 7. Data Output:
CSV File: wallet_scores.csv

Contents: wallet_id, score

Purpose: Final output used for reporting and dashboard visualizations.

 8. Visualization Layer (Optional Dashboard)
Tools: Streamlit, Matplotlib, Seaborn

Visuals:

Score distribution, bar plots, feature correlation, heatmap

Purpose: Help explain how scores are distributed and what features influence risk.

#  Distribution of Wallet Risk Scores
<img width="1145" height="660" alt="image" src="https://github.com/user-attachments/assets/be3f794e-4c9a-4605-b61c-519bd492709b" />

The distribution of wallet risk scores (ranging from 0 to 1000) gives insights into how risky or safe each wallet is based on its historical behavior with the Compound V2 lending protocol.

Here’s a breakdown of what we observe and interpret from the histogram:

# Shape of the Distribution
The distribution is right-skewed or positively skewed, meaning:

Most wallets have moderate to low risk scores (e.g., between 300 to 700).

Fewer wallets fall into the very high score range (above 800) — indicating excellent behavior.

A small number of wallets are below 300, potentially exhibiting poor repayment patterns or frequent liquidations.

# Key Observations
Range	Behavior Characteristics	Wallet Count (approx.)
800–1000	Very low-risk wallets: high repayments, low borrows, no liquidations	Few
600–800	Low-risk wallets: reliable repayments, good net positions	Moderate
400–600	Moderate risk: mixed behavior, some borrow or liquidation events	Many
200–400	High risk: high borrow with low repayments, some liquidations	Moderate
< 200	Very high-risk: very poor repayment or heavily liquidated	Few

# Why This Distribution Makes Sense
Most users engage safely with DeFi protocols but not all are highly optimal.

Only a few wallets are exceptional (top 10–15%) with consistent deposits and repayments.

Liquidations and missed repayments significantly reduce scores.

# Visual Analysis Tips
If you used the seaborn histogram from the notebook:


sns.histplot(df["score"], bins=20, kde=True, color="skyblue", edgecolor="black")
Histogram Bars show the count of wallets in score intervals (bins).

KDE (Kernel Density Estimate) curve overlays a smoothed view of the data distribution.

You may also observe long tails toward high scores.

# Summary:
Most wallets fall in the mid-risk category.

Few wallets score extremely well or extremely poorly.

Repayment behavior, liquidations, and net deposit/borrow balance play a major role in shaping this score distribution.
# All Wallets by Risk Score
<img width="436" height="827" alt="image" src="https://github.com/user-attachments/assets/f52400cc-2dd3-47be-b60c-1b7965f9a808" />

This chart plots each of the 100 wallets on the Y-axis and their corresponding risk score on the X-axis using a horizontal bar graph.

# Key Observations from the Plot
Feature	Description
Wallet IDs (Y-axis)	Each wallet is represented by its address (usually truncated for readability). These are sorted in descending order of their scores.
Risk Score (X-axis)	A value between 0 and 1000, assigned based on repayment behavior, liquidations, and other activity on Compound V2.
Color Scheme	A gradient color palette like "viridis" shows score intensity (darker = higher risk score).
Bar Length	The longer the bar, the better the wallet’s behavior (closer to 1000). Shorter bars suggest riskier wallets.

# Insights
The top-scoring wallets (above 850) are few but show exceptional DeFi behavior: consistent deposits, repayments, and no liquidations.

The middle cluster (400–700) includes most wallets — showing balanced or moderate usage, possibly with some debt but not defaulting.

A handful of low-score wallets (below 300) indicate high risk, possibly due to:

Large borrow amounts

Low repayments

Frequent liquidations or withdrawals

# Why This View Matters
This bar chart is helpful when:

Identifying wallets to flag for risk mitigation or audits.

Comparing performance across all users.

Making credit-limit decisions or whitelisting wallets for protocol access.

 # Repayment Ratio vs Risk Score – Scatter Plot Explanations


 <img width="1120" height="733" alt="image" src="https://github.com/user-attachments/assets/3d66ef96-3579-4fe1-be42-feb6f8578e07" />

This scatter plot visualizes the relationship between a wallet’s repayment behavior and its assigned risk score.

# What’s Plotted
Axis	Description
X-axis	Repayment Ratio = total_repaid / total_borrowed (capped at 1.0 if no borrowing)
Y-axis	Risk Score (0–1000), derived from multiple features including repayment, net position, etc.

Each point on the chart represents one wallet.

# Key Observations from the Plot
Positive Trend

A positive correlation is observed.

Wallets with higher repayment ratios generally receive higher risk scores.

Low Repayment Ratio = Low Score

Wallets with repayment ratio close to 0 often have low scores (100–400) — indicating risky behavior or default tendencies.

High Repayment Ratio = High Score

Wallets with repayment ratio above 0.8 are usually safe borrowers.

These get scores 700 and above.

Flat Zone at Repayment Ratio = 1.0

Many wallets show perfect or capped repayment (1.0), suggesting:

They repaid all they borrowed.

Or never borrowed (score based on other factors).

Trend Line

A regression line (dark red) is included to highlight the overall correlation.

# Interpretation
Repayment Ratio	Behavior	Risk Implication
~0	No or poor repayment	 High risk
0.5 – 0.8	Partial repayment	 Moderate risk
>0.8 – 1.0	Fully repaid / healthy borrower	 Low risk

# Why This Plot Matters
Repayment ratio is one of the strongest predictors of on-chain creditworthiness.

This plot helps visualize why certain wallets scored higher based on actual repayment patterns.

Useful for auditing, model justification, and feature validation.

< img width="1106" height="646" alt="image" src="https://github.com/user-attachments/assets/cf8c1f8e-77a1-40eb-a67b-51f0c1975d24" />

<img width="951" height="648" alt="image" src="https://github.com/user-attachments/assets/14742caa-c29b-4508-bff3-d69721291b01" />


 # Use Cases
DeFi Lending Risk Management: Lenders can screen wallets before providing credit.

Protocol Credit Ratings: Automate eligibility scoring for users.

Web3 Insurance: Score-based premium models.

Fraud Detection: Identify wallets with aggressive liquidation or bad borrowing behavior.

# Future Work
Time-Series Modeling: Use transaction timestamps to study behavior over time.

Clustering: Group wallets into risk tiers.

Integrate Aave or Compound V3: Use cross-protocol behavior.

Include Token Volatility: Account for market risk alongside behavioral risk.

Use PCA or UMAP: Dimensionality reduction for more compact scoring.

# Wallet Risk Scoring Engine for Compound Protocol

## 📖 Table of Contents
- [1. Project Overview](#1-project-overview)
- [2. Methodology](#2-methodology)
  - [2.1. Data Collection](#21-data-collection)
  - [2.2. Feature Engineering & Rationale](#22-feature-engineering--rationale)
  - [2.3. Risk Scoring Model](#23-risk-scoring-model)
- [3. Technology Stack](#3-technology-stack)
- [4. Repository Structure](#4-repository-structure)
- [5. How to Run the Analysis](#5-how-to-run-the-analysis)
- [6. Summary of Results](#6-summary-of-results)
- [7. Conclusion](#7-conclusion)

---

## 1. Project Overview

This project presents a wallet risk scoring system designed to assess the risk profiles of Ethereum wallet addresses based on their historical transaction data on the **Compound V2 & V3 lending protocols**. 

The primary objective is to create a quantifiable, data-driven model that assigns each wallet a risk score between 0 and 1000, where a lower score indicates lower risk and a higher score indicates higher risk. This model analyzes on-chain behaviors to distinguish between safe, reliable users and those who exhibit high-risk leveraging or poor debt management.

The analysis was performed on a list of 100 wallet addresses provided for the assignment.

---

## 2. Methodology

Our approach is divided into three core stages: collecting the necessary on-chain data, engineering meaningful features that reflect financial behavior, and developing a weighted model to calculate the final risk score.

### 2.1. Data Collection

To ensure the analysis was focused strictly on the Compound protocol as required, we sourced data by tracking specific on-chain events associated with user lending and borrowing activities.

- **Data Source:** The Graph Protocol (Compound v2 Subgraph) and direct Ethereum node queries (via `web3.py`).
- **Protocol Events Tracked:**
  - `Mint`: Supplying assets to the protocol (collateral).
  - `Borrow`: Taking out a loan against collateral.
  - `RepayBorrow`: Making a repayment on an existing loan.
  - `LiquidateBorrow`: A third-party repaying a user's underwater loan and seizing their collateral. This is a critical risk indicator.
- **Data Points Collected:** For each transaction, we gathered details such as the asset involved, the amount, the USD equivalent at the time of the transaction, and the block timestamp.

### 2.2. Feature Engineering & Rationale

Raw transaction data was processed to engineer a set of features that serve as proxies for a wallet's financial health and risk appetite.

| Feature Name | Description & Rationale | Risk Implication |
| :--- | :--- | :--- |
| **Liquidation Events** | A count of how many times the wallet has been partially or fully liquidated. **This is the most significant indicator of high-risk behavior**, as it demonstrates a failure to manage debt effectively. | **Very High** |
| **Loan-to-Value (LTV) Ratio** | The ratio of the wallet's total borrowed value in USD to its total supplied collateral value in USD. A higher LTV means the user is more leveraged and closer to a liquidation threshold. | **High** |
| **Health Factor** | A representation of the safety of a user's position, calculated based on their collateral and debt. A factor below 1.0 is subject to liquidation. We analyze the wallet's average and current health factor. | **High** |
| **Borrow-to-Supply Ratio** | The ratio of total borrow events to total supply events. A user who borrows frequently relative to their supply actions may be using the protocol more aggressively. | **Medium** |
| **Collateral & Debt Diversity** | The number of unique assets the wallet has supplied as collateral and the number of unique assets it has borrowed. Over-reliance on a single, volatile asset for collateral is riskier than a diversified portfolio. | **Medium** |
| **Account Age & Activity** | The time since the wallet's first interaction with the Compound protocol and the total number of transactions. Older, more established accounts with a consistent history are generally less risky. | **Low** |
| **Repayment History** | A ratio of total repayment value to total borrow value. A history of successfully repaying large loans is a strong positive signal of reliability. | **Low** |

### 2.3. Risk Scoring Model

The final risk score is calculated using a weighted scoring model, which ensures that the most critical risk factors have the greatest impact on the outcome.

**Step 1: Normalization**
Each feature was normalized to a common scale (0 to 1) using the Min-Max normalization formula. This prevents features with large value ranges (like total borrowed USD) from disproportionately influencing the score over binary features (like liquidation events).

X_norm = (X - X_min) / (X_max - X_min)


**Step 2: Weighted Sum**
We assigned a weight to each normalized feature based on its importance in determining risk. Features indicating negative behavior were given negative weights, while positive indicators received positive weights.

| Feature | Sample Weight |
| :--- | :---: |
| Liquidation Events | -40 |
| High LTV Ratio | -25 |
| Low Health Factor | -20 |
| Repayment History | +10 |
| Account Age | +5 |

The final raw score was computed as the weighted sum of all normalized features.

**Step 3: Final Score Scaling**
The resulting raw scores were scaled to the required **0 to 1000** range to produce the final risk score for each wallet.

---

## 3. Technology Stack

- **Language:** Python 3
- **Core Libraries:**
  - `pandas`: For data manipulation and analysis.
  - `web3.py`: For interacting with the Ethereum blockchain.
  - `requests`: For querying The Graph API.
  - `gspread`: For connecting to the Google Sheets data source.
- **Environment:** Google Colaboratory (Jupyter Notebook)

---

## 4. Repository Structure


.
├── Zeru2.ipynb     # The main Jupyter Notebook with all the code for the analysis.
├── wallet_risk_scores.csv        # The final output file with wallet addresses and their risk scores.
└── README.md                     # This documentation file.


---

## 5. How to Run the Analysis

1.  **Clone the repository:**
    ```bash
    git clone <https://github.com/predator-911/aave-credit-scoring/task2>
    ```
2.  **Open the Notebook:** Upload and open `Zeru2.ipynb` in Google Colaboratory or a local Jupyter environment.
3.  **Install Dependencies:** The first cell in the notebook contains the necessary `pip` commands to install all required libraries.
4.  **Run All Cells:** Execute the cells sequentially to perform the full analysis, from data fetching to exporting the final CSV.

---

## 6. Summary of Results

The analysis of the 103 wallet addresses yielded the following key insights:

- **Total Wallets Analyzed:** 103
- **Average Risk Score:** 402.7
- **Score Range:** 332 (Lowest Risk) - 448 (Highest Risk)

#### Top 5 Highest-Risk Wallets Identified:
| Wallet Address | Risk Score |
| :--- | :---: |
| `0x93f0891bf71d8abed78e0de0885bd26355bb8b1d` | 448 |
| `0x4814be124d7fe3b240eb46061f7ddfab468fe122` | 445 |
| `0x9e6ec4e98793970a1307262ba68d37594e58cd78` | 445 |
| `0x4db0a72edb5ea6c55df929f76e7d5bb14e389860` | 436 |
| `0x27f72a000d8e9f324583f3a3491ea66998275b28` | 435 |

---

## 7. Conclusion

This project successfully demonstrates a systematic approach to on-chain risk assessment. By focusing on protocol-specific interactions and engineering features that capture nuanced financial behavior, we were able to build a robust model that effectively differentiates wallet risk profiles. The methodology is transparent, justified, and scalable for analyzing larger sets of addresses.

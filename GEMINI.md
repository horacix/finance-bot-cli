# Finance-bot

A Python-based tool for automated portfolio monitoring and rebalancing, integrated with Monarch Money and AWS.

## Project Overview

Finance-bot is designed to automate the lifecycle of investment portfolio management. It connects to the **Monarch Money API** to retrieve real-time data on holdings and account balances, compares the current state against user-defined **target allocations**, and generates actionable recommendations for rebalancing or investing new cash.

### Core Technologies
- **Language:** Python 3
- **API Integration:** GraphQL (Monarch Money)
- **Data Stores:** AWS DynamoDB (for tracking state)
- **Notifications:** AWS SNS (for alerts)
- **Configuration:** YAML (for portfolios, symbols, and settings)
- **Deployment:** Docker & GitHub Actions (CI/CD)

### Architecture
- `main.py`: The entry point and core logic engine. It handles authentication, data fetching, allocation calculation, and recommendation generation.
- `config.yml`: Global thresholds and preferred ticker symbols for each asset class.
- `accounts.yml`: Defines logical portfolios (e.g., "ira", "short-term") and their component Monarch account IDs and target allocations.
- `symbols.yml`: A mapping of ticker symbols to asset classes (e.g., `VTI` -> `stock`, `BND` -> `bond`).

## Building and Running

### Prerequisites
- Python 3.x
- Monarch Money credentials
- AWS account with SNS and DynamoDB access
- MFA TOTP secret for Monarch (stored in `MONARCH_TOKEN` env var)

### Installation
```bash
pip install -r requirements.txt
```

### Environment Variables
The following environment variables must be set:
- `MONARCH_USERNAME`: Your Monarch Money login email.
- `MONARCH_PASSWORD`: Your Monarch Money password.
- `MONARCH_TOKEN`: The TOTP secret key for Monarch MFA.
- `AWS_TOPIC_ARN`: The ARN of the SNS topic for notifications.
- `AWS_ACCESS_KEY_ID`: AWS access key.
- `AWS_SECRET_ACCESS_KEY`: AWS secret key.
- `AWS_DEFAULT_REGION`: AWS region.

### Execution
Run the bot with:
```bash
python main.py
```

**Optional Arguments:**
- `--local`: Run in local mode (prints recommendations to terminal, skips SNS and DynamoDB updates).
- `--debug`: Enable verbose logging and save API responses to the `./out/` directory.
- `--account <name>`: Process only a specific portfolio defined in `accounts.yml` (e.g., `--account ira`).
- `--rebalance`: Force a rebalance calculation, ignoring the configured threshold.

### Docker
To build and run via Docker:
```bash
docker build -t finbot .
docker run --env-file .env finbot
```

## Development Conventions

- **Configuration-First:** All asset mappings and target allocations should be managed through `symbols.yml` and `accounts.yml` rather than hardcoded in `main.py`.
- **Threshold-Based Rebalancing:** The bot uses a percentage-based threshold (defined in `config.yml`) to determine if an asset class has drifted enough to trigger a rebalance alert.
- **MFA Support:** The bot uses `oathtool` to generate TOTP codes dynamically, allowing it to bypass manual MFA prompts during execution.
- **Notification Logic:**
    - **Rebalance:** Triggered when an asset class drifts beyond the threshold.
    - **Investment:** Triggered when unmapped ("none") cash in an investment account exceeds `min_investment_balance`.
    - **Sweep:** Triggered when the main checking account balance is outside the `low`/`high` bounds.
    - **Vesting:** Triggered when non-zero balances are detected in specific vesting account IDs.

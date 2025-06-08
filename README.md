# Decentralized Stablecoin Protocol (DSC)

A fully decentralized, overcollateralized stablecoin protocol inspired by DAI, implemented in Solidity. This protocol allows users to deposit crypto collateral (WETH, WBTC), mint a stablecoin (DSC), and ensures system solvency through robust oracles and liquidation mechanisms.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Contracts](#contracts)
- [Project Structure](#project-structure)
- [Deployment](#deployment)
- [Testing](#testing)
- [Technical Details](#technical-details)
- [Security](#security)
- [Authors](#authors)
- [License](#license)

---

## Overview

This protocol enables users to:
- Deposit supported collateral (WETH, WBTC)
- Mint DSC stablecoins against their collateral
- Burn DSC to unlock collateral
- Be liquidated if their health factor falls below the threshold

The protocol uses Chainlink price feeds (or mocks for local testing) to ensure accurate and up-to-date collateral valuations. All critical operations are covered by unit and invariant tests.

---

## Architecture

- **DSCEngine**: Core contract managing collateral deposits, DSC minting/burning, and liquidations.
- **DecentralizedStableCoin**: ERC20 token contract for DSC, with mint/burn restricted to DSCEngine.
- **OracleLib**: Library to ensure price feed data is fresh and not stale.
- **Mocks**: For testing, including price feed and DSC mocks.

---

## Contracts

### src/DecentralizedStableCoin.sol

- ERC20 token representing DSC.
- Inherits from `ERC20Burnable` and `Ownable`.
- Only DSCEngine can mint/burn DSC.
- Used as the protocol's stablecoin.

### src/DSCEngine.sol

- Main protocol logic.
- Handles:
  - Collateral deposits/withdrawals (WETH, WBTC)
  - Minting and burning DSC
  - Liquidation of undercollateralized positions
  - Health factor calculations
- Uses Chainlink price feeds for collateral valuation.
- Implements reentrancy protection.

### src/libraries/OracleLib.sol

- Checks if Chainlink price data is stale (older than 3 hours).
- Reverts if stale data is detected to prevent protocol exploits.

---

## Project Structure

```
src/
  ├── DecentralizedStableCoin.sol
  ├── DSCEngine.sol
  └── libraries/
        └── OracleLib.sol

script/
  ├── DeployDSC.s.sol
  └── HelperConfig.s.sol

test/
  ├── unit/
  │     └── DSCEngineTest.t.sol
  ├── fuzz/
  │     ├── Invariants.t.sol
  │     ├── Handler.t.sol
  │     └── OpenInvariantsTest.t.sol
  └── mocks/
        ├── MockV3Aggregator.sol
        └── DSCMock.sol
```

---

## Deployment

Deployment is automated using Foundry scripts.

### script/HelperConfig.s.sol

- Provides network-specific configuration (price feeds, collateral addresses).
- Supports both local (Anvil) and testnet (Sepolia) deployments.
- Deploys mock contracts for local testing.

### script/DeployDSC.s.sol

- Deploys `DecentralizedStableCoin`, `DSCEngine`, and configures them.
- Sets up supported collateral and price feeds.
- Transfers ownership of DSC to DSCEngine.

#### Example Deployment Command

```bash
forge script script/DeployDSC.s.sol --fork-url <YOUR_RPC_URL> --broadcast
```

---

## Testing

### Unit Tests

- **test/unit/DSCEngineTest.t.sol**: Covers all core protocol logic:
  - Collateral deposit and withdrawal
  - Minting and burning DSC
  - Liquidation scenarios
  - Error handling (zero amounts, unauthorized tokens, etc.)

### Fuzz & Invariant Tests

- **test/fuzz/Invariants.t.sol**: Ensures protocol invariants (e.g., total collateral value always exceeds DSC supply).
- **test/fuzz/Handler.t.sol**: Simulates random user actions for robustness.
- **test/fuzz/OpenInvariantsTest.t.sol**: (Commented) Additional invariant tests.

### Mocks

- **test/mocks/MockV3Aggregator.sol**: Simulates Chainlink price feeds.
- **test/mocks/DSCMock.sol**: Simulates DSC contract for negative test cases.

#### Run All Tests

```bash
forge test
```

---

## Technical Details

- **Collateral Types**: WETH, WBTC (ERC20)
- **Liquidation Threshold**: 200% (liquidation occurs if collateral value < 2x DSC minted)
- **Liquidation Bonus**: 10% of collateral
- **Oracle Timeout**: 3 hours (stale data protection)
- **Access Control**: Only DSCEngine can mint/burn DSC
- **Reentrancy Protection**: All state-changing functions are guarded

---

## Security

- **Oracle Freshness**: Protocol halts if price data is stale.
- **ReentrancyGuard**: Prevents reentrancy attacks.
- **Strict Input Validation**: All user inputs are checked for validity.
- **Comprehensive Testing**: Unit, fuzz, and invariant tests ensure protocol safety.

---

## Authors

- **fawarano** — Protocol design and implementation.

---

## License

MIT

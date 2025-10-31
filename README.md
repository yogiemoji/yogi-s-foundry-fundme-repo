# FundMe - Decentralized Crowdfunding Contract

A Solidity smart contract built with Foundry that enables decentralized crowdfunding with USD-denominated minimum contributions powered by Chainlink Price Feeds.

## Overview

FundMe is a crowdfunding smart contract that:
- Accepts ETH donations with a minimum $5 USD value requirement
- Uses Chainlink Price Feeds for real-time ETH/USD conversion
- Allows only the contract owner to withdraw collected funds
- Supports deployment across multiple networks (Mainnet, Sepolia, Local)

## Features

- **USD-Denominated Minimums**: Ensures contributions meet a $5 minimum regardless of ETH price fluctuations
- **Multi-Network Support**: Automatically configures for Ethereum Mainnet, Sepolia testnet, or local Anvil
- **Gas-Optimized Withdrawals**: Two withdrawal implementations for gas comparison
- **Comprehensive Testing**: Full unit and integration test coverage
- **Automated Interactions**: Scripts for funding and withdrawing from deployed contracts

## Prerequisites

- [Foundry](https://book.getfoundry.sh/getting-started/installation) installed
- Basic understanding of Solidity and smart contracts

## Installation

1. Clone the repository:
```bash
git clone <your-repo-url>
cd foundry-fund-me
```

2. Install dependencies:
```bash
forge install
```

3. Create a `.env` file (optional, for testnet/mainnet deployment):
```bash
SEPOLIA_RPC_URL=your_sepolia_rpc_url
MAINNET_RPC_URL=your_mainnet_rpc_url
PRIVATE_KEY=your_private_key
ETHERSCAN_API_KEY=your_etherscan_api_key
```

## Usage

### Build

Compile the smart contracts:
```bash
forge build
```

### Test

Run all tests:
```bash
forge test
```

Run tests with detailed output:
```bash
forge test -vvv
```

Run tests on Sepolia fork:
```bash
forge test --fork-url $SEPOLIA_RPC_URL
```

Run specific test file:
```bash
forge test --match-path test/unit/FundMeTest.t.sol
```

### Deploy

Deploy to local Anvil:
```bash
# Terminal 1: Start Anvil
anvil

# Terminal 2: Deploy
forge script script/DeployFundMe.s.sol --rpc-url http://localhost:8545 --private-key <anvil_private_key> --broadcast
```

Deploy to Sepolia testnet:
```bash
forge script script/DeployFundMe.s.sol --rpc-url $SEPOLIA_RPC_URL --private-key $PRIVATE_KEY --broadcast --verify --etherscan-api-key $ETHERSCAN_API_KEY
```

### Interact with Deployed Contract

Fund a deployed contract:
```bash
forge script script/Interactions.s.sol:FundFundMe --rpc-url <rpc_url> --private-key <private_key> --broadcast
```

Withdraw funds (owner only):
```bash
forge script script/Interactions.s.sol:WithdrawFundMe --rpc-url <rpc_url> --private-key <private_key> --broadcast
```

### Format Code

```bash
forge fmt
```

### Gas Snapshots

Generate gas usage report:
```bash
forge snapshot
```

## Project Structure

```
foundry-fund-me/
├── src/
│   ├── FundMe.sol           # Main crowdfunding contract
│   └── PriceConverter.sol   # Library for ETH/USD conversions
├── script/
│   ├── DeployFundMe.s.sol   # Deployment script
│   ├── HelperConfig.s.sol   # Multi-network configuration
│   └── Interactions.s.sol   # Fund/Withdraw interaction scripts
├── test/
│   ├── unit/
│   │   └── FundMeTest.t.sol           # Unit tests
│   ├── integration/
│   │   └── FundMeTestIntegration.t.sol # Integration tests
│   └── mocks/
│       └── MockV3Aggregator.sol       # Mock Chainlink price feed
└── lib/                     # Dependencies (forge-std, chainlink, etc.)
```

## Smart Contract Architecture

### FundMe.sol
The main contract that:
- Stores funders and their contributions
- Enforces minimum USD contribution ($5)
- Provides two withdrawal methods: `withdraw()` and `cheaperWithdraw()`
- Uses Chainlink Price Feeds for ETH/USD conversion

### PriceConverter.sol
A library providing:
- `getPrice()`: Fetches current ETH price from Chainlink
- `getConversionRate()`: Converts ETH amount to USD value

### HelperConfig.sol
Manages network-specific configurations:
- Sepolia: Uses live Chainlink price feed
- Mainnet: Uses live Chainlink price feed
- Local/Anvil: Deploys mock price feed

## Key Concepts Demonstrated

- Using Chainlink Price Feeds for external data
- Library pattern for code reusability
- Multi-network deployment strategy with mocks
- Custom errors for gas efficiency
- Modifier pattern for access control
- Foundry testing best practices
- Gas optimization techniques

## Testing

The project includes comprehensive tests:
- **Unit Tests**: Test individual contract functions in isolation
- **Integration Tests**: Test full deployment and interaction workflows
- **Fork Tests**: Test against live network state

All tests pass on both local Anvil and Sepolia fork environments.

## Gas Optimization

The contract includes two withdrawal implementations:
- `withdraw()`: Standard implementation
- `cheaperWithdraw()`: Gas-optimized version (saves ~800 gas per funder)

Compare gas usage by running:
```bash
forge snapshot --diff
```

## Dependencies

- [forge-std](https://github.com/foundry-rs/forge-std): Foundry standard library
- [chainlink-brownie-contracts](https://github.com/smartcontractkit/chainlink-brownie-contracts): Chainlink smart contract interfaces
- [foundry-devops](https://github.com/Cyfrin/foundry-devops): Tools for working with deployed contracts

## Resources

- [Foundry Book](https://book.getfoundry.sh/)
- [Chainlink Price Feeds](https://docs.chain.link/data-feeds/price-feeds/addresses)
- [Solidity Documentation](https://docs.soliditylang.org/)

## License

MIT

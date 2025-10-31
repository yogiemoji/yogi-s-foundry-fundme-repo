# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Foundry-based Ethereum smart contract project implementing a crowdfunding contract (FundMe) that accepts ETH donations with a minimum USD value requirement, enforced via Chainlink price feeds.

## Core Architecture

### Main Contracts

**FundMe** (`src/FundMe.sol`): The main crowdfunding contract
- Uses Chainlink AggregatorV3Interface for ETH/USD price conversions
- Enforces minimum funding amount (MINIMUM_USD = 5e18 = $5)
- Owner-only withdrawal functionality with two implementations: `withdraw()` and `cheaperWithdraw()` (gas-optimized)
- Fallback/receive functions automatically route plain ETH transfers to `fund()`
- State variables follow naming conventions: `s_` prefix for storage, `i_` prefix for immutable

**PriceConverter** (`src/PriceConverter.sol`): Library for ETH/USD conversions
- `getPrice()`: Fetches current ETH price from Chainlink oracle
- `getConversionRate()`: Converts ETH amount to USD equivalent
- Used as a library attached to uint256 in FundMe via `using PriceConverter for uint256`

### Deployment & Configuration

**HelperConfig** (`script/HelperConfig.s.sol`): Multi-network configuration manager
- Automatically selects network configuration based on `block.chainid`
- Sepolia (chainid 11155111): Uses live Chainlink price feed (0x694AA1769357215DE4FAC081bf1f309aDC325306)
- Mainnet (chainid 1): Uses live price feed (0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419)
- Anvil/Local: Deploys MockV3Aggregator with 8 decimals and initial price of 2000e8
- Key pattern: Mock deployments happen within `vm.startBroadcast()/stopBroadcast()` while helper config creation happens outside (won't be deployed on-chain)

**DeployFundMe** (`script/DeployFundMe.s.sol`): Deployment script
- Uses HelperConfig to get appropriate price feed address for current network
- Deploys FundMe with network-specific Chainlink price feed

**Interactions** (`script/Interactions.s.sol`): Post-deployment interaction scripts
- `FundFundMe`: Funds the most recently deployed FundMe contract with 0.1 ETH
- `WithdrawFundMe`: Withdraws funds from the most recently deployed contract
- Uses `foundry-devops` library's `DevOpsTools.get_most_recent_deployment()` to locate deployed contracts

### Testing Structure

**Unit Tests** (`test/unit/FundMeTest.t.sol`)
- Tests individual FundMe contract functions in isolation
- Uses `funded` modifier to set up common test state (Alice funds contract)
- Tests cover: minimum USD check, ownership verification, funding data structures, withdrawal (single/multiple funders), gas optimization comparisons
- Uses Foundry cheatcodes: `vm.prank()`, `vm.deal()`, `hoax()`, `vm.expectRevert()`, `vm.txGasPrice()`, `gasleft()`

**Integration Tests** (`test/integration/FundMeTestIntegration.t.sol`)
- Tests full deployment + interaction script flow
- Verifies end-to-end user funding and owner withdrawal scenarios

## Common Commands

### Build & Compile
```bash
forge build
```

### Testing
```bash
# Run all tests
forge test

# Run tests with verbosity (shows logs, gas usage)
forge test -vv
forge test -vvv    # Show stack traces
forge test -vvvv   # Show setup/teardown traces

# Run specific test file
forge test --match-path test/unit/FundMeTest.t.sol

# Run specific test function
forge test --match-test testFundFailsWIthoutEnoughETH

# Run tests on forked network (e.g., Sepolia)
forge test --fork-url $SEPOLIA_RPC_URL
```

### Gas Snapshots
```bash
# Generate gas usage snapshot
forge snapshot

# Compare gas changes
forge snapshot --diff .gas-snapshot
```

### Formatting
```bash
forge fmt
```

### Deployment

```bash
# Deploy to Anvil (local)
forge script script/DeployFundMe.s.sol --rpc-url http://localhost:8545 --private-key <anvil_private_key> --broadcast

# Deploy to Sepolia testnet
forge script script/DeployFundMe.s.sol --rpc-url $SEPOLIA_RPC_URL --private-key $PRIVATE_KEY --broadcast --verify --etherscan-api-key $ETHERSCAN_API_KEY

# Deploy to mainnet (use with caution)
forge script script/DeployFundMe.s.sol --rpc-url $MAINNET_RPC_URL --private-key $PRIVATE_KEY --broadcast --verify --etherscan-api-key $ETHERSCAN_API_KEY
```

### Interactions with Deployed Contracts

```bash
# Fund a deployed contract
forge script script/Interactions.s.sol:FundFundMe --rpc-url <rpc_url> --private-key <private_key> --broadcast

# Withdraw from deployed contract (must be owner)
forge script script/Interactions.s.sol:WithdrawFundMe --rpc-url <rpc_url> --private-key <private_key> --broadcast
```

### Local Development Workflow

```bash
# Terminal 1: Start local Anvil node
anvil

# Terminal 2: Deploy to local node
forge script script/DeployFundMe.s.sol --rpc-url http://localhost:8545 --private-key <anvil_key> --broadcast

# Terminal 2: Interact with deployed contract
forge script script/Interactions.s.sol:FundFundMe --rpc-url http://localhost:8545 --private-key <anvil_key> --broadcast
```

### Cast Commands (Blockchain Interaction)

```bash
# Query contract state
cast call <contract_address> "getOwner()" --rpc-url <rpc_url>
cast call <contract_address> "MINIMUM_USD()" --rpc-url <rpc_url>

# Send transactions
cast send <contract_address> "fund()" --value 0.1ether --private-key <private_key> --rpc-url <rpc_url>

# Get contract balance
cast balance <contract_address> --rpc-url <rpc_url>

# Convert units
cast --to-wei 0.1ether
cast --from-wei 100000000000000000
```

## Key Dependencies

- **forge-std**: Foundry standard library for testing and scripting
- **chainlink-brownie-contracts**: Chainlink smart contract interfaces (AggregatorV3Interface)
- **foundry-devops**: Tools for finding most recently deployed contracts

## Remappings

The project uses import remapping defined in `foundry.toml`:
```
@chainlink/contracts/=lib/chainlink-brownie-contracts/contracts/
```

## Important Patterns & Conventions

### Storage Variable Naming
- `s_variableName`: Storage variables (e.g., `s_funders`, `s_priceFeed`)
- `i_variableName`: Immutable variables (e.g., `i_owner`)
- Constants use UPPER_SNAKE_CASE (e.g., `MINIMUM_USD`)

### Custom Errors
Use custom errors instead of require strings for gas efficiency:
```solidity
error FundMe__NotOwner();
if (msg.sender != i_owner) revert FundMe__NotOwner();
```

### Deployment Script Pattern
- Code before `vm.startBroadcast()` runs locally (not deployed)
- Code between `vm.startBroadcast()` and `vm.stopBroadcast()` is broadcast as transactions
- Use this pattern to set up configuration without deploying unnecessary contracts

### Multi-Network Support
Always use HelperConfig pattern to handle different networks:
1. Check `block.chainid` to determine network
2. Return appropriate configuration (live addresses or deploy mocks)
3. Never hardcode addresses directly in main contracts

### Testing Best Practices
- Use Arrange-Act-Assert pattern in tests
- Create modifiers for common setup (e.g., `funded` modifier)
- Use `makeAddr()` for generating test addresses
- Use `hoax()` for combined `prank + deal`
- Use `assertApproxEqAbs()` when comparing values that may differ due to gas costs

## Gas Optimization Notes

The contract includes two withdrawal implementations:
- `withdraw()`: Standard implementation with storage array access in loop
- `cheaperWithdraw()`: Optimized version that reads array length into memory before loop

When modifying withdrawal logic, maintain both versions for educational/comparison purposes.

## Chainlink Integration

This project relies on Chainlink Price Feeds. When adding new networks:
1. Find the ETH/USD price feed address at https://docs.chain.link/data-feeds/price-feeds/addresses
2. Add network configuration in HelperConfig
3. For local/test networks without price feeds, use MockV3Aggregator

## Documentation

Foundry Book: https://book.getfoundry.sh/

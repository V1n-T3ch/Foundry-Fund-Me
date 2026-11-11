# FundMe

FundMe is a Solidity smart contract project built with [Foundry](https://book.getfoundry.sh/). Users can fund the contract with ETH, provided the contribution is worth at least **$50 USD**, and the contract owner can withdraw the accumulated balance.

The contract uses a Chainlink `AggregatorV3Interface` price feed to convert ETH contributions to USD. Local deployments create a mock price feed automatically; Sepolia deployments use Chainlink's Sepolia ETH/USD feed.

## Features

- Accept ETH through `fund`, `receive`, or `fallback`.
- Enforce a minimum contribution of `50 USD`.
- Track each funder's total contribution.
- Restrict withdrawals to the deployer.
- Provide both standard and gas-optimized withdrawal implementations.
- Deploy and interact with Foundry scripts.
- Run unit and integration tests against a local mock price feed.

## Project structure

```text
src/
  Fundme.sol              # Main FundMe contract
  PriceConverter.sol      # Chainlink price conversion helpers
script/
  DeployFundme.s.sol      # Deployment script
  HelperConfig.s.sol      # Local and Sepolia price-feed configuration
  Interactions.s.sol      # Fund and withdraw scripts
test/
  unit/FundMeTest.t.sol   # Contract unit tests
  integration/            # Deployment and interaction tests
  mocks/                  # Mock Chainlink aggregator
```

## Prerequisites

- [Foundry](https://book.getfoundry.sh/getting-started/installation)
- Git
- An RPC URL and funded account for public-network deployments

Verify the installation:

```shell
forge --version
anvil --version
```

## Installation

Clone the repository and install the pinned dependencies:

```shell
git clone <repository-url>
cd fund-me
forge install
```

The project uses:

- `forge-std` for testing and scripting
- Chainlink contracts for `AggregatorV3Interface`
- `foundry-devops` to locate the latest deployment for interaction scripts

## Build, format, and test

```shell
# Compile contracts
forge build

# Run all tests
forge test

# Run tests with a detailed trace
forge test -vvvv

# Format Solidity files
forge fmt

# Generate a gas snapshot
forge snapshot
```

The test suite covers minimum contribution validation, ownership, price-feed integration, funder bookkeeping, access-controlled withdrawals, and scripted funding/withdrawal interactions.

## Run a local blockchain

Start Anvil in a separate terminal:

```shell
anvil
```

The repository's Makefile starts Anvil with deterministic accounts and one-second block time:

```shell
make anvil
```

The default local deployment command uses Anvil's first private key:

```shell
make deploy
```

Equivalent Forge command:

```shell
forge script script/DeployFundMe.s.sol:DeployFundMe --rpc-url http://127.0.0.1:8545 --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80 --broadcast
```

On a non-Sepolia chain, `HelperConfig` deploys a local `MockV3Aggregator` with 8 decimals and an initial ETH price of `$2,000`.

## Deploy to Sepolia

Create a local `.env` file. Do not commit it:

```dotenv
SEPOLIA_RPC_URL=https://sepolia.infura.io/v3/<project-id>
ETHERSCAN_API_KEY=<etherscan-api-key>
PRIVATE_KEY=<your-testnet-wallet-private-key>
```

Configure an account in your Foundry wallet, then deploy:

```shell
make deploy-sepolia ARGS="--network sepolia" ACCOUNT=<foundry-account-name>
```

The deployment script selects Chainlink's Sepolia ETH/USD feed:

```text
0x694AA1769357215DE4FAC081bf1f309aDC325306
```

Never place private keys, seed phrases, or API credentials in source control. Prefer Foundry's encrypted account management or another secure signer.

## Interact with a deployment

The interaction scripts use `foundry-devops` to find the most recent `FundMe` deployment for the active chain.

Fund the contract with `0.1 ETH`:

```shell
make fund SENDER_ADDRESS=<sender-address>
```

Withdraw the contract balance as the owner:

```shell
make withdraw SENDER_ADDRESS=<owner-address>
```

For Sepolia, include the network selector and the required account configuration:

```shell
make fund SENDER_ADDRESS=<sender-address> ARGS="--network sepolia" ACCOUNT=<foundry-account-name>
make withdraw SENDER_ADDRESS=<owner-address> ARGS="--network sepolia" ACCOUNT=<foundry-account-name>
```

## Contract interface

The `FundMe` contract exposes:

- `fund()` — accepts ETH when the contribution is worth at least `$50`.
- `withdraw()` — clears funder records and sends the balance to the owner.
- `cheaperWithdraw()` — equivalent withdrawal flow using a cached funder-array length.
- `getAddressToAmountFunded(address)` — returns a funder's contributed wei.
- `getFunder(uint256)` — returns a funder by index.
- `getOwner()` — returns the deployer.
- `getVersion()` — returns the configured Chainlink feed version.

The `receive` and `fallback` functions forward direct ETH transfers to `fund()`.

## Make targets

| Target | Description |
| --- | --- |
| `make build` | Compile contracts |
| `make test` | Run tests |
| `make format` | Format Solidity files |
| `make snapshot` | Generate gas snapshots |
| `make deploy` | Deploy locally using Anvil |
| `make deploy-sepolia ARGS="--network sepolia"` | Deploy to Sepolia |
| `make fund SENDER_ADDRESS=<address>` | Fund the latest deployment |
| `make withdraw SENDER_ADDRESS=<address>` | Withdraw the latest deployment |
| `make clean` | Remove Foundry build artifacts |

## License

The Solidity source files use the MIT license unless otherwise noted in their SPDX headers.

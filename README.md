# Synchronicity Price Adapters

The repository contains the Price Adapter contracts for the assets, which prices are correlated with different token rather than the pool's base one. The primary focus of these adapters is to provide accurate and reliable pricing information for assets in relation to a reference token whether it is on ETH-based or USD-based pools.

These price adapters could be used for a simple 2-step price conversion using two underlying oracles. One of the use-cases is to provide `WBTC / USD` price feed using Chainlink's `WBTC / BTC` and `BTC / USD` oracles. The same approach could be used for ETH-correlated assets, for example, `stETH / ETH` and `ETH / USD` feeds could be used to calculate the `stETH / USD` price.

Another application of the adapter is to provide the reliable stablecoins prices on the ETH-based pools. For example, the Aave v2 pool on Ethereum uses ETH-based oracles to calculate the collateral value, debt value and health factor of a user. This, coupled with the delay at which different price feeds update, introduces unnecessary volatility in positions that involve stablecoins used both as collateral and as debt.
Replacement of the current ETH-based oracles for stablecoins by using USD pairs instead will normalize the USD price using the ETH oracle and will reduce the volatility between stablecoins as all the stablecoin price feeds will update atomically when the ETH price changes.

This repository also contains the proposal smart contracts for using price adapters for stablecoins on the **Aave v2 Ethereum** and **Aave Arc** pool and deployment scripts for `WBTC / USD` and `wstETH / USD` adapters.

### Stablecoins

The proposed change is to deploy `CLSynchronicityPriceAdapterBaseToPeg` for all stablecoin assets and utilize these adapters in the `AaveOracle` smart contract, in order to accurately calculate the price of the asset in relation to `ETH`.
This will be achieved by querying [Chainlink Data Feeds](https://docs.chain.link/docs/ethereum-addresses/) for the pairs of `Asset/USD` and `ETH/USD`, then using the formula
$$Price(Asset / ETH) = {DataFeed(Asset / USD) \over DataFeed(ETH / USD)}$$
This will provide a more accurate representation of the value of the stablecoin assets.

### WBTC

To provide the `WBTC / USD` price feed `CLSynchronicityPriceAdapterPegToBase` contract utilizing `WBTC / BTC` and `BTC / USD` oracles is deployed.

General formula for this adapter is
$$Price(Asset / BASE) = {DataFeed(Asset / PEG) * DataFeed(Peg / BASE)}$$
and it can be re-used for any simple 2-step conversion.

### wstETH Adapter

Special price adapter for `wstETH / USD` is added as additionaly to using `stETH / ETH` and `ETH / USD` price feeds it requires an extra step to get the ration between `stETH` and `wstETH` for the price calculation.

## Implementation

### Price Adapter

[CLSynchronicityPriceAdapterBaseToPeg](/src/contracts/CLSynchronicityPriceAdapter.sol)

- Price adapter smart contract where `ChainlinkAggregator` addresses for `Asset / USD` and `ETH / USD` are set.
- Feeds must have the same decimals value.
- Using this two feeds, it calculates the price for pair `Asset / ETH`.
- Returning price is calculated with up to 18 decimals.

[CLSynchronicityPriceAdapterPegToBase](/src/contracts/CLSynchronicityPriceAdapterPegToBase.sol)

- Price adapter smart contract where `ChainlinkAggregator` addresses for `Asset / ETH` and `ETH / USD` are set.
- Using this two feeds, it calculates the price for pair `Asset / USD`.
- Returning price is calculated with up to 18 decimals.

[CLwstETHSynchronicityPriceAdapter](/src/contracts/CLwstETHSynchronicityPriceAdapter.sol)

- Price adapter smart contract which calculates `wstETH / USD` price based on `stETH / ETH` and `ETH / USD` feeds.
- Returning price is calculated with 8 decimals.

### Governance Payloads

[ProposalPayloadStablecoinsPriceAdapter](/src/contracts/ProposalPayloadStablecoinsPriceAdapter.sol)

- Proposal payload for the Aave v2 Ethereum pool.
- For all Aave v2 Ethereum stablecoin assets deploys `CLSynchronicityPriceAdapter` and sets it as an asset source by calling `setAssetSources` function on the `AaveOracle` contract.

[ArcProposalPayloadStablecoinsPriceAdapter](/src/contracts/ArcProposalPayloadStablecoinsPriceAdapter.sol)

- Proposal payload for the Aave Arc pool.
- For all Aave Arc stablecoin assets deploys `CLSynchronicityPriceAdapter` and sets it as an asset source by calling `setAssetSources` function on the `AaveOracle` contract.

### Deployment scripts

[DeployWBTCAdapter](/scripts/DeployWBTCAdapter.s.sol) is used to deploy the price adapter for `WBTC / USD`.

[DeployWstETH](/scripts/DeployWstETH.s.sol) is used to deploy adapter for `wstETH / USD`.

## Aave v2 Ethereum stablecoin assets and USD price feeds

List of affected Aave v2 Ethereum stablecoin assets and used Chainlink Data Feeds for `Asset / USD` pair.
| Asset | Asset address | Chainlink Data Feed address |
| --- | --- | --- |
| USDT | 0xdAC17F958D2ee523a2206206994597C13D831ec7 | 0x3E7d1eAB13ad0104d2750B8863b489D65364e32D |
| BUSD | 0x4Fabb145d64652a948d72533023f6E7A623C7C53 | 0x833D8Eb16D306ed1FbB5D7A2E019e106B960965A |
| DAI | 0x6B175474E89094C44Da98b954EedeAC495271d0F | 0xAed0c38402a5d19df6E4c03F4E2DceD6e29c1ee9 |
| SUSD | 0x57Ab1ec28D129707052df4dF418D58a2D46d5f51 | 0xad35Bd71b9aFE6e4bDc266B345c198eaDEf9Ad94 |
| tUSD | 0x0000000000085d4780B73119b644AE5ecd22b376 | 0xec746eCF986E2927Abd291a2A1716c940100f8Ba |
| USDC | 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 | 0x8fFfFfd4AfB6115b954Bd326cbe7B4BA576818f6 |
| GUSD | 0x056Fd409E1d7A124BD7017459dFEa2F387b6d5Cd | 0xa89f5d2365ce98B3cD68012b6f503ab1416245Fc |
| USDP | 0x8E870D67F660D95d5be530380D0eC0bd388289E1 | 0x09023c0DA49Aaf8fc3fA3ADF34C6A7016D38D5e3 |
| FRAX | 0x853d955aCEf822Db058eb8505911ED77F175b99e | 0xB9E1E3A9feFf48998E45Fa90847ed4D467E8BcfD |
| LUSD | 0x5f98805A4E8be255a32880FDeC7F6728C6568bA0 | 0x3D7aE7E594f2f2091Ad8798313450130d0Aba3a0 |

## Aave v2 ARC stablecoin assets and USD price feeds

List of affected Aave v2 Arc stablecoin assets and used Chainlink Data Feeds for `Asset / USD` pair.

| Asset | Asset address                              | Chainlink Data Feed address                |
| ----- | ------------------------------------------ | ------------------------------------------ |
| USDC  | 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 | 0x8fFfFfd4AfB6115b954Bd326cbe7B4BA576818f6 |

## Aave v3 assets

| Asset  | Asset address                              | Chainlink Data Feed addresses                                                                              |
| ------ | ------------------------------------------ | ---------------------------------------------------------------------------------------------------------- |
| WBTC   | 0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599 | WBTC/BTC: 0xfdFD9C85aD200c506Cf9e21F1FD8dd01932FBB23, BTC/USD: 0xF4030086522a5bEEa4988F8cA5B36dbC97BeE88c  |
| wstETH | 0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0 | stETH/ETH: 0x86392dC19c0b719886221c78AB11eb8Cf5c52812, ETH/USD: 0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419 |

## Security

### Foundry Tests

[CLSynchronicityPriceAdapterFormulaTest](./src/test/CLSynchronicityPriceAdapterFormulaTest.t.sol)

- Validates that formula used in price adapter is correct.
- For `TESTS_NUM` number of tests, makes mock aggregator with price of asset in `i-th` test set to $Price(ETH / USD) \over i$. Validates that price returned from the `CLSynchronicityPriceAdapter` is $1 ETHER /over i$.

[PriceChangeTest](./src/test/PriceChangeTest.t.sol)

- Validates that price difference between price feed for pair `Asset / ETH` and price from the adapter is less than `2%`.

[CLSynchronicityPriceAdapterPegToBaseTest](./src/test/CLSynchronicityPriceAdapterPegToBaseTest.t.sol)

- Validates that formula used in price adapter is correct.

[CLwstETHSynchronicityPriceAdapterTest](./src/test/CLwstETHSynchronicityPriceAdapterTest.t.sol)

- Validates that formula used in price adapter is correct.

[ProposalPayloadStablecoinsPriceAdapterTest](./src/test/ProposalPayloadStablecoinsPriceAdapterTest.t.sol)

- Validates that after proposal in Aave v2 Ethereum pool is accepted, all asset sources for stablecoin assets are changed.

[ArcProposalPayloadStablecoinsPriceAdapterTest](./src/test/ArcProposalPayloadStablecoinsPriceAdapterTest.t.sol)

- Validates that after proposal in Aave v2 Ethereum pool is accepted, all asset sources for stablecoin assets are changed.

### Audits

[SigmaP](./security/sigmap/audit-report.md)

[Certora](./security/Certora/Certora%20Review.pdf)

## Setup

### Install

To install and execute the project locally, you need:

- `npm install` : To install prettier for linting.
- `forge install` : This project is made using [Foundry](https://book.getfoundry.sh/) so to run it you will need to install it, and then install its dependencies.

### Setup environment

```sh
cp .env.example .env
```

### Build

```sh
forge build
```

### Test

```sh
forge test
```

### Copyright

2023 BGD Labs

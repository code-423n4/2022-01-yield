# Yield-Convex contest details

- $25,650 USDC main award pot
- $1,350 USDC gas optimization award pot
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2022-01-yield-contest/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts January 28, 2022 00:00 UTC
- Ends January 30, 2022 23:59 UTC

This repo will be made public before the start of the contest. (C4 delete this line when made public)

# Contest scoping

Yield v2 is a collateralized debt engine paired with a custom automated market maker.
We aim to provide our users a way to use their convex tokens as a collateral and at the same time let the accrue rewards they would have received for staking the convex token with convex finance.

## Smart Contracts

There are 3 smart contracts that are in the scope:

### ConvexStakingWrapper.sol (295 sloc)

Adapted convex wrapper contract upgraded to use solidity 0.8.6.

#### External contracts called

1. convexBooster: 0xF403C135812408BFbE8713b5A23a04b3D48AAE31
2. crv: 0xD533a949740bb3306d119CC777fa900bA034cd52
3. cvx: 0x4e3FBD56CD56c3e72c1403e103b45Db9da5B9D2B

#### Libraries

1. @yield-protocol/utils-v2

### ConvexYieldWrapper.sol (88 sloc)

A wrapper contract inheriting from ConvexStakingWrapper above with a way to calculate user balance from amount deposited in Join

#### External contracts called:

1. 3CRV: 0x6c3F90f043a72FA612cbac8115EE7e52BDe6E490
2. cvx3CRV: 0x30d9410ed1d5da1f6c8391af5338c93ab8d4035c
3. BaseRewardPool: 0x689440f2Ff927E1f24c72F1087E1FAF471eCe1c8
4. Cauldron: 0xc88191F8cb8e6D4a668B047c1C8503432c3Ca867

#### Libraries

1. @yield-protocol/vault-interfaces

### Cvx3CrvOracle.sol (70 sloc)

A simple oracle contract that provides 3CRV/ETH price feed

#### External contracts called:

1. 3CRVPool: 0xbEbc44782C7dB0a1A60Cb6fe97d0b483032FF1C7
2. DAI/ETH Chainlink: 0x773616E4d11A78F511299002da57A0a94577F1f4
3. USDC/ETH Chainlink: 0x986b5E1e1755e3C2440e960477f25201B0a8bbD4
4. USDT/ETH Chainlink: 0xEe9F2375b4bdF6387aa8265dD4FB8F16512A1d46

#### Libraries

1. @yield-protocol/utils-v2
2. @yield-protocol/vault-interfaces

## Areas of concern

The major area of concern is the math in the ConvexStakingWrapper that has been modified to not use the safe math as solidity version was upgraded. And also the ERC20 library used was changed from openzeppelin to yield-protocol/utils-v2.

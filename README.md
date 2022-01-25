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

### Implementation Decisions

To provide our users a way to use their token as a collateral and at the same time be able to claim rewards there needs to be a way which would allow the tokens to be staked on behalf of the users.
During our research we came across Abracadabra which provides their user a similar facility and thats how we came across [this contract](https://etherscan.io/address/0xd92494CB921E5C0d3A39eA88d0147bbd82E51008). We choose to move ahead with this contract as it satisfied our requirements and was being used successfully on a live project.
To comply with our existing contracts we choose to migrate [the contract](https://github.com/convex-eth/platform/blob/main/contracts/contracts/wrappers/ConvexStakingWrapper.sol) from 0.6.12 to 0.8.6. This lead to removal of safemath. We also choose to move ahead with using our own [ERC20 library](https://www.npmjs.com/package/@yield-protocol/utils-v2) and not using openzeppelin to make it lighter.

## Smart Contracts

There are 3 smart contracts that are in the scope:

### ConvexStakingWrapper.sol (295 sloc)

A wrapper contract which wraps convex token and stakes the convex token on user's behalf & allow them to claim rewards. This is an adapted [convex wrapper contract](https://github.com/convex-eth/platform/blob/main/contracts/contracts/wrappers/ConvexStakingWrapper.sol) upgraded to use solidity 0.8.6.

#### External contracts called

1. [convexBooster](https://etherscan.io/address/0xF403C135812408BFbE8713b5A23a04b3D48AAE31)
2. [crv](https://etherscan.io/address/0xD533a949740bb3306d119CC777fa900bA034cd52)
3. [cvx](https://etherscan.io/address/0x4e3FBD56CD56c3e72c1403e103b45Db9da5B9D2B)

#### Libraries

1. [@yield-protocol/utils-v2](https://www.npmjs.com/package/@yield-protocol/utils-v2)

### ConvexYieldWrapper.sol (88 sloc)

A wrapper contract inheriting from ConvexStakingWrapper above with a way to calculate user balance from amount deposited in Join.

#### External contracts called

1. [3CRV](https://etherscan.io/address/0x6c3F90f043a72FA612cbac8115EE7e52BDe6E490)
2. [cvx3CRV](https://etherscan.io/address/0x30d9410ed1d5da1f6c8391af5338c93ab8d4035c)
3. [BaseRewardPool](https://etherscan.io/address/0x689440f2Ff927E1f24c72F1087E1FAF471eCe1c8)
4. [Cauldron](https://etherscan.io/address/0xc88191F8cb8e6D4a668B047c1C8503432c3Ca867)

#### Libraries

1. [@yield-protocol/vault-interfaces](https://www.npmjs.com/package/@yield-protocol/vault-interfaces)

### Cvx3CrvOracle.sol (70 sloc)

A simple oracle contract that provides 3CRV/ETH price feed

#### External contracts called

1. [3CRVPool](https://etherscan.io/address/0xbEbc44782C7dB0a1A60Cb6fe97d0b483032FF1C7)
2. [DAI/ETH Chainlink](https://etherscan.io/address/0x773616E4d11A78F511299002da57A0a94577F1f4)
3. [USDC/ETH Chainlink](https://etherscan.io/address/0x986b5E1e1755e3C2440e960477f25201B0a8bbD4)
4. [USDT/ETH Chainlink](https://etherscan.io/address/0xEe9F2375b4bdF6387aa8265dD4FB8F16512A1d46)

#### Libraries

1. [@yield-protocol/utils-v2](https://www.npmjs.com/package/@yield-protocol/utils-v2)
2. [@yield-protocol/vault-interfaces](https://www.npmjs.com/package/@yield-protocol/vault-interfaces)

### How does it work with Yield protocol?

For a token to be used as a collateral, yield protocol requires them to be trasferred to a dedicated join and then perform borrowing/depositing action.
Since, we are wrapping the token we first the steps that are followed changes. So, rather than transferring them directly to the Join & pouring it goes as follows:

1. User approves the ladle to spend a specified amount of convex token
2. User initiates a batch call which does the following:
   1. Transfer the convex token from user to the wrapper contract
   2. Wrap the token and transfer the wrapped token to the specified join
   3. Perform the pouring action

Similar the process of repaying the debt also changes.

1. User transfers the fyTokens
2. User initiates a batch call which does the following:
   1. Call `user_checkpoint` on wrapper to checkpoint the user's balance
   2. Initiate pouring action to repay the debt
   3. Unwrap the wrapped token and transfer them to the user

## Design choice

### Permissionless wrap/unwrap

To keep things simple for the user we went ahead with a permissionless wrapping & unwrapping. This however, makes the function exploitable. Here is an example of how it could be exploited:

1. User A transfers their convex token to wrapper contract
2. User B immediately calls the wrap function passing their address for both to & from as a result they would end up receiving the wrapped convex token of user A which they can unwrap by calling unwrap function.

Despite the above exploit we are still moving ahead with the design as we perform the wrapping and unwrapping only in batch calls which ensures that during a transaction the above exploit cannot happen.

### Setting user vaults to 0 in removeVault

Removing the vaultId from the array would be an expensive affair and there is not a lot of gas savings due to its removal in the getDepositedBalance function. Hence, we decided to move ahead with not removing the element from the array.

# Areas of concern

The major area of concern is the math in the ConvexStakingWrapper that has been modified to not use the safe math as solidity version was upgraded. And also the ERC20 library used was changed from openzeppelin to yield-protocol/utils-v2.

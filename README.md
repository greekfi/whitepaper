# Fully Collateralized DeFi Options [WIP]
## Technical White Paper v1.0

### Abstract
We present a decentralized options protocol built on EVM-compatible blockchains that enables the creation, trading, and settlement of fully collateralized options contracts, calls and puts. The protocol supports any ERC20 token or native currency as the underlying asset, with settlement in ERC20 tokens also, including stablecoins such as USDC. Our design prioritizes security, capital efficiency, and composability/modularity while eliminating counterparty risk through full collateralization.

### 1. Introduction
#### 1.1 Background
Traditional options markets require trusted intermediaries and expose participants to counterparty risk. Decentralized finance (DeFi) enables the creation of trustless, transparent options markets where smart contracts enforce the terms and hold collateral.

Most options platforms and centralized exchanges which offer options are not fully collateralized. They are not secure and expose users to counterparty risk. The ability to move positions between platforms is limited and the ability to earn yield on collateral by selling covered calls is virtually non-existent. We leave the exploration of current options platforms and centralized exchanges to the appendix.

#### 1.2 Motivation

By introducing a smart contract into the options system, we have to break down the process into steps that differ from traditional options. We have to account for the fact that the smart contract will hold the collateral and that the process of exercising and settling options will be done through the smart contract.

This departure from tradition requires a new way of thinking about options and the role of the smart contract. 
Most options experts assume margin accounts and clearing counterparty to handle all the mechanics of options: collateral, exercise, settlement, etc. In a way, the onchain system can be interpreted more easily than the traditional systems that make many assumptions.

#### 1.3 Call Options

A call option is a contract that gives the buyer the right to buy a certain amount of an underlying asset (ASSET) prior to an expiration date (EXP) at a specified price (STRIKE) of a purchasing currency (CONSD). The seller of the call option is obligated to sell the underlying asset if the buyer decides to exercise the option priot to the expiration date. 

| Variable | Description |
|----------|-------------|
| ASSET    | The underlying asset that the option is written on. |
| CONSD   | The currency that the option is priced in. |
| STRIKE   | The price of the underlying asset that the option gives the buyer the right to buy. |
| EXP      | The date the option expires. |


Traditional Example: Alice owns 2 ETH and wants to write options on her position. She writes a call option on her position with a strike price of 3000 USD and an expiration date of 30 days from now. Bob buys the option from her. Bob now has the right to buy 2 ETH from Alice at the price of 3000 USD per ETH prior to the expiration date.


#### 1.4  Call Options on Chain
Example: Alice owns 2 wETH and wants to write options on her position. She decides to write a call option with a strike price of 3000 USD and an expiration date of 30 days from now. 

Alice deposits 2 wETH into the Option Swap Contract (OSC) as collateral. The OSC mints and transfers 2 Long Call Option Tokens (LONG) and 2 Short Call Option Tokens (SHORT) to Alice that represents her position. 

Alice lists the LONG for sale on a market (Uniswap etc.) and Bob buys the option from Alice. Bob has the choice to either hold the LONG or sell it on the market if the price moves in his favor. 

From Bob's perspective, there are three possible outcomes:

1. Bob allows the LONG to expire.
2. Bob sells the LONG on the market before expiration.
3. Before expiration, Bob interacts with the LONG smart contract to exercise the option by depositing USDC and receiving wETH.

From Alice's perspective, there are two possible outcomes:

1. After expiration, 
    1. Alice can claim the underlying asset (wETH) from the OSC, or if all the collateral has been exercised...
    2. Alice can claim the purchase currency (USDC) from OSC.
2. Before expiration, 
    1. Alice can buy back the LONG from the market and withdraw her collateral through redemption by holding the LONG and the SHORTs
    2. Alice can sell the LONG on the market.

The OSC allows anyone to bring any ERC20 token as collateral and write options on it. This is a powerful feature that allows for a wide range of options to be written on any ERC20 asset.


![Option Swap Contract Diagram](diagram1.png)

#### 1.5 Put Options On Chain
This process is similar to the call option system, the main difference is that the Put Option Writer needs to collateralize the purchase currency (CONSD) instead of the underlying asset (ASSET). This currency is used to purchase the underlying asset at the strike price from the Put Option Buyer.

Let's dive into an example. Alice is willing to buy 2 wETH at the price of 2000 USDC per wETH. She collateralizes 4000 USDC into the OSC and gets 2 Long Put Option Tokens (LONG) and 2 Short Put Option Tokens (SHORT) that represent her position and the collateral she put up.

Alice lists the LONG for sale on a market (Uniswap etc.) and Bob buys the option from Alice. Bob has the choice to either hold the LONG or sell it on the market if the price moves down in his favor.

From Bob's perspective, there are three possible outcomes:

1. Bob allows the LONG to expire.
2. Bob sells the LONG on the market before expiration.
3. Before expiration, Bob interacts with the LONG smart contract to exercise the option by depositing wETH and receiving USDC.

<!-- ![Put Option Swap Contract Diagram](diagram2.png) -->

#### 1.6 Symmetry in Calls and Puts
Both of the on-chain Call and Put strategies simply involve two steps: 1. collateralizing an asset and 2. exercising to swap a currency consideration for the collateral. In the Calls example, wETH is used as collateral and the currency is USDC.  In the Puts example, USDC is used as collateral and the currency is wET. 

Because the collateral and consideration are swapped, mathematically, the strike price is inverted (i.e. 2000 USDC per ETH instead of 0.0005 ETH per USDC). Since they are nearly symmetric, a simple flag is added to the option contract to indicate if it is a call or a put to display the appropriate strike price in traditional terms.

### 2. Technical

#### 2.1 Option Swap Contract
The option swap contract (OSC) is the gateway to the options system. 
It provides the interface to users to collateralize assets in turn creating option tokens and collateral tokens.
The OSC provides the following key functions:

| Function | Description |
|----------|-------------|
| Mint New Option and Collateral Tokens | Allows users to create new Options Tokens (OP) and Collateral Tokens (COL) by specifying parameters such as strike price, expiration date, and underlying asset, and exercise currency. This generates a new pair of Smart Contracts for the respective quintuple (STRIKE, EXP, ASSET, CONSD, isPut). |
| collaterize or mint(address asset, uint256 amount) | Allows users to collateralize an asset and receive OP and COL tokens in return. |

Once these contracts are created, the OSC does not need to be involved in the option lifecycle. The OP and COL contracts are self-contained and can be traded and exercised on secondary markets. The OSC is simply a factory that creates the contracts. The mint function is simply a convenience function to mint the tokens and collateralize the asset in one step if the OP/COL pair doesn't exist.

#### 2.1a Standard ERC20 Functions (for Convenience):

| Function | Description |
|----------|-------------|
| transfer(address to, uint256 amount) | Transfers a specified amount of tokens to the given address |
| transferFrom(address from, address to, uint256 amount) | Transfers tokens from one address to another, given approval |
| approve(address spender, uint256 amount) | Approves a spender to spend a certain amount of tokens |
| balanceOf(address account) | Returns the token balance of a given address |
| totalSupply() | Returns the total supply of tokens |
| allowance(address owner, address spender) | Returns the remaining allowance for a spender |
| name() | Returns the name of the token |
| symbol() | Returns the symbol of the token |
| decimals() | Returns the number of decimals for the token |


#### 2.2 Long Option Smart Contracts (LONG)

The Long Option Smart Contract (LONG) represents the ERC20 Token representing the Long Option. It is a tradable token that represents the option contract. The LONG contract allows the owner to exercise the option at any time before expiration. The buyer can exercise the option by depositing the exercise currency (CONSD) and receiving the underlying asset (ASSET).

The LONG contract owns the SHORT contract, allowing proper interaction with the collateral, mainly through exercise along with redemption/burning if the user owns the SHORT token. 

In addition to the ERC20 functions, the LONG contract includes the following functions:

| Function | Description |
|----------|-------------|
| constructor(string name, string symbol, address collateralAddress, address considerationAddress, uint256 expirationDate, uint256 strikeNum, uint256 strikeDen, bool isPut) | Creates both the long and short option tokens with specified parameters including name, symbol, asset addresses, expiration, strike price (as fraction num/den), and option type. |
| mint(address to, uint256 amount) | Mints both long and short option tokens to the specified address. Requires sufficient collateral. |
| exercise(address contractHolder, uint256 amount) | Allows exercising the option, transferring collateral to holder and consideration to the vault. Can only be called before expiration. |
| redeem(address contractHolder, uint256 amount) | Allows redeeming both long and short tokens together before expiration to recover collateral. |
| isExpired() | Checks if the option has expired |
| optionType() | Returns the type of the option ("CALL" or "PUT") |
| getCollateralBalance() | Returns the contract's balance of collateral tokens |
| getConsiderationBalance() | Returns the contract's balance of consideration tokens |

These functions provide the core functionality for the OP token, including standard ERC20 operations and option-specific actions like exercising and redeeming collateral.

#### 2.3 Short Option Smart Contracts (SHORT)

The Collateral Smart Contract (COL) represents the ERC20 Token for the Collateral. It is a tradable token that represents the collateral deposited by the user. The COL contract allows the owner to redeem the collateral at any time after the option expires.

In addition to the ERC20 functions, the COL contract includes the following functions:

| Function | Description |
|----------|-------------|
| constructor(string name, string symbol, address collateralAddress, address considerationAddress, uint256 expirationDate, uint256 strikeNum, uint256 strikeDen, bool isPut) | Creates the short option token with specified parameters. |
| mint(address to, uint256 amount) | Only callable by long option contract. Mints short tokens and transfers collateral to vault. |
| redeem(address to, uint256 amount) | Allows redeeming short tokens after expiration for either collateral or consideration based on exercise status. |
| redeemPair(address to, uint256 amount) | Only callable by long option contract. Allows redeeming paired tokens before expiration. |
| exercise(address contractHolder, uint256 amount) | Handles the exercise process by transferring collateral to exerciser and collecting consideration. |
| isBalanced() | Checks if the vault maintains proper balance of assets |
| vaultCollateral() | Returns the vault's collateral token balance |
| vaultConsideration() | Returns the vault's consideration token balance |


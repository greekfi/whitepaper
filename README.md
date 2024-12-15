# Greek.fi - Decentralized Options
## Overview
Greek.fi is the only decentralized options protocol that allows for the creation of options with full collateralization. We provide simplified extensions that allow for more options strategies, such as covered call vaults, along with taking loans on the collateral, and more.


## Introduction
We have built a decentralized American style options protocol that allows for the creation of options with full collateralization.

Typically, options have been centralized, with providers like Deribit being the most popular. These providers use margin to allow a variety of options to be created and traded. Additionally, most derivative strategies involve perpetual futures, which require oracles for pricing. 

Our protocol is designed to be a decentralized alternative to these providers. No oracles are required, and no margin is required. Additionally, any locked collateral used to create one side of the trade can be used as collateral for loans, as the position is represented by an ERC20 token.

## Protocol Overview

Our protocol is simple. 
Any ERC20 token can be used as collateral for options. This includes WBTC, WETH, stETH, SBTC, AAVE, UNI, and more.
The user chooses the strike price, expiration, and option type (call or put) when creating the option.
A user mints two ERC20 tokens when creating an option:
- LONG: Represents the right to exercise the option
- SHORT: Represents the obligation and right to the collateral

Use any EVM compatible chain to mint the options.

### What can you do with the tokens?
#### LONG token
The long token represents the position of the option buyer, they are entitled to purchase the underlying collateral asset when the option is exercised.
- The LONG token can allow you to exercise the option at any time before expiration.
- The LONG token can be traded on DEXs like Uniswap, CoW, and RFQs like 0x, Bebop.

#### SHORT token
The short token represents the position of the option writer, they are obligated to swap the underlying collateral asset for the consideration asset when the LONG option is exercised.
- The SHORT token can be used to redeem the underlying asset after expiration.
- The SHORT token can be used as collateral for loans (Silo).
- Together, the LONG and SHORT tokens can be redeemed for the underlying asset before expiration to recover the collateral (Neutral position).

### How is this possible?
The two tokens/contracts are linked to each other. When the LONG token is exercised, the LONG contract interacts with the SHORT contract to swap the underlying collateral asset for the consideration asset.

Similarly, when using both tokens together, the LONG and SHORT contracts interact to redeem both tokens for the collateral, pre-expiration. Post expiration, the SHORT token, alone, can be redeemed for the underlying asset.

### No Margin? No Oracles?
Because the LONG and SHORT tokens are linked to each other, the protocol is able to provide a margin-free experience. 

### Example: Minting Options
1. Connect your wallet to the protocol
2. Select option parameters:
   - Collateral Asset: WETH
   - Consideration Asset: USDC
   - Strike Price: 5000 USDC
   - Expiration Date: 30 days
   - Option Type: Call
3. Mint Approve the LONG contract to capture your WETH
4. Receive LONG and SHORT tokens

## Trading
Once options are minted, the obvious question is how to trade them. We have considered using a DEX AMM, such as Uniswap, but options could have low volume and low liquidity. This can cause slippage and nobody wants that.

The obvious solution is to use RFQs, such as 0x, Bebop, etc. This allows for the options to be traded at a price that is determined by the market. This would allow partnering with market makers to provide liquidity for the options. This would also solve pricing the options where the MMs set their own prices. This will create a new market maker ecosystem.

### What about the Short Token?
Good Question. The SHORT token represents two things: Collateral/consideration ownership and a short position in the option trade. Let's say you sold an in the money call (WETH at 3000 expiring soon). When you sold your LONG token you received a premium of 1000 USDC in addition to IV and Time Value. This is potentially priced in the SHORT token making it tradable since it has a value.

## Vaults
Vaults allow for more complex strategies using the options protocol. This simplifies the experience for users and reduces the amount of work required to implement these strategies. 

#### Covered Call Vaults
One strategy, such as covered call vaults, which allow for the creation of options with a portion of the collateral as the option premium. A simple strategy could involve users depositing WETH as collateral, and then the vault handles 
1. minting options with the collateral, 
2. selling the options, 
3. let the options expire worthless, 
4. redeem the collateral, 
5. mint more options with the collateral and repeat. 

This strategy is nearly identical to ETF strategies (ie XYLD). Covered call vaults do not exist on chain, but this protocol allows for the creation of these strategies. This same strategy can be applied for Covered Put vaults, where the vault mints the put options and sells them.

#### Margined Options Vaults
Similar to covered call vaults, margined options vaults allow for the minting of call options using collateral. Afterwards, the vault takes a loan on the collateral (AAVE, Silo, etc) and uses the proceeds to mint more options. This allows for the vault to mint more options than it would have been able to otherwise. 


## FAQ
#### In the traditional covered call world, I don't need to mint tokens or anything. Why do I need to mint tokens here?

In traditional call strategies, the custodian (E-Trade, Fidelity, etc) holds the collateral and establishes the short position. On Chain, we do not have custodians. We have smart contracts. We need to mint tokens to represent the two positions, which technically is neutral wrt to the option trade. 

#### Why do this on chain?
You don't have to. If you're looking for high frequency trading, this is not for you. If you're looking for a low-friction, low-cost way to create covered options strategies, this is for you. 

#### Ok, I'm holding X, why do this? couldn't I just get a loan on the X and buy more X? 
Again, you don't have to, but this is almost like staking. 

#### Explain this SHORT token? isn't that just a PUT?
The short token is simply a combination of two things: Your ownership of the collateral + the short position in the option trade. The SHORT token is basically looking at your brokerage account and seeing "1 wETH + (-1 call option)". If you tack on a LONG token it's like seeing "1 wETH + (-1 call option) + 1 call option" which equals "1 wETH".

#### How do you do this for PUTs?
Yes! Believe it or not, puts are simply calls the consideration and collateral are swapped. For example, in a put use USDC becomes the collateral and WETH is the consideration. What this means is that you have the option to"buy" USDC with your WETH.

 The only real difference between puts and calls is the strike price. In a call you buy 1 wETH for 4000 USDC, so in a put instead of saying "buy USDC for 0.00025 wETH/USDC" you say "sell you 1 wETH for 4000 USDC/wETH".

#### I want to buy some options, how do I do that?
Use the trade page to buy options. It uses x0 and Bebop to provide liquidity and execute trades.

#### Okay, I'm up on my options, how do I sell them?
Use the trade page to sell options, as well as buy. 

#### Can I try this on Testnets?
Yes, definitely. Choose testnets when you mint. We'll give you free wETH testnet tokens to mint with.

## Appendix
### Key Concepts
- **Call Option**: Right to buy an asset at a specified price before expiration
- **Put Option**: Right to sell an asset at a specified price before expiration 
- **Strike Price**: The price at which the option can be exercised
- **Expiration Date**: The last date the option can be exercised
- **LONG Token**: Represents the right to exercise the option
- **SHORT Token**: Represents the obligation and right to the collateral
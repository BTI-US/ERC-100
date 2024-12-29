---
title: ERC-100 Token Standard
description: Overview of the ERC-100 fungible token standard, how it works, and its comparison with ERC-20.
lang: en
---

## Introduction {#introduction}

### What is ERC-100? {#what-is-erc100}

ERC-100 is a fungible token standard similar to the ERC-20 standard. The main difference is that ERC-100 not only defines the token application interface but also extends the interaction logic with liquidity pools. It introduces a unique token price model that allows custom logic to be executed based on real-time prices during token transfers, thereby supporting dynamic interactions with liquidity pools.

### Differences from ERC-20 {#erc20-differences}

ERC-100 addresses the limitations of ERC-20 in terms of liquidity pool interactions and dynamic price adjustments, offering the following enhancements:

- **Liquidity Pool Interaction Logic**: Supports operations such as buying, selling, adding, and removing liquidity with tokens and liquidity pools.
- **Price-Aware Transfers**: Token prices dynamically update and take effect during interactions with liquidity pools.
- **Custom Logic Support**: Allows custom behaviors based on token prices (e.g., dynamic slippage control, fee adjustments).

## Prerequisites {#prerequisites}

- [Accounts](https://ethereum.org/en/developers/docs/accounts/)
- [Smart Contracts](https://ethereum.org/en/developers/docs/smart-contracts/)
- [Token Standards](https://ethereum.org/en/developers/docs/standards/tokens/)
- [ERC-20](https://ethereum.org/en/developers/docs/standards/tokens/erc-20/)

## Body {#body}

ERC-100 is a token standard for implementing token application interfaces in smart contracts. It also declares related interfaces for token interactions with liquidity pools. Smart contracts that implement the following methods and events can be considered ERC-100 compliant token contracts. Once deployed, they will be responsible for tracking tokens created on Ethereum and integrating with liquidity pools.

Contracts can have more functions than these; developers can add other features as needed. For example, they can extend support for complex liquidity pool management logic or on-chain price oracle integration.

### Methods {#methods}

ERC-100 tokens must implement the following methods:

```solidity
function name() public view returns (string)
function symbol() public view returns (string)
function decimals() public view returns (uint8)
function totalSupply() public view returns (uint256)
function balanceOf(address account) public view returns (uint256 balance)
function transfer(address recipient, uint256 amount) public returns (bool success)
function getSwapActionFromUpdate(address from, address to) public view returns (SwapAction)
function getTokenPrice() public view returns (uint256 price, uint256 token, uint256 weth)
function getNewPriceAfterBuy(uint256 amount) public view returns (uint256)
function getNewPriceAfterSell(uint256 amount) public view returns (uint256)
function getReserves() public view returns (uint256 token, uint256 weth)
```

### Events {#events}

```solidity
event PriceUpdated(address indexed pool, uint256 newPrice)
event ActionDetermined(address indexed from, address indexed to, SwapAction action)
```

### Examples {#examples}

The following example demonstrates the core functionalities of ERC-100, including price calculation and liquidity pool interactions:

```solidity
pragma solidity ^0.8.20;
abstract contract ERC100 is ERC20, IERC100 {
    IUniswapV2Router02 public immutable uniswapV2Router;
    IUniswapV2Pair public immutable uniswapV2Pair;

    constructor(string memory name_, string memory symbol_, address router_) ERC20(name_, symbol_) {
        uniswapV2Router = IUniswapV2Router02(router_);
        uniswapV2Pair = IUniswapV2Pair(IUniswapV2Factory(uniswapV2Router.factory()).createPair(address(this), uniswapV2Router.WETH()));
    }

    function getSwapActionFromUpdate(address from, address to) public view returns (SwapAction) {
        SwapAction action = SwapAction.Transfer;
        if (from == address(uniswapV2Pair)) {
            if (to == address(uniswapV2Router)) {
                action = SwapAction.RemoveLiquidity;
            } else {
                action = SwapAction.Buy;
            }
        } else if (to == address(uniswapV2Pair)) {
            action = SwapAction.Sell;
        }
        return action;
    }

    function getTokenPrice() public view returns (uint256 price, uint256 token, uint256 weth) {
        price = 0;
        (token, weth) = getReserves();
        if (token != 0 && weth != 0) {
            price = token / weth;
        }
    }

    function getNewPriceAfterBuy(uint256 amount) public view returns (uint256) {
        (uint256 price, uint256 token, uint256 eth) = getTokenPrice();
        token = token - amount;
        uint256 eth_in = (eth * amount) / token;
        eth_in = (1000 * eth_in) / 997;
        eth = eth + eth_in;
        uint256 new_price = token / eth;
        return new_price;
    }

    function getNewPriceAfterSell(uint256 amount) public view returns (uint256) {
        (uint256 price, uint256 token, uint256 eth) = getTokenPrice();
        eth = (eth * token) / (token + ((amount * 997) / 1000));
        token = token + amount;
        uint256 new_price = token / eth;
        return new_price;
    }

    function getReserves() public view returns (uint256 token, uint256 weth) {
        (uint112 reserve0, uint112 reserve1,) = uniswapV2Pair.getReserves();
        (uint _token, uint _weth) = uniswapV2Pair.token0() == address(this) ? (reserve0, reserve1) : (reserve1, reserve0);
        token = uint256(_token);
        weth = uint256(_weth);
    }
}
```

## FAQ {#faq}

### How does ERC-100 support interactions with liquidity pools? {#liquidity-pool-interaction}

ERC-100 uses the `getSwapActionFromUpdate` method to dynamically determine the type of interaction (buy, sell, add, or remove liquidity). Additionally, through price calculation functions (such as `getNewPriceAfterBuy` and `getNewPriceAfterSell`), users can predict price changes after transactions.

### Why introduce price-aware logic? {#price-awareness}

Price-aware logic allows market prices to be considered during token interactions, enabling more complex scenarios (such as preventing malicious arbitrage).

### How to extend ERC-100? {#extending-erc100}

Developers can extend the standard by inheriting the ERC-100 contract and adding new features, such as integrating on-chain price oracles or introducing custom fee mechanisms.

## Limitations {#limitations}

- **Complexity**: Compared to ERC-20, ERC-100's logic is more complex, which may lead to higher gas costs.
- **Compatibility**: Existing ERC-20 tools and platforms may need adjustments to fully support ERC-100.
- **Adoption Rate**: As a new standard, ERC-100 may take time to be widely accepted by the community.

## Further Reading {#further-reading}

- [Uniswap Whitepaper](https://app.uniswap.org/whitepaper.pdf)
- [ERC-20 Standard](https://eips.ethereum.org/EIPS/eip-20)

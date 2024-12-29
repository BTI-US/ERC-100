Usage Guide
===========

This guide explains how to implement and interact with the ERC-100 token standard.

Prerequisites
-------------

- Basic understanding of Ethereum accounts and smart contracts.
- Familiarity with the ERC-20 token standard.
- Access to an Ethereum development environment (e.g., Truffle, Hardhat) and a smart contract deployment tool.

Implementation
--------------

Implement the required functions and events in your smart contract. Below is the basic implementation for the ERC-100 token:

.. code-block:: solidity

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

Interacting with Liquidity Pools
--------------------------------

ERC-100 introduces advanced liquidity pool interactions. You can use the following methods to interact with pools:

- **getSwapActionFromUpdate**:
  Determines the swap action type (buy, sell, add/remove liquidity) based on the addresses involved.
- **getTokenPrice**:
  Fetches the current price of the token by querying the reserves of the liquidity pool.
- **getNewPriceAfterBuy**:
  Calculates the new token price after a buy operation based on the amount of tokens being bought.
- **getNewPriceAfterSell**:
  Calculates the new token price after a sell operation based on the amount of tokens being sold.
- **getReserves**:
  Retrieves the reserves of the token and WETH in the liquidity pool.

Example: Fetching the Token Price
----------------------------------

To fetch the current token price, you can call the `getTokenPrice` method:

.. code-block:: solidity

   uint256 price;
   uint256 tokenReserves;
   uint256 wethReserves;
   (price, tokenReserves, wethReserves) = erc100Instance.getTokenPrice();

This function will return the price of the token, along with the current reserves of the token and WETH in the liquidity pool.

Example: Determining the Action Type
--------------------------------------

You can determine whether an action is a buy, sell, or liquidity removal by using the `getSwapActionFromUpdate` method:

.. code-block:: solidity

   SwapAction action = erc100Instance.getSwapActionFromUpdate(fromAddress, toAddress);

Where `fromAddress` and `toAddress` are the addresses involved in the transaction.

Conclusion
----------

ERC-100 enhances the ERC-20 standard by adding dynamic interactions with liquidity pools. It provides advanced features like price awareness and custom logic for liquidity management, making it a suitable choice for decentralized finance (DeFi) applications.

API Reference
=============

.. module:: erc100
   :synopsis: ERC-100 Token Standard API documentation

Overview
--------

The ERC-100 token standard introduces advanced features for interacting with liquidity pools and price updates within decentralized finance (DeFi) ecosystems. It extends the basic ERC-20 token functionality by adding methods to track and update token prices, as well as determine liquidity actions such as buys, sells, and liquidity removals. This API reference provides detailed information on the available methods and events in the ERC-100 token contract, which can be used to implement and interact with DeFi protocols.

Methods
-------

.. function:: name()

   Returns the name of the token.

   This function retrieves the human-readable name of the token, such as "MyToken". It is commonly used in user interfaces and token metadata.

   **Returns**: 
   - `string`: The name of the token.

.. function:: symbol()

   Returns the symbol of the token.

   The symbol is a short identifier for the token, typically used for display purposes, such as "MTK" or "ETH". It is often used in token lists and exchanges to represent the token.

   **Returns**: 
   - `string`: The symbol of the token.

.. function:: decimals()

   Returns the number of decimals used by the token.

   The `decimals` function returns the number of decimal places the token supports, which defines the smallest unit of the token that can be transferred. For example, if `decimals()` returns 18, the smallest unit is 1e-18 of the token.

   **Returns**: 
   - `uint8`: The number of decimals the token uses.

.. function:: totalSupply()

   Returns the total supply of the token.

   This function returns the total amount of tokens currently in circulation. It is used to get a snapshot of the overall supply of the token.

   **Returns**: 
   - `uint256`: The total supply of the token.

.. function:: balanceOf(account)

   Returns the balance of the given account.

   This function allows querying the balance of a specific account (address). It can be used to check how many tokens an address holds.

   **Parameters**:
   - `account` (`address`): The address of the account whose balance is being queried.

   **Returns**: 
   - `uint256`: The token balance of the specified account.

.. function:: transfer(recipient, amount)

   Transfers tokens from the caller to the recipient.

   This function enables the transfer of tokens from the sender's address to the recipient's address. The transaction must be approved and executed by the sender.

   **Parameters**:
   - `recipient` (`address`): The address to which tokens are being sent.
   - `amount` (`uint256`): The number of tokens to transfer.

   **Returns**: 
   - `bool`: Returns `true` if the transfer is successful.

.. function:: getSwapActionFromUpdate(from, to)

   Determines the type of swap action based on transaction details.

   This function determines the action type (buy, sell, add/remove liquidity) for a specific transaction, depending on the addresses involved. It helps identify whether a transaction is a swap, liquidity addition, or liquidity removal.

   **Parameters**:
   - `from` (`address`): The sender address.
   - `to` (`address`): The recipient address.

   **Returns**: 
   - `SwapAction`: An enumerated value indicating the type of action:
     - `SwapAction.Buy`
     - `SwapAction.Sell`
     - `SwapAction.RemoveLiquidity`
     - `SwapAction.AddLiquidity`
     - `SwapAction.Transfer`

.. function:: getTokenPrice()

   Returns the current token price.

   This function calculates and returns the current price of the token based on the liquidity pool's reserves. The price is determined by the ratio of token reserves to WETH (or another base token) reserves.

   **Returns**: 
   - `uint256`: The current token price, expressed as an integer (with decimals depending on the token’s precision).
   - `uint256`: The amount of token in the reserves.
   - `uint256`: The amount of WETH (or base token) in the reserves.

.. function:: getNewPriceAfterBuy(amount)

   Calculates the new token price after a buy operation.

   This function estimates the price of the token after a buy transaction. It takes into account the slippage and the token amount being purchased, and it returns the adjusted token price post-purchase.

   **Parameters**:
   - `amount` (`uint256`): The amount of tokens being purchased.

   **Returns**: 
   - `uint256`: The new token price after the buy transaction, factoring in the updated token and liquidity pool reserves.

.. function:: getNewPriceAfterSell(amount)

   Calculates the new token price after a sell operation.

   Similar to the `getNewPriceAfterBuy` function, this function estimates the token price after a sell transaction, considering the liquidity pool reserves and the amount of tokens being sold.

   **Parameters**:
   - `amount` (`uint256`): The amount of tokens being sold.

   **Returns**: 
   - `uint256`: The new token price after the sell transaction.

.. function:: getReserves()

   Retrieves the token and WETH reserves.

   This function provides the current reserves of the token and WETH in the liquidity pool. It is critical for calculating the token price and understanding liquidity availability.

   **Returns**: 
   - `uint256`: The token reserve.
   - `uint256`: The WETH reserve.

Events
------

.. event:: PriceUpdated(pool, newPrice)

   Emitted when the token price is updated.

   This event is triggered whenever the price of the token is updated. It can be used to track price changes in external systems like decentralized exchanges (DEXs) or trading platforms.

   **Parameters**:
   - `pool` (`address`): The address of the liquidity pool where the price update occurred.
   - `newPrice` (`uint256`): The new price of the token after the update.

.. event:: ActionDetermined(from, to, action)

   Emitted when a swap action is determined.

   This event is fired when the type of swap action (buy, sell, liquidity add/remove) is identified. It can be used to log transaction types or trigger external processes based on swap actions.

   **Parameters**:
   - `from` (`address`): The address initiating the transaction.
   - `to` (`address`): The recipient address of the transaction.
   - `action` (`SwapAction`): The determined swap action (e.g., `SwapAction.Buy`, `SwapAction.Sell`).

FAQ
===

How does ERC-100 support interactions with liquidity pools?
-----------------------------------------------------------

ERC-100 dynamically determines interaction types (buy, sell, add, or remove liquidity) using the ``getSwapActionFromUpdate`` method. Additionally, price calculation functions (e.g., ``getNewPriceAfterBuy`` and ``getNewPriceAfterSell``) enable users to predict price changes after transactions.

Why introduce price-aware logic?
--------------------------------

Price-aware logic allows market prices to be considered during token interactions, enabling more complex scenarios (e.g., preventing malicious arbitrage).

How to extend ERC-100?
----------------------

Developers can extend the standard by inheriting the ERC-100 contract and adding new features, such as on-chain price oracle integration or custom fee mechanisms.
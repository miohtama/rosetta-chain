# Elrond Fungible Token Model

Elrond uses [ESDT token standard](https://docs.elrond.com/developers/esdt-tokens/).

Tokens are supported natively, like native EGLD. Issuing out a token does not require you to deploy a smart contract. Any ESDT token 

Mikko's token user and developer experience sanity checklist

* ✅ Native currency and tokens are not handled different in smart contracts (as opposite to ERC-20 `transfer()` vs. `payable`, [NEP-141](https://github.com/near/NEPs/discussions/146))

* ✅ Tokens can be send and received in smart contracts with a single transaction (as opposite to ERC-20 `transferFrom()`)

* ✅ [You can query all available token balances of in a wallet](https://docs.elrond.com/developers/esdt-tokens/#get-all-esdt-tokens-for-an-address) with a single call (as opposite to ERC-20 where the wallet provider needs to manage a centralised token registry of some sort)

* ✅ You can query newly issued tokens from the blockchain (as opposite to ERC-20 which lacks `Created` event)

* ❓ You can monitor for any incoming token transactions with a single WebSocket subscription

* ❌ Tokens have human-readable metadata by the token issuer, like the token website URL ([ESDT lacks any meaningful metadata (https://docs.elrond.com/developers/esdt-tokens/)])

* ❌ Tokens can attach special transfer logic, like whitelisting or blacklisting (ESDT does not have transfer hook, only [freezing](https://docs.elrond.com/developers/esdt-tokens/#freezing-and-unfreezing))
 
* ❌ Token transfers require have native token balance in the wallet (so-called multi token problem, no built-in fee markets or relayers)

## Issuing a token

TODO

## Sending tokens to an address

TODO

## Cross-shard transactions

TODO

## Open questions

Node scalability: What if create 10,000 tokens and send them to a single wallet: there is no pagination for token balance call.

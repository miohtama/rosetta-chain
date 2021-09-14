# Elrond Fungible Token Model

Elrond uses [ESDT token standard](https://docs.elrond.com/developers/esdt-tokens/).

Tokens are supported natively, like native EGLD. Issuing out a token does not require you to deploy a smart contract. Any ESDT token 

Mikko's token user and developer experience sanity checklist

* ✅ Native currency and tokens are not handled different in smart contracts (as opposite to ERC-20 `transfer()` vs. `payable`, [NEP-141](https://github.com/near/NEPs/discussions/146))

* ✅ Tokens can be send and received in smart contracts with a single transaction (as opposite to ERC-20 `transferFrom()`)

* ✅ [You can query all available token balances of in a wallet](https://docs.elrond.com/developers/esdt-tokens/#get-all-esdt-tokens-for-an-address) with a single call (as opposite to ERC-20 where the wallet provider needs to manage a centralised token registry of some sort)

* ✅ You can query newly issued tokens from the blockchain (as opposite to ERC-20 which lacks `Created` event)

* ✅ Token transfers have a memo field, so you can distinguish transactions in centralised exchange easily 

* ❓ You can monitor for any incoming token transactions with a single WebSocket subscription

* ❌ Tokens have human-readable metadata by the token issuer, like the token website URL ([ESDT lacks any meaningful metadata (https://docs.elrond.com/developers/esdt-tokens/)])

* ❌ Tokens can attach special transfer logic, like whitelisting or blacklisting (ESDT does not have transfer hook, only [freezing](https://docs.elrond.com/developers/esdt-tokens/#freezing-and-unfreezing))
 
* ❌ Token transfers require have native token balance in the wallet (so-called multi token problem, no built-in fee markets or relayers)

## Issuing a token

Here are the steps required to issue out a new token.

Install erdpy.

```shell
python -m venv venv
source venv/bin/activate
pip install erdpy
```

Create a test wallet:

```shell
erdpy wallet derive test-wallet.pem
```

[Get devnet EGLD to your wallet](https://r3d4.fr/elrond/testnet/index.php).

Start interactive IPython console.

```python
pip install ipython
ipython
```

Paste in the following snippet with `%cpaste` command:

```python
from binascii import hexlify
from erdpy.accounts import Account, Address
from erdpy.proxy import ElrondProxy
from erdpy.transactions import BunchOfTransactions
from erdpy.transactions import Transaction
from erdpy.wallet import signing

TOKEN_NAME = b"Cowdings"
TOKEN_SYMBOL = b"MOO"

DECIMALS = 18
SUPPLY = 1000 * 10**DECIMALS


def hex_string(s: str) -> str:
    assert type(s) == bytes, "Make sure everything is bytes data or utf-8 encoded"
    return hexlify(s).decode("ascii")


def hex_int(i: int) -> str:
    assert type(i) == int, "Make sure everything is bytes data or utf-8 encoded"
    return hex(i)[2:]


proxy = ElrondProxy("https://devnet-gateway.elrond.com")
sender = Account(pem_file="test-wallet.pem")
sender.sync_nonce(proxy)

tx = Transaction()
tx.nonce = sender.nonce
tx.value = str(int(0.05 * 10**18))  # 0.05 ERD, as required for issuing a token according to the documentation
tx.sender = sender.address.bech32()
# System contract address to issue out the new token as per
# https://docs.elrond.com/developers/esdt-tokens/#issuance-of-fungible-esdt-tokens
# https://docs.elrond.com/developers/esdt-tokens/#issuance-examples
tx.receiver = "erd1qqqqqqqqqqqqqqqpqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqzllls8a5w6u"
tx.gasPrice = 1000000000
tx.gasLimit = 60_000_000  # From issuance examples
tx.data = f"issue" \
          f"@{hex_string(TOKEN_NAME)}" \
          f"@{hex_string(TOKEN_SYMBOL)}" \
          f"@{hex_int(SUPPLY)}" \
          f"@{hex_int(DECIMALS)}" 

tx.chainID = "D"  # For devnet https://devnet-gateway.elrond.com/network/config
tx.version = 1  # No idea where this should come from, because the lack of documentation

tx.signature = signing.sign_transaction(tx, sender)
txid = tx.send(proxy)

print(f"Token created in the transaction https://devnet-explorer.elrond.com/transactions/{txid}")
```

## Checking out issued tokens

Now the full token supply is in the issuers address.

The token identifer has been randomly generated. You can find out the issued token identifier by loooking the transaction on an explorer. 

https://devnet-explorer.elrond.com/transactions/bd600fd5b96fdba08dee3a391039853a6d848e0a05622fad96d9489738c975e1

The initial transfer of the supply contains the token identifier in the hexadecimal format:

```
ESDTTransfer@4d4f4f2d656534373931@3635c9adc5dea00000
```

You can decode this:

```python
from binascii import unhexlify
unhexlify("4d4f4f2d656534373931")
```

```
In [3]: unhexlify("4d4f4f2d656534373931")
Out[3]: b'MOO-ee4791'
```

This this ticker (symbol + randomly generated id) you can check the token balance:

```python
from erdpy.accounts import Account, Address
from erdpy.proxy import ElrondProxy

TOKEN_ID = "MOO-ee4791"

proxy = ElrondProxy("https://devnet-gateway.elrond.com")
sender = Account(pem_file="test-wallet.pem")

# balance': '1000000000000000000000', 'properties': '', 'tokenIdentifier': 'MOO-ee4791'}
balance_data = proxy.get_esdt_balance(sender.address, TOKEN_ID)

print("Raw balance reply", balance_data)
balance_decimal = int(balance_data["balance"]) / 10**18

print(f"You have {balance_decimal:,.6f} tokens of {TOKEN_ID}")
```

## Sending tokens to an address

Let's send tokens to a friend.

I picked my friend as an address `erd1pdv0h3ddqyzlraek02y5rhmjnwwapjyhqm983kfcdfzmr6axqhdsfg4akx` randomly from the [devnet explorer](https://devnet-explorer.elrond.com).

Transfering tokens work by sending a transaction to the receiver.
The transaction `Data` field must contain a special payload that is picked by a system contract (?). [Gas limit must be set to 250,000](https://docs.elrond.com/developers/esdt-tokens/#transfers).

```python
from binascii import hexlify
from erdpy.accounts import Account, Address
from erdpy.proxy import ElrondProxy
from erdpy.transactions import BunchOfTransactions
from erdpy.transactions import Transaction
from erdpy.wallet import signing


TOKEN_ID = b"MOO-ee4791"


def hex_string(s: str) -> str:
    assert type(s) == bytes, "Make sure everything is bytes data or utf-8 encoded"
    return hexlify(s).decode("ascii")


def hex_int(i: int) -> str:
    assert type(i) == int, "Make sure you give an int"
    return hex(i)[2:]


proxy = ElrondProxy("https://devnet-gateway.elrond.com")
sender = Account(pem_file="test-wallet.pem")
sender.sync_nonce(proxy)

receiver = "erd1pdv0h3ddqyzlraek02y5rhmjnwwapjyhqm983kfcdfzmr6axqhdsfg4akx"

# 3.0 tokens = 
token_value = int(3.0 * 10**18)

tx = Transaction()
tx.nonce = sender.nonce
tx.value = "0"  # Must be zero?
tx.sender = sender.address.bech32()
# System contract address to issue out the new token as per
# https://docs.elrond.com/developers/esdt-tokens/#issuance-of-fungible-esdt-tokens
# https://docs.elrond.com/developers/esdt-tokens/#issuance-examples
tx.receiver = receiver
tx.gasPrice = 1000000000
tx.gasLimit = 500_000  # https://docs.elrond.com/developers/esdt-tokens/#transfers
tx.data = f"ESDTTransfer" \
          f"@{hex_string(TOKEN_ID)}" \
          f"@{hex_int(token_value)}"

tx.chainID = "D"  # For devnet https://devnet-gateway.elrond.com/network/config
tx.version = 1  # No idea where this should come from, because the lack of documentation

tx.signature = signing.sign_transaction(tx, sender)
txid = tx.send(proxy)

print(f"Tokens transferred in the transaction https://devnet-explorer.elrond.com/transactions/{txid}")
```

## Open questions

Node scalability: What if create 10,000 tokens and send them to a single wallet: there is no pagination for token balance call.

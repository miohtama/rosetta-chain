# Elrond Hello World smart contract

A smart contract with Hello World string and a counter.

Note this example is in C, although Elrond smart contract SDK is Rust based.

## How to create a project template

[Documentation](https://docs.elrond.com/developers/tutorials/counter/).

Install [erdpy](https://github.com/ElrondNetwork/elrond-sdk-erdpy).

```shell
python -m venv venv
source venv/bin/activate
pip install erdpy
```

(These steps have been already executed in this repo.)

Then create `mycounter` contract:

```shell
erdpy contract new --template="simple-counter" mycounter
```

## Building the contract

This creates Wasm files out from the source code:

```shell
erdpy --verbose contract build mycounter
```

## Getting testnet account and testnet EGLD balance

Create a wallet PEM file using erdpy

Create a new wallet.

```shell
erdpy wallet derive testnet-wallet.pem
```

Write down your address given by the command.

```shell
INFO:cli.wallet:Created PEM file [testnet-wallet.pem] for [erd1tw0k2hw0h5r394h5qccwyssc6zzza9anxy25zha67nqzjwqs32tqhexaww]
```

[Visit the Elrond testnet faucet](https://r3d4.fr/elrond/testnet/index.php). Note that Elrond has multiple test networks (testnet, devnet), so pick the correct one.

Request testnet EGLD to your address.

Check your wallet address balance.

```shell
erdpy account get --proxy="https://testnet-gateway.elrond.com" --address=erd1tw0k2hw0h5r394h5qccwyssc6zzza9anxy25zha67nqzjwqs32tqhexaww 
```

## Deploying the contract

Now you can deploy the contract:

```shell
erdpy --verbose contract deploy --project=mycounter --pem="testnet-wallet.pem" --gas-limit=5000000 --proxy="https://testnet-gateway.elrond.com" --outfile="counter.json" --recall-nonce --send
```

The output is like:

```
INFO:accounts:Account.sync_nonce()
INFO:accounts:Account.sync_nonce() done: 0
INFO:cli.contracts:Contract address: erd1qqqqqqqqqqqqqpgqrm8wsmymfzwx5aygwzyuvmvj2f2n2ns232tq4qkng5
INFO:transactions:Transaction.send: nonce=0
INFO:transactions:Hash: 2345254d00dd84990bb598cdc75ac6b23c35ae58f3e183a9d59dc276a1f5d103
```

Write down the contract address or save it in the shell:

```shell
export CONTRACT_ADDRESS=erd1qqqqqqqqqqqqqpgqrm8wsmymfzwx5aygwzyuvmvj2f2n2ns232tq4qkng5
```

## Testing the contract

Now you can call contract 

```shell
erdpy --verbose contract call $CONTRACT_ADDRESS --pem="testnet-wallet.pem" --gas-limit=2000000 --function="increment" --proxy="https://testnet-gateway.elrond.com" --recall-nonce --send
```

The output is like:

```
INFO:transactions:Hash: 93e7d0cc8a21658fa88fbf8d18a93beb01a7689b04102c4f0ac72f0c46a82f05
```

```json
{
    "emitted_tx": {
        "tx": {
            "nonce": 2,
            "value": "0",
            "receiver": "erd1qqqqqqqqqqqqqpgqrm8wsmymfzwx5aygwzyuvmvj2f2n2ns232tq4qkng5",
            "sender": "erd1tw0k2hw0h5r394h5qccwyssc6zzza9anxy25zha67nqzjwqs32tqhexaww",
            "gasPrice": 1000000000,
            "gasLimit": 2000000,
            "data": "aW5jcmVtZW50",
            "chainID": "T",
            "version": 1,
            "signature": "b2de7206445dc4a9c048a2fa9dbf05e0e2c318be1f55f6ecd1a0888fd2e5d6cd7659522d9d1dbd297aebb0d06a7a810f5cd311e3872dba951748d3848ed8fd05"
        },
        "hash": "93e7d0cc8a21658fa88fbf8d18a93beb01a7689b04102c4f0ac72f0c46a82f05",
        "data": "increment"
    }
}
```

Now you can read the new `counter` value from the blockchain:

```shell
erdpy contract query $CONTRACT_ADDRESS --function="get" --proxy="https://testnet-gateway.elrond.com"
```

You will get output:

```json
[
    {
        "base64": "Ag==",
        "hex": "02",
        "number": 2
    }
]
```
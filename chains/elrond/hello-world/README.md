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

## Getting testnet account and balance

1. Create a wallet on
2. Write down your address
3. Go to testnet faucat 
4. 


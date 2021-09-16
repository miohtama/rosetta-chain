# Token receiver smart contract

An example how to build a smart contract that receives tokens in Elrond.

The smart contract has three functionalities

* Receive named token

* Tell how many tokens it has received

* Send tokens back

## Bootstrapping

Create a Rust smart contract skeleton using erdpy. This is all in nested `token-receiver` folder.

```shell
python -m venv venv
source venv/bin/activate
pip install erdpy
erdpy contract new token-receiver --template adder
```

## Compiling the contract

Let's try if the default skeleton compiles

```shell
cd token-receiver
erdpy contract build
```


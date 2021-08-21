# Setting up Elrond API node

## Generating validator key

[This is required](https://github.com/ElrondNetwork/elrond-go/issues/3367).

We [generate key PEM file with erdpy](https://docs.elrond.com/sdk-and-tools/erdpy/deriving-the-wallet-pem-file/#docsNav).

```shell
python -m venv venv
source venv/bin/activate
pip install erdpy
```

Creating the key:

```
erdpy wallet derive validator.pem
```

## Running node

Run

```shell
docker-compose up elrond
```

## Documentation

[Elrond Testnet Docker](https://docs.elrond.com/validators/testnet/use-docker-testnet/)
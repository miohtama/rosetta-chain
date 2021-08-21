# Setting up Elrond API node

## Generating validator key

[Valiator key is required even though not used](https://github.com/ElrondNetwork/elrond-go/issues/3367).

Creating the validator key by [compiling necessary Elrond binaries](https://github.com/ElrondNetwork/elrond-go) and running `keygenerator`:

```sh
git clone https://github.com/ElrondNetwork/elrond-go
cd elrond-go
GO111MODULE=on go mod vendor
(cd cmd/keygenerator && go build)
```

Generate keys:

```sh
cmd/keygenerator/keygenerator --console-out
```

Then save this output in `validator.pem`.

## Running node

Docker Compose maps the local `validator.pem` for the node to start.

Run

```shell
docker-compose up elrond
```

## Testing the node

You should see the node connecting to the network:

```
    
```

## Documentation

[Elrond Testnet Docker](https://docs.elrond.com/validators/testnet/use-docker-testnet/)
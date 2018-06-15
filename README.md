# Deploy smart contract to the remote testnet via the local nodeos

## Start the local nodeos

It is needed to start a local nodeos that is connected to the remote testnet.
Please refer to this link:

`https://github.com/FeeSimple/eos-tracker/tree/master/nodeos/client_node_connect_to_testnet`

## Contract deployment steps

The local nodeos is supposed to listen at `http://localhost:8877`

1. Create wallet if not yet

`cleos wallet create`

Remember to store the output password for later unlocking.
By default, wallet daemon is listenning at `http://localhost:6666`

2. Unlock the wallet if currently locked

`cleos --wallet-url http://localhost:6666 wallet unlock`

3. Import private key

It's needed to import private key of the account used to deploy contract.
Right here, we can utilize the created account on the remote testnet because
this account is funded with enough RAM.

```
"name": "useraaaaaaaa",
"private-key": "5JtUScZK2XEp3g9gh7F8bwtPTRAkASmNrrftmx4AxDKD5K4zDnr",
"public-key": "EOS69X3383RzBZj41k73CSjUNXM5MYGpnDxyPnWUKPEtYQmTBWz4D"
```

`cleos --wallet-url http://localhost:6666 --url http://localhost:8877 wallet import 5JtUScZK2XEp3g9gh7F8bwtPTRAkASmNrrftmx4AxDKD5K4zDnr`

4. Create contract files if not available

If not available, it's possible to create contract skeleton by using the following cmd:

`eosiocpp -n ${contract_name}`

5. Compile and build contract

```
eosiocpp -o ${contract_name}.wast ${contract_name}.cpp
eosiocpp -g ${contract_name}.abi ${contract_name}.hpp
```

6. Deploy contract

Do not stay inside the contract folder as the it will never work.
Right here, we use the account `useraaaaaaaa` for deployment.

`cleos --wallet-url http://localhost:6666 --url http://localhost:8877 set contract useraaaaaaaa ${path_to_contract_folder}`

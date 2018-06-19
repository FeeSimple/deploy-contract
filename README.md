# Deploy smart contract to the remote testnet via the local nodeos

## Start the local nodeos

It is needed to start a local nodeos that is connected to the remote testnet.
Please refer to this link:

(https://github.com/FeeSimple/eos-tracker/tree/master/nodeos/client_node_connect_to_testnet)

## Contract deployment steps

The local nodeos is supposed to listen at `http://localhost:8877`

### Create wallet if not yet

`cleos wallet create`

Remember to store the output password for later unlocking.
By default, wallet daemon is listenning at `http://localhost:6666`

### Unlock the wallet if currently locked

`cleos --wallet-url http://localhost:6666 wallet unlock`

-> enter the password to unlock

### Import private key

It's needed to import private key of the account (into the local wallet) used to deploy contract.
Right here, we can utilize the created account on the remote testnet because
this account is funded with enough RAM.

```
"name": "useraaaaaaaa",
"private-key": "5JtUScZK2XEp3g9gh7F8bwtPTRAkASmNrrftmx4AxDKD5K4zDnr",
"public-key": "EOS69X3383RzBZj41k73CSjUNXM5MYGpnDxyPnWUKPEtYQmTBWz4D"
```

`cleos --wallet-url http://localhost:6666 --url http://localhost:8877 wallet import 5JtUScZK2XEp3g9gh7F8bwtPTRAkASmNrrftmx4AxDKD5K4zDnr`

### Create contract files if not available

If not available, it's possible to create contract skeleton by using the following cmd:

`eosiocpp -n ${contract_name}`

### Compile and build contract

```
eosiocpp -o ${contract_name}.wast ${contract_name}.cpp
eosiocpp -g ${contract_name}.abi ${contract_name}.cpp
```

### Deploy contract

Do not pass the abi file or wast file as arguments as the it will never work.
Right here, we use the account `useraaaaaaaa` for deployment.

**Command:**

`cleos --wallet-url http://localhost:6666 --url http://localhost:8877 set contract ${account_name} ${path_to_contract_folder}`

**Example:**

```
cleos --wallet-url http://localhost:6666 --url http://localhost:8877 set contract useraaaaaaaa hello

Reading WAST/WASM from hello/hello.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 051481bef826a097082ef4e5ba93f1467459d774990538484e90a45803245f32  1800 bytes  781 us
#         eosio <= eosio::setcode               {"account":"useraaaaaaaa","vmtype":0,"vmversion":0,"code":"0061736d01000000013b0c60027f7e006000017e6...
#         eosio <= eosio::setabi                {"account":"useraaaaaaaa","abi":"0e656f73696f3a3a6162692f312e30000102686900010475736572046e616d65010...
warning: transaction executed locally, but may not be confirmed by the network yet

```

## Contract interaction

**Command:**

`cleos --wallet-url http://localhost:6666 --url http://localhost:8877 push action ${account_name} ${contract_method} '[${argument_list}]' -p ${account_name}@active`

**Example:**

```
cleos --wallet-url http://localhost:6666 --url http://localhost:8877 push action useraaaaaaaa hi '["trung"]' -p useraaaaaaaa@active

executed transaction: 1c9d5d983de01f57a5afdede69d293fd86a9c59139cc49ca7d1de0ab6b31ee7a  104 bytes  904 us
#  useraaaaaaaa <= useraaaaaaaa::hi             {"user":"trung"}
>> Hello Hello, trung
warning: transaction executed locally, but may not be confirmed by the network yet
```

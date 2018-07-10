# Deploy smart contract to the remote testnet via the local nodeos

## Start the local nodeos

It is needed to start a local nodeos that is connected to the remote testnet.
Please refer to this link:

(https://github.com/FeeSimple/eos-tracker/tree/master/nodeos/client_node_connect_to_testnet)

## Contract deployment steps

The local nodeos is supposed to listen at `http://138.197.194.220:8877`

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

`cleos --wallet-url http://localhost:6666 --url http://138.197.194.220:8877 wallet import 5JtUScZK2XEp3g9gh7F8bwtPTRAkASmNrrftmx4AxDKD5K4zDnr`

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

`cleos --wallet-url http://localhost:6666 --url http://138.197.194.220:8877 set contract ${account_name} ${abs_path_to_contract_folder} ${abs_path_to_wast_file} ${abs_path_to_abi_file}`

**Note:**
Must use absolute path. Otherwise, the deployment cmd will be failed

**Example:**

```
cleos --wallet-url http://localhost:6666 --url http://138.197.194.220:8877 set contract useraaaaaaaa ~/Work/reference/dyver/eos-todo/contract/ ~/Work/reference/dyver/eos-todo/contract/hello.wast ~/Work/reference/dyver/eos-todo/contract/hello.abi

Reading WAST/WASM from hello/hello.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 051481bef826a097082ef4e5ba93f1467459d774990538484e90a45803245f32  1800 bytes  781 us
#         eosio <= eosio::setcode               {"account":"useraaaaaaaa","vmtype":0,"vmversion":0,"code":"0061736d01000000013b0c60027f7e006000017e6...
#         eosio <= eosio::setabi                {"account":"useraaaaaaaa","abi":"0e656f73696f3a3a6162692f312e30000102686900010475736572046e616d65010...
warning: transaction executed locally, but may not be confirmed by the network yet

```

**Error note:**

```
Error 3040005: Expired Transaction
Please increase the expiration time of your transaction!
Error Details:
expired transaction 1ef65a2816337fca606a243bf89c2f898e28467b0480f6021b395ddfe8f67dbd
```

** Root cause:**

The running local nodeos is incurring high latency.
Either wait until the latency value is decreased or restart the local nodeos

```
0|script_c | 3229491ms thread-0   producer_plugin.cpp:290       on_incoming_block    ] Received block 5bc67b62c9008b98... #97000 @ 2018-06-19T17:53:00.000 signed by producer111a [trxs: 0, lib: 96867, conf: 108, latency: 75649491 ms]
0|script_c | 3240221ms thread-0   producer_plugin.cpp:290       on_incoming_block    ] Received block 510669cfd7c66f58... #98000 @ 2018-06-19T18:01:20.000 signed by producer111d [trxs: 0, lib: 97863, conf: 0, latency: 75160221 ms]
0|script_c | 3250914ms thread-0   producer_plugin.cpp:290       on_incoming_block    ] Received block c1523a5a97110731... #99000 @ 2018-06-19T18:09:40.000 signed by producer111g [trxs: 0, lib: 98859, conf: 0, latency: 74670914 ms]
```

## Contract interaction

### Invoke contract method (via the account used in contract deployment)

**Command:**

`cleos --wallet-url http://localhost:6666 --url http://138.197.194.220:8877 push action ${account_name} ${contract_method} '[${argument_list}]' -p ${account_name}@active`

**Example:**

```
cleos --wallet-url http://localhost:6666 --url http://138.197.194.220:8877 push action useraaaaaaaa hi '["trung"]' -p useraaaaaaaa@active

executed transaction: 1c9d5d983de01f57a5afdede69d293fd86a9c59139cc49ca7d1de0ab6b31ee7a  104 bytes  904 us
#  useraaaaaaaa <= useraaaaaaaa::hi             {"user":"trung"}
>> Hello Hello, trung
warning: transaction executed locally, but may not be confirmed by the network yet
```

### Query contract code (via the account used in contract deployment)

**Command:**

`cleos --wallet-url http://localhost:6666 --url http://138.197.194.220:8877 get code ${account_name}`

**Example:**

```
cleos --wallet-url http://localhost:6666 --url http://138.197.194.220:8877 get code useraaaaaaaa

code hash: f1c645067bde280a6407a986d69b94ece3720891a403fefe3f00b940ffd1ace0
```

## Account interaction

### Get account details

**Command:**

`cleos --wallet-url http://localhost:6666 --url http://138.197.194.220:8877 get account ${account_name}`

**Example:**

```
cleos --wallet-url http://localhost:6666 --url http://138.197.194.220:8877 get account useraaaaaaaa

privileged: false
permissions:
     owner     1:    1 EOS69X3383RzBZj41k73CSjUNXM5MYGpnDxyPnWUKPEtYQmTBWz4D
        active     1:    1 EOS69X3383RzBZj41k73CSjUNXM5MYGpnDxyPnWUKPEtYQmTBWz4D
memory:
     quota:     645.7 Mb     used:     33.29 Kb   

net bandwidth: (averaged over 3 days)
     staked:  291582899.8560 SYS           (total stake delegated from account to self)
     delegated:       0.0000 SYS           (total staked delegated to account from others)
     used:              3.66 Kb   
     available:        96.12 Tb   
     limit:            96.12 Tb   

cpu bandwidth: (averaged over 3 days)
     staked:  291582899.8560 SYS           (total stake delegated from account to self)
     delegated:       0.0000 SYS           (total staked delegated to account from others)
     used:             13.65 ms   
     available:         2800 hr   
     limit:             2800 hr   

proxy:     producer111a
```

### Check account balance

**Command:**

`cleos --wallet-url http://localhost:6666 --url http://138.197.194.220:8877 get currency balance eosio.token ${account_name}`

**Example:**

```
cleos --wallet-url http://localhost:6666 --url http://138.197.194.220:8877 get currency balance eosio.token useraaaaaaaa

10.0000 SYS
```

### Create new account

**Command:**

`cleos --wallet-url http://localhost:6666 --url http://138.197.194.220:8877 system newaccount ${creator_account} ${new_account_name} ${pubkey} --stake-net "${some_amount} SYS" --stake-cpu "${some_amount} SYS" --buy-ram "${some_amount} SYS"`

**Note on new account name:**

`must be exactly 12 characters and only contains the following symbol .12345abcdefghijklmnopqrstuvwxyz`

**Example:**

```
cleos --wallet-url http://localhost:6666 --url http://138.197.194.220:8877 system newaccount useraaaaaaaa trung1234512 EOS7CJJwBTeUyNBLYF8TWZWenFTXtTp1o1RjgjnHV9wGstiPitFdT --stake-net "2 SYS" --stake-cpu "2 SYS" --buy-ram "1 SYS"

1529285ms thread-0   main.cpp:426                  create_action        ] result: {"binargs":"608c31c6187315d6204221430436f5cd10270000000000000453595300000000"} arg: {"code":"eosio","action":"buyram","args":{"payer":"useraaaaaaaa","receiver":"trung1234512","quant":"1.0000 SYS"}}
1529287ms thread-0   main.cpp:426                  create_action        ] result: {"binargs":"608c31c6187315d6204221430436f5cd204e0000000000000453595300000000204e000000000000045359530000000000"} arg: {"code":"eosio","action":"delegatebw","args":{"from":"useraaaaaaaa","receiver":"trung1234512","stake_net_quantity":"2.0000 SYS","stake_cpu_quantity":"2.0000 SYS","transfer":false}}
executed transaction: 65aae773de68c389418304e864bd692b6efe6252c00a134592eabf730902ba3d  344 bytes  8505 us
#         eosio <= eosio::newaccount            {"creator":"useraaaaaaaa","name":"trung1234512","owner":{"threshold":1,"keys":[{"key":"EOS7CJJwBTeUy...
#         eosio <= eosio::buyram                {"payer":"useraaaaaaaa","receiver":"trung1234512","quant":"1.0000 SYS"}
#   eosio.token <= eosio.token::transfer        {"from":"useraaaaaaaa","to":"eosio.ram","quantity":"0.9950 SYS","memo":"buy ram"}
#  useraaaaaaaa <= eosio.token::transfer        {"from":"useraaaaaaaa","to":"eosio.ram","quantity":"0.9950 SYS","memo":"buy ram"}
#     eosio.ram <= eosio.token::transfer        {"from":"useraaaaaaaa","to":"eosio.ram","quantity":"0.9950 SYS","memo":"buy ram"}
#   eosio.token <= eosio.token::transfer        {"from":"useraaaaaaaa","to":"eosio.ramfee","quantity":"0.0050 SYS","memo":"ram fee"}
#  useraaaaaaaa <= eosio.token::transfer        {"from":"useraaaaaaaa","to":"eosio.ramfee","quantity":"0.0050 SYS","memo":"ram fee"}
#  eosio.ramfee <= eosio.token::transfer        {"from":"useraaaaaaaa","to":"eosio.ramfee","quantity":"0.0050 SYS","memo":"ram fee"}
#         eosio <= eosio::delegatebw            {"from":"useraaaaaaaa","receiver":"trung1234512","stake_net_quantity":"2.0000 SYS","stake_cpu_quanti...
#  producer111a <= eosio::delegatebw            {"from":"useraaaaaaaa","receiver":"trung1234512","stake_net_quantity":"2.0000 SYS","stake_cpu_quanti...
#   eosio.token <= eosio.token::transfer        {"from":"useraaaaaaaa","to":"eosio.stake","quantity":"4.0000 SYS","memo":"stake bandwidth"}
#  useraaaaaaaa <= eosio.token::transfer        {"from":"useraaaaaaaa","to":"eosio.stake","quantity":"4.0000 SYS","memo":"stake bandwidth"}
#   eosio.stake <= eosio.token::transfer        {"from":"useraaaaaaaa","to":"eosio.stake","quantity":"4.0000 SYS","memo":"stake bandwidth"}
warning: transaction executed locally, but may not be confirmed by the network yet

```

**Note:**

```
Error 3010001: Invalid name
Name should be less than 13 characters and only contains the following symbol .12345abcdefghijklmnopqrstuvwxyz
Error Details:
Name not properly normalized (name: trung6789123, normalized: trung....123)
```

```
https://github.com/EOSIO/eos/issues/3480
Since you get this error message, you certainly have system contract installed.
System contract allows to create only one short (<12 characters) account name per 24 hours (one that got the highest bid).
If you want to just create an account (without doing bidding) - the name should be exactly 12 characters long.
```

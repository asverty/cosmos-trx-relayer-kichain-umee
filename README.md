# Checking

Open kid/config/config.toml and check up that TCP for the RCP server listen on laddr = "tcp://127.0.0.1:26657"

Change it if it needed and restart the service by the command:

> $ sudo systemctl restart kichaind

## Check the status:

> $ kid status 2>&1 | jq

# Install the relayer

## Download the archive:

> $ wget https://github.com/cosmos/relayer/releases/download/v0.9.3/Cosmos.Relayer_0.9.3_linux_amd64.tar.gz

## Unpackage it:

> $ tar -zxvf Cosmos.Relayer_0.9.3_linux_amd64.tar.gz

## And copy:

> $ cp "Cosmos Relayer" /usr/local/bin/rly

## Init relayer:

> $ rly config init

## Edit config.yaml:
```
global:
  api-listen-addr: :5183
  timeout: 3m
  light-cache-size: 20
chains:
- key:
  chain-id: umee-betanet-1
  rpc-addr: http://161.97.78.75:26657
  account-prefix: umee
  gas-adjustment: 1.5
  gas-prices: 0.025uumee
  trusting-period: 10m
- key:
  chain-id: kichain-t-4
  rpc-addr: http://127.0.0.1:26657
  account-prefix: tki
  gas-adjustment: 1.5
  gas-prices: 0.025utki
  trusting-period: 10m
paths: {}
```

## Umee key creating:

> $ rly keys add umee-betanet-1 umeekey

## Output:
```
{"mnemonic":"_____ ____ ____ ____ ...":"umee1.........................."}
```

## Kichain node key restore:

> $ rly keys restore kichain-t-4 kikey "mnemonic divided by space"

## Output:
```
{"mnemonic":"_____ ____ ____ ____ ...":"tki1..........................."}
```

## Adding keys to kichain:

> $ rly chains edit umee-betanet-1 key umeekey

> $ rly chains edit kichain-t-4 key kikey

## Request tokens from Umee faucet at Umee Discord server and check balance:

## Umee balance:

> $ rly query balance umee-betanet-1

## Output:

> $ rly query balance umee-betanet-1
> 100000000uumee
  
## Kichain balance:
> $ rly query balance kichain-t-4

## Output:
> $ rly query balance kichain-t-4
> 14567355utki
    
# Running relayer
  
## Run clients:
> $ rly light init umee-betanet-1 -f
> $ rly light init kichain-t-4 -f

## Output:
```
$ rly light init umee-betanet-1 -f
successfully created light client for umee-betanet-1 by trusting endpoint http://161.97.78.75:26657
$ rly light init kichain-t-4 -f
successfully created light client for kichain-t-4 by trusting endpoint http://127.0.0.1:26657
```

## Create path from UMEE to KI:
> $ rly paths generate umee-betanet-1 kichain-t-4 umee_to_ki_path --port=transfer

## Output:
```
Generated path(umee_to_ki_path), run 'rly paths show umee_to_ki_path --yaml' to see details
```

## Check if everything's ok:
> $ rly chains list

## Output:
```
0: umee-betanet-1       -> key(✔) bal(✔) light(✔) path(✔)
1: kichain-t-4          -> key(✔) bal(✔) light(✔) path(✔)
```

## Open channel from UMEE to KI:
> $ rly tx link umee_to_ki_path

## If no output - "Channel created", repeat command for opennig the chanel. 
## Output:
```
[2021-09-12|05:37:10] ★ Clients created: client(07-tendermint-1) on chain[umee-betanet-1] and client(07-tendermint-13) on chain[kichain-t-4]
[2021-09-12|05:37:10] ★ Connection created: [umee-betanet-1]client{07-tendermint-1}conn{connection-1} -> [kichain-t-4]client{07-tendermint-13}conn{connection-17}
[2021-09-12|05:37:11] ★ Channel created: [umee-betanet-1]chan{channel-0}port{transfer} -> [kichain-t-4]chan{channel-61}port{transfer}
```
  
## Send tokens from UMEE to KI:
> $ rly tx transfer umee-betanet-1 kichain-t-4 1000000uumee tki.......................... --path umee_to_ki_path

## Output:
```
[2021-09-12|05:37:45] ✔ [umee-betanet-1]@{289955} - msg(0:transfer) hash(203CB6D10C8F4CA225BFC089F25AB957FC1EFCBEEDFF2749DE159ACE5E00DC5C)
```
  
## Check balance:
> $ rly query balance kichain-t-4

## Output:
```
$ rly query balance kichain-t-4
10000000utki
```

## Edit "channel-id" lines in umee_to_ki_path in config.yaml if balance hasn't change:
```
paths:
  umee_to_ki_path:
    src:
      chain-id: umee-betanet-1
      client-id: 07-tendermint-9
      connection-id: connection-54
      channel-id: channel-0
      port-id: transfer
      order: UNORDERED
      version: ics20-1
    dst:
      chain-id: kichain-t-4
      client-id: 07-tendermint-290
      connection-id: connection-303
      channel-id: channel-61
      port-id: transfer
      order: UNORDERED
      version: ics20-1
    strategy:
      type: naive
```
  
## Re-open channel and wait for "Channel created":
> $ rly tx link umee_to_ki_path

## Transfer tokens from Umee:
> $ rly tx transfer umee-betanet-1 kichain-t-4 1000000uumee tki................................. --path umee_to_ki_path

## Check the balance:
> $ rly query balance kichain-t-4

## Transfer tokens from Ki. 

## Update clients:
> $ rly light init umee-betanet-1 -f
> $ rly light init kichain-t-4 -f

## Create path from Ki to Umee:
> $ rly paths generate kichain-t-4 umee-betanet-1 ki_to_umee_path --port=transfer

## Open channel and wait for "Channel created":
> $ rly tx link ki_to_umee_path

## Check the balance:
> $ rly query balance umee-betanet-1

## Command for transaction:
> $ rly tx transfer kichain-t-4 umee-betanet-1 1000000utki umee1........................... --path ki_to_umee_path

## Edit "channel-id" lines in umee_to_ki_path in config.yaml if balance hasn't change:
```
paths:
  ki_to_umee_path:
    src:
      chain-id: kichain-t-4
      client-id: 07-tendermint-295
      connection-id: connection-308
      channel-id: channel-61
      port-id: transfer
      order: UNORDERED
      version: ics20-1
    dst:
      chain-id: umee-betanet-1
      client-id: 07-tendermint-9
      connection-id: connection-60
      channel-id: channel-0
      port-id: transfer
      order: UNORDERED
      version: ics20-1
    strategy:
      type: naive
```

## Re-open channel and wait for "Channel created":
> $ rly tx link ki_to_umee_path

## Command for transaction:
> $ rly tx transfer kichain-t-4 umee-betanet-1 1000000utki umee1........................... --path ki_to_umee_path

## Check the balance:
> $ rly query balance umee-betanet-1

## Make 5 transaction and save the hashes.

# Possible issues

## If error raised after "rly tx transfer":
```
Error: failed to get trusted header, please ensure header at the height 286777 has not been pruned by the connected node: verify from #288349 to #288957 failed: old header has expired at 2021-09-12 05:35:58 +0000 UTC (now: 2021-09-12 05:39:53 +0200 CEST)
```

## delete light folder in .relayer and use commands below:
> $ rly light init umee-betanet-1 -f
> $ rly light init kichain-t-4 -f
> $ rly tx link

## If error raised after "rly tx link ":
```
Error: failed to update off-chain light client for chain kichain-t-4: verify from #288349 to #288957 failed: old header has expired at 2021-09-12 05:35:58 +0000 UTC (now: 2021-09-12 05:39:53 +0200 CEST2)
```
  
## update clients and repeat commands below:
> $ rly light init umee-betanet-1 -f
> $ rly light init kichain-t-4 -f
> $ rly tx link

# List of hashes
  
## From Umee to Ki:
```
203CB6D10C8F4CA225BFC089F25AB957FC1EFCBEEDFF2749DE159ACE5E00DC5C
EA384E26D7A383BF71BA1ABA67E3E93282E58606E46BE952249B5113F59EAB37
BA185B685196C3197748619B9B57D488AF0CE70E5B28496A49D7A2E31B911846
0F9C5BE87342BDBA25B054DBF329240D8DA457B6F2130FD389787D2E6E55B3EA
2CE1C27E3C0EB3F242C6F1C8566C5D6426E6E1642EB23A8CEF4DE3239E1F67F2
```  
## From Ki to Umee:
```
C141A63EA825700D305A91D5B517F9753CA61E497402D6729BA0F241F6C4664B
21A32227DC5BF19D9D0E92FDEEDBF2C8B118B11BC100F5CEFD8A8E46B4D03FDB
0DC778BC785AAAFF8600733BB08D2AB2DC9E4AEF2BCA71F9B40C0538B0567DE1
D871397C228CD6162BB110555756B5955B0C8942305DB68B7D28B58FAFCA011B
6F018CEEB0DBB1920EB1EBD74FDE4F409BB56A57229B51A5ADB5A5807CDD3ECF
```

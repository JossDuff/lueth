# LuEth

Private educational Ethereum network for Lehigh University Blockchain Lab.  No assets on this chain hold value.  To interact with this network or block explorer you will have to either be on Lehigh University wifi or the Lehigh University VPN.

# About
LuEth is a [private Besu](https://besu.hyperledger.org/private-networks) node running an EVM blockchain.  This blockchain is completely cut off from all peer to peer connection, it is the sole source of truth for the LuEth network.  Students and educators can interact with LuEth by making rpc calls to the endpoint http://vitalik.cse.lehigh.edu:8545.  This is also the url that students will supply to metamask to make transactions.

Example rpc call to get the current block number of LuEth:
```bash
curl -X POST -H "Content-Type: application/json"   --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'   http://vitalik.cse.lehigh.edu:8545
```

# Starting and stopping
This node runs out of a `docker-compose.yml` file.
To see all running docker containers: `docker ps` and look for `lueth-besu-1`.

To stop:
```bash
# make sure you're in ~/lueth-besu
docker compose down
```

To start:
```bash
# make sure you're in ~/lueth-besu
# the -d argument makes it run in the background
docker compose up -d
```

To check logs:
```bash
docker logs lueth-besu-1
```

You can start and stop at will without losing any data.  The chain will simply pick up at the block it left off.  To remove all data, include the `-v` argument in `docker compose down -v` which deletes all volumes.  WARNING: don't do this unless you have a backup of the chain data.  If you do this without a backup, LuEth will start from block 0 and lose all state.

# Updating
The goal of this network is to maintain feature parity with the Ethereum blockchain.  Ethereum has somewhat frequent updates - 1 or 2 a year.  Ethereum updates consist of an execution client update and a consensus client update.  We are only concerned with the execution client updates because we don't run Ethereum consensus in LuEth (see [#what-lueth-can-and-cant-be-used-for]).  Check [this official list](https://github.com/ethereum/execution-specs#ethereum-protocol-releases) of ethereum execution client updates.  At the time of writing this (August 30th, 2025), LuEth is up to date for the most recent Ethereum execution client update, `Prague`.

Read the Besu [upgrade guide](https://besu.hyperledger.org/private-networks/how-to/upgrade) along with the description below.

When a new execution client update is released we must update our LuEth node to maintain feature parity and ensure LuEth has all the features of the Ethereum network.  First, check the Besu github and wait for them to make a release that includes the features for the new update.  For example, [their release for the Pectra update](https://github.com/hyperledger/besu/releases/tag/25.4.0).  Ethereum names their releases as a combination of both the name of the execution client update and the consensus client update.  "Pectra" = "Prague" (name of the execution client update) + "Electra" (name of the consensus client update).  I know, its a bit confusing. When you see that the Besu team has made a release, go into the `genesis.json` file and add a new item under config with the name of the execution client update and the UNIX TIMESTAMP that this update will go into effect.  Make sure this is a time ~20 minutes in the future from the current LuEth block (get the current LuEth block with the curl command above).  DO NOT MAKE ANY OTHER CHANGES TO `genesis.json`.

Heres an example scenario.  It is currently time 1759861377, and the `prague` execution client update just came out as part of the `pectra` Ethereum release.  We saw that the Besu team made an update to their client on github so we are ready to update LuEth.  We want to apply this update at time 1759862377.
`genesis.json` before adding the update:
```
    # lines omitted
    "arrowGlacierBlock": 0,
    "grayGlacierBlock": 0,
    "shanghaiTime": 0,
    "cancunTime": 0,
    # REPLACE THIS COMMENT WITH THE EXECUTION CLIENT UPDATE NAME AND BLOCK NUMBER IT WILL GO INTO EFFECT.
    # MAKE SURE IT IS ~100 BLOCKS AHEAD OF THE CURRENT LUETH BLOCK.
    "clique": {
            "blockperiodseconds": 12,
            "epochlength": 30000,
            "createemptyblocks": true
    }
    # lines omitted
```

`genesis.json` AFTER adding the update:
```
    # lines omitted
    "arrowGlacierBlock": 0,
    "grayGlacierBlock": 0,
    "shanghaiTime": 0,
    "cancunTime": 0,
    # PRAGUE WILL GO INTO EFFECT AT THIS TIMESTAMP
    "pragueTime": 1759862377,
    "clique": {
            "blockperiodseconds": 12,
            "epochlength": 30000,
            "createemptyblocks": true
    }
    # lines omitted
```

Then pull the latest docker image of Besu that contains this update by simple `docker compose down; docker compose up -d` from the `~/lueth-besu` folder.  You're all done!  The change should go into effect by block 200.

# Changing block time
At some point in the future Ethereum will lower its block time from 12 seconds.  LuEth is currently hard coded to 12 seconds and we will have to make another change to `genesis.json`, similar to above, to change this.  Follow [this Besu guide](https://besu.hyperledger.org/private-networks/how-to/configure/consensus/clique#configure-block-time-on-an-existing-network) to change the block time of LuEth.  Remember to set the `block` that this change goes into effect to some block time in the future.  Again, restart the node when this change is made.

# What LuEth can and can't be used for
LuEth is configured to be a suitable testing ground for any smart contract or transactional work that students might want to do.  It has the same block time and EVM version as mainnet Ethereum.  However the gas fees a significantly lower than mainnet Ethereum.

What LuEth is NOT suitable for is consensus or peer to peer network research.  The LuEth node running on this machine has p2p networks disabled and runs a proof of authority consensus ([clique](https://besu.hyperledger.org/private-networks/how-to/configure/consensus/clique) proof of authority): it is the sole producer of blocks and doesn't accept any peer node connections.

# Files in this folder
`genesis.json` contains the genesis state of the LuEth network.  It notes all the execution client updates and the block it occured at.  It also notes the initial FakeEth allocation to a few addresses.  If you wish to receive some FakeEth, email Professor Korth at hfk2@lehigh.edu.  Most of the values in this file are constants and shouldn't be changed with the exception of the `config` section that will need to be added to when we upgrade LuEth.

`docker-compose.yml` is the docker compose file that indicates the files and volumes needed to start/stop the LuEth node as well as the docker image that contains the Besu client we are running.  It is unlikely you will ever need to change this file.

`private_key` is the private key of the LuEth node that is used to sign blocks.  This private key contains no real funds but it shouldn't be changed or sent anywhere.

`config.toml` is the configuration file for LuEth's Besu node.  Config values might need to be updated or changed in the future.  Be careful, as some config changes will require a network restart.  Find a full list of config values [here](https://besu.hyperledger.org/public-networks/reference/cli/options).

# LuEth Block explorer
LuEth's block explorer is hosted at `http://vitalik.cse.lehigh.edu:8040` running out of `~/blockscout`.  It is a [blockscout](https://github.com/blockscout/blockscout/) block explorer.  Env values can be found [here](https://docs.blockscout.com/setup/env-variables).  It can be a bit of a pain to configure.  It also runs in docker compose.  Start and stop it from the `~/blockscout/docker-compose` folder with the commands
```bash
# stopping
# add the -v flag if you also want to delete it's volumes
docker compose -f no-services.yml down
```
```bash
# starting
docker compose -f no-services.yml up -d
```


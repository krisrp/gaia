[![CircleCI](https://circleci.com/gh/cosmos/gaia/tree/master.svg?style=svg)](https://circleci.com/gh/cosmos/gaia/tree/master)

# Cosmos-Hub Staking Module

This project is a demonstration of the Cosmos-Hub with basic staking
functionality staking module designed to get validators acquainted
with staking concepts and procedures.

The validator set is determined as the validators with the top 100 delegated atoms, and is updated every block. Currently, all bonding and unbonding is instantaneous. Absent features are validator rewards and the unbonding wait period.

See the [cosmos-sdk documentation](https://cosmos-sdk.readthedocs.io) for more.

## For a slightly outdated video tutorial, please watch this:

[![Cosmos Validator Testnet Setup Video](https://img.youtube.com/vi/B-shjoqvnnY/0.jpg)](https://www.youtube.com/watch?v=B-shjoqvnnY)


## Prerequisites

You need [Go](https://golang.org/) installed and $GOPATH configured.

## Installation

```
go get github.com/cosmos/gaia
cd $GOPATH/src/github.com/cosmos/gaia
make all
```

Verify that you have the correct version (v0.5.0) installed with the command

```
gaia version
```

## Initialize the client

Initialize and connect the gaia client to a full node.

```
gaia client init --chain-id=gaia-2 --node=tcp://gaia-2-node0.testnets.interblock.io:46657
```

Verify that the validator hash matches `0F9453689828B8EF14F71514339C2506B826BBF9` then select `y`.

This acts as a lite client which allows for simplified key management without needing to run a full node.

### Create a wallet address

```
MYNAME=<your name>
gaia client keys new $MYNAME
gaia client keys list
MYADDR=<your newly generated address>
```

*IMPORTANT*: Carefully write down the recovery phrase on a piece of paper so
you can recover your account later on!

Now visit the testnet faucet and send yourself some `fermion` tokens by using [this utility](http://www.cosmosvalidators.com/).
Fermions are the native staking token on the gaia testnet.

Check your balance to see if you've received them:

```
gaia client query account $MYADDR
```

### Stake your tokens with a validator

List all the validators and their public keys.

```
gaia client query candidates
```

Choose one of the validators `pubkey` from the `data` field.
You can query information about each of the validators with

```
gaia client query candidate --pubkey=<the validator public key from above>
```

And you can stake tokens against it with

```
gaia client tx delegate --amount=1fermion --name=$MYNAME --pubkey=<the validator pubkey from above>
```

Check to see if your account balance has reduced

```
gaia client query account $MYADDR
```

Recheck the validator `shares` and `voting_power` balance to see if they've received your tokens

```
gaia client query candidate --pubkey=<the validator public key from above>
```

Unbond your shares from the validator

```
gaia client tx unbond --shares=1 --name=$MYNAME --pubkey=<the validator pubkey from above>
```

Finally recheck your balance to verify that your token is back.
*Note*: There is no wait time for unbonding on the current testnet, but there
will be in the final release of Cosmos Hub. Be careful who you choose to
delegate to!

## Run your own full node

Check that the computer you wish to run your node on meets the [minimum
hardware requirements](https://cosmos.network/validators/faq)

You need to tell your node which network to connect to. Here we'll be using the
gaia-2 testnet.

```
GAIANET=$GOPATH/src/github.com/cosmos/gaia/testnets/gaia-2/gaia
cd $GAIANET
```

The next command tells your node to download the *entire* testnet blockchain, so expect it to take
from minutes to hours depending on the speed of your internet connection.

```
gaia node start --home=$GAIANET  &> gaia.log &
```

You can view the progress of it with `tail -f gaia.log`.
Once blocks slow down to about one block per second, you're all caught up.

The `gaia node start` command automatically generates a validator private key found in
`$GAIANET/priv_validator.json`. Let's get the pubkey data for our validator
node. The pubkey is located under `"pub_key"{"data":` within the json file

```
cat $GAIANET/priv_validator.json
MYVALIDATORPUBKEY=<your newly generated address>
```

If you have a json parser like `jq`, you can get just the pubkey like so:

```
MYVALIDATORPUBKEY=$(cat $GAIANET/priv_validator.json | jq -r .pub_key.data)
```

Now instead of connecting the lite client to the `gaia-2-node0.testnets.interblock.io:46657` node, you can connect to your own full node.

Next let's initialize the gaia client to start interacting with the testnet:

## WARNING: You wrote down your recovery phrase earlier didn't you!?

If not, go back and write it down before continuing!

```
# DANGER - FINAL WARNING!: This command wipes out your gaia accounts. Have your recovery phrase written down before you run it. No turning back!
gaia client init --chain-id=gaia-2 --node=tcp://localhost:46657
```

Did you get an ERROR telling you to `--force-reset if you
really want to wipe it out`? Good! Add this option then rerun it. It's to stop you
from being lazy and copy-pasting all these commands.

## Recover your account(s)

It's important to be able to recover your account. Anyone with that recovery phrase
can spend your money. Don't be like those people who lost $100 million dollars
worth of Bitcoin on an old hard disk because they never wrote down their
recovery phrase.

```
gaia client keys recover $MYNAME
# Then enter your recovery phrase
```

Repeat this for any other accounts on this testnet that you wish to recover.

Check your balance using the above client commands and verify that your full
node is working.

## Make your node a validator

The top 100 nodes with the highest amount of tokens delegated to them will be
validators. All you need to do to become a testnet validator is delegate some
tokens to your node.

You need MYVALIDATORPUBKEY from above for this.

```
gaia client tx delegate --amount=1fermion --name=$MYNAME --pubkey=$MYVALIDATORPUBKEY
```

Check if your validator pubkey appears in the list of validators

```
gaia client query candidates
```

Unbonding is as easy as running the earlier command.

*IMPORTANT*: Remember to unbond before stopping your node!


### Additional examples

Refer to the [cosmos-sdk documentation](https://cosmos-sdk.readthedocs.io) for further examples, like how to run your own local testnet.

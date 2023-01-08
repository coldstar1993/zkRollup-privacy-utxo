# Architecture
Anomix Network is a layer2 Of Mina, based on UTXO account model, with capabilities to make your fund operations private, anonymous and un-tracable.

Within Anomix Network, you will get a totally new L2 account derived from L1 account, and any one could not link L2 account with L1 account util it obtains the private key of L1 account, meaning any of your operations in Anomix Network could neither be linked with L1 account.

With aid of Anomix Network, you could benefit from `anonymous & priate` fund operations, such as `depositing fund from L1, withdrawing fund to L1, transferring fund to any one within L2`, etc. No body could trace your funds circulation, because all sensitive parts about your operations happen on your local devices (leveraging zero knowledge proof) and all your status info are also stored within L2 network under solid encryption.

Based on UTXO account model, all your behaviors including account management & fund operations are via `Note` pattern within Anomix Network. 
* account management is via `account notes`
* fund operations are via `value notes`

Go through this documentation for More details!

## Network Components & Roles
Anomix Network contains three major components described as belows:

`User zkApps`
* integrated with `Anomix SDK` to interact with Anomix Network
* calculate L2 fund operations zkproof and send as a L2 tx to `Anomix Sequencer`
  * Anomix L2 User Tx Circuit

`Anomix Sequencer`
* recieve, validate and store L2 User Tx from `User zkApps`
* calculate recursive zkProof for latest L2 User Tx
  * Anomix L2 Inner Rollup Circuit
  * Anomix L2 Outer Rollup Circuit
* publish rollup block within a L1 tx to `Anomix Rollup Processor Contract`
* maintain L2 status(i.e. each merkle trees)
* listen for onchain events, .etc.

`Anomix Rollup Processor Contract`
* deployed on Mina L1
* verify each rollup block associated with its zk-Proof from `Anomix Sequencer`
* maintain the merkle tree root of the whole Anomix Network

Besides, Anomix Network provide an SDK for any zkApps to make a integration:
* `Anomix SDK`

And, to improve the efficiency of proof generation at `User zkApps` and `Anomix Sequencer`, we design a seperated standalone zkproof-generating service:
* `Anomix Proof Generator`
  * recieve zkproof generating requests from `User zkApps` and `Anomix Sequencer` and return proof result

Further, Anomix Network has its own L2 explorer for user to take a high-level or detailed look at L2 status.
* `Anomix L2 Explorer`

## Storage Layers
All L2 tx from `User zkApps` are persisted into Database by `Anomix Sequencer` after basical validation.

When Anomix Network starts, `Anomix Sequencer` constructs Multiple Merkle Trees in memory based on all confirmed L2 tx set. These Merkle trees represent the Latest status of the whole L2 Network.
* data tree
  * account note 
  * value note 
* nullifier tree
  * account note nullifier
  * value note nullifier
* root tree
  * records all historical root of data tree.

## Overview of Rollup Processing
<img src="./pic/AnomixNetwork_Architeture.png" style="border-radius: 20px">
The pic briefly illustrates the overview of components & roles inside Anomix Network, as well as their cooperations. Now, let us make a brief description on the total flow

* Any zkApps integrated with anomix sdk could easily leverage anonymous & private fund operations from anomix network. As pic, when user journey arrives funds operations(like pay), user locally construct a L2 user tx with zkproof and send it to `Anomix Sequencer`
* `Anomix Sequencer` is responsible for serveral tasks
  * recieve L2 user tx, and make a basical check on tx format & field validity, and then store it into database.
  * a rollup pipeline would be started up in time to retrieve latest pending L2 tx from DB, divide them into pieces of given amount each, and make an proof aggregation for L2 user tx in each piece. we call this progress 'inner rollup proof generation' (leverage calculation service from the standalone `Anomix Proof Generator`).
  * serveral inner rollup proof are further integrated into the final 'outer rollup proof' (leverage calculation service from the standalone `Anomix Proof Generator`). 
  * invoke `Anomix Rollup Processor Contract` to make final circuit witness and generate final proof, then construct the final L1 tx and send it to Mina Network.
  * `Anomix Sequencer` always listens for the L1 tx confirmation and maintain the Merkle tree root.


# Flows

## Account Creation & Registration & Migration

Within Anomix Network, L2 Account is totally different from L1 account. In general, Anomix account consists of three parts: 
* a unique account alias
  * beside traditional wallet address, within Anomix Network, people could send funds to any one via target's account alias. 
* viewing key pairs
  * for encryption&decryption of notes, like account notes, value notes, note nullifiers.
* spending key pairs
  * to sign L2 tx for spending value notes.

When you first create the Anomix account, it just consists of viewing key pairs, in which the public key for note encryption and the private key for note decryption as well as note spending.

Further, You had better make an account registration, meaning registering a unique account alias as further recipient name, as well as registering serveral spending key pairs for `more convinient & secure` note spending across multiple devices.

### Account Creation
As a new user entering Anomix Network, you need to create your own `Account`. Anomix account is a key pair derived uniquely from your L1 wallet, making usage of different cryptographic algorithm from Mina account generation. Each L1 wallet could only generate one corresponding Anomix L2 account. 

Brief progress of L2 account generation:
* Anomix client asks your explorer wallet extension like `Auro wallet`, for the signature of a specific data piece.
* Anomix client generate your L2 account's private key from the signature, then generate your L2 account's public key. The key pairs are the `viewing keys` of your L2 account.

Then Your own Anomix L2 account is born officially! 

Apparently you `never` need to specially-carefully-privately keep L2 account's private key, Since it could be generated any time with your L1 account any where.


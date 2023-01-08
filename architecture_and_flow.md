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


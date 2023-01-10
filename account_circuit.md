In this section, Let us walk through the detailed Account operations journey!

## Account Creation
Recall the `Account Creation` section on [architecture_and_flow.md](./architecture_and_flow.md)

Within _fund deposit_ and _fund transfer_ flow, if recipient never registers a  _account spending key_ : 
* senders are not supposed to specify that the new _value note_ must be spent by specified _account spending key_, 
* Senders directly encrypt the new _value note_ with recipient's _account viewing key_.
* Then, recipient could decrypt & spend the new _value note_ by _account viewing private key_.

Further detail about the circuit, go to [join_split_circuit.md](./join_split_circuit.md)

## Account Registration 
Recall that in previous descriptions, within `Account Registration`, user has to provide an alias and spending keypair, and construct an **Account Note**.

User Journey as blow:
1. user provide an alias and spending keypair,(normally Anomix client will first check uniqueness & correctness of them, and prompt user if any thing wrong)
2. circuit pseudo code (zkProgram)
  * circuit inputs (_highlighted fields are public inputs_)
    * alias
    * `alias_nullifier`
    * non-existence merkle proof of _alias_nullifier_

    * new account spending key pair
    * non-existence merkle proof of this new _Account Note_ on _data tree_
    * new `account spending key nullifier`
    * non-existence merkle proof of new _account spending key nullifier_
    * signature from _account spending private key_ on (_alias_nullifier_, _new account spending key nullifier_)

    * account viewing private key,
    * account viewing public key,
    * existence merkle proof of _account viewing key_
    * account viewing key nullifier,
    * non-existence merkle proof of _account viewing key nullifier_
    * signature from _account viewing private key_ on (_alias_nullifier_, _new account spending key nullifier_)

    * // if this not the first registration, you must know one of the existing _account spending key_
      * an exsiting account spending key
      * existence merkle proof of exsiting _account spending key_
      * exsiting _account spending key nullifier_
      * non-existence merkle proof of exsiting _account spending key nullifier_
      * signature from _account spending key_ on (_alias_nullifier_, _new account spending key nullifier_)

  * circuit constraints
    * CHECK non-existence merkle proof of _account viewing key nullifier_
    * VERIFY signature by _account viewing private key_ on (_alias_nullifier_, _account spending key nullifier_)

    * CHECK existence merkle proof of _account viewing key_ 
    * CHECK non-existence merkle proof of _account viewing key nullifier_
    * VERIFY signature from _account viewing private key_ on (_alias_nullifier_, _account spending key nullifier_)
    
    * // if this not the first registration, you must know one of the existing _account spending key_
      * CHECK existence merkle proof of exsiting _account spending key_
      * CHECK _account spending key nullifier_ is from HASH(_account_viewing_key_nullifier_, _spending_public_key_)
      * CHECK non-existence merkle proof of exsiting _account spending key nullifier_
      * VERIFY signature from _account spending key_ on (_alias_nullifier_, _new account spending key nullifier_)

    * CHECK _alias_nullifier_ is from _alias_
    * CHECK non-existence merkle proof of _alias_nullifier_

    * CHECK _account spending key nullifier_ is from _account spending key_
    * CHECK non-existence merkle proof of this new _Account Note_ on _data tree_
    * CHECK non-existence merkle proof of _account spending key nullifier_

1. Explorer wallet extention will be triggered to construct and broadcast a L1 tx with given amount of registration fee.

_NOTE:_ Anomix Network does not ensure one _account spending keypair_ is registered in Accounts of different _alias_. i.e. if a user might create multiple accounts(of seperate _alias_) based on different L1 wallets, then he might register the same spending keypair for his different accounts. But this is not really suggested!


## Account Migration
_Account Migration_ normally happens when _viewing private key_ or _spending keys_ are suspected being exposed.

Totally, _Account Migration_ means you change _viewing private key_ or _spending keys_ of your account to new ones. That's to say, you make the existing  _viewing private key_ or _spending keys_ invalid by nullifiers. But you are not able to change your unique alias. 

Before _Account Migration_, do actions ASAP as below:
* To avoid losing funds, You must ASAP spend all your unspent `value notes` into new `value notes` with old _spending keys_. And the New `value notes` should be encrypted by new _viewing private key_(though it's not registered).
* You must encrypt all your exsiting `value notes` with new _viewing private key_.

You can see, _Account Migration_ is a big action!

### User Journey as blow:
1. if you only expose _account viewing private key_, then just change _account viewing key_.
    * _account viewing key_ is derived from L1 account. change _account viewing key_ means changing L1 account to a new one for the L2 account(alias).
2. if you only expose one _account spending private key_, then just change the _account spending key_

#### change _account viewing key_
Scenario steps: 
1. when `Account Migration` flow starts, user is prompted to switch the explorer wallet extention(like Auro wallet) to a new L1 account.  
2. Then Anomix client asks your explorer wallet extension for the signature of a specific data piece.
3. Anomix client generate your L2 account's private key from the signature, then generate your L2 account's public key. The key pairs are the new viewing keys of your L2 account.
4. circuit pseudo code (zkProgram):
  * circuit inputs (_highlighted fields are public inputs_)
    * new account viewing private key,
    * `new account viewing public key`,

    * alias
    * `alias_nullifier`
    * existence merkle proof of _alias_nullifier_

    * existing account viewing private key,
    * existing account viewing public key,
    * existence merkle proof of existing _account viewing key_
    * existing account viewing key nullifier,
    * non-existence merkle proof of _account viewing key nullifier_
    * signature from existing _account viewing private key_ on (_alias_nullifier_, _new account viewing public key_)

  * circuit constraints
    * CHECK _alias_nullifier_ is from _alias_
    * CHECK non-existence merkle proof of _alias_nullifier_

    * CHECK existence merkle proof of existing _account viewing key_
    * CHECK existing account viewing key nullifier is from _existing account viewing public key_,
    * CHECK non-existence merkle proof of _account viewing key nullifier_
    * VERIFY signature from existing _account viewing private key_ on (_alias_nullifier_, _new account viewing public key_)

    * CHECK `new account viewing public key` is generated from _new account viewing private key_
5. When this L2 tx is done, then Anomix client will re-encrypt all historical value notes with `new account viewing public key`.

#### change one _account spending key_
Totally, you first make existing _account spending key_ invalid by nullifier, and then register a new _account spending key_.

Scenario One -- user just update _account viewing key_: 
* no need to make the exposed _account spending key_ nullified, because its originally attached _account viewing key_ is nullified, then senders will not choose this _account spending key_ any more (circuit will check). 
  
  i.e. the _account spending key_ is seen as 'nullified' defaultly although it's not on nullifier tree!

Scenario Two -- user never update _account viewing key_: 
1. User provide a new _account spending key_
2. circuit pseudo code (zkProgram):
  * circuit inputs (_highlighted fields are public inputs_)
    * alias
    * `alias_nullifier`
    * existence merkle proof of _alias_nullifier_
  
    * new account spending private key,
    * `new account spending public key`,
    * non-existence merkle proof of this new _Account Note_ on _data tree_
    * `new account spending key nullifier`
    * non-existence merkle proof of this new _account spending key nullifier_

    * existing account spending private key,
    * existing account spending public key,
    * existence merkle proof of existing _account spending key_
    * `existing account spending key nullifier`,
    * non-existence merkle proof of existing _account spending key nullifier_
  
    * account viewing private key,
    * account viewing public key,
    * existence merkle proof of _account viewing key_
    * account viewing key nullifier,
    * non-existence merkle proof of _account viewing key nullifier_
    * signature from _account viewing private key_ on (_alias_nullifier_, _new account spending key nullifier_)

  * circuit constraints
    * CHECK _alias_nullifier_ is from _alias_
    * CHECK non-existence merkle proof of _alias_nullifier_

    * CHECK existence merkle proof of _account viewing key_
    * CHECK _account viewing key nullifier_ is from HASH(_alias nullifier_, _account viewing public key_)
    * CHECK non-existence merkle proof of _account viewing key nullifier_
    * VERIFY signature from _account viewing private key_ on (_alias_nullifier_, _new account spending key nullifier_)
  
    * CHECK existence merkle proof of existing _account spending key_
    * CHECK existing account spending key nullifier` is from HASH(_account viewing key nullifier_, _existing account spending public key_),
    * CHECK non-existence merkle proof of existing _account spending key nullifier_

    * CHECK non-existence merkle proof of this new _Account Note_ on _data tree_
    * CHECK _new account spending key nullifier_ is from  HASH(_account viewing key nullifier_, _new account spending public key_),
    * CHECK non-existence merkle proof of _account spending key nullifier_
    * CHECK if _new account spending private key_ is derived from _new account spending public key_
  
3. When this L2 tx is done, _existing account spending public key_ will be nullified (i.e. existing _account spending key nullifier_ will be on _nullifier tree_) and  new _Account Note_ is on _data tree_.
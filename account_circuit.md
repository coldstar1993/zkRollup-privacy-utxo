In this section, Let us walk through the detailed Account operations journey!

## Account Creation


## Account Registration 
Recall that in previous descriptions, within `Account Registration`, user has to provide an alias and spending keypair, and construct an **Account Note**.

User Journey as blow:
* user provide an alias and spending keypair,(normally Anomix client will first check uniqueness & correctness of them, and prompt user if any thing wrong)
* circuit pseudo code
  * circuit inputs (_highlighted fields are public inputs_)
    * alias
    * `alias_nullifier`
    * non-existence merkle proof of _alias_nullifier_

    * new account spending key pair
    * non-existence merkle proof of this new _Account Note_ on _data tree_
    * new `account spending key nullifier`
    * non-existence merkle proof of new _account spending key nullifier_
    * signature from _account spending private key_ on (_alias_nullifier_, _new account spending key nullifier_)

    * // if this is first registration, you have to prove you know _account viewing private key_:
      * account viewing private key,
      * account viewing public key,
      * existence merkle proof of _account viewing key_
      * account viewing key nullifier,
      * non-existence merkle proof of _account viewing key nullifier_
      * signature from _account viewing private key_ on (_alias_nullifier_, _account spending key nullifier_)
    * // else: this if not the first registration, you have to prove you know one of existing valid _account spending private key_:
      * an exsiting account spending key
      * existence merkle proof of exsiting _account spending key_
      * exsiting _account spending key nullifier_
      * non-existence merkle proof of exsiting _account spending key nullifier_
      * signature from _account spending key_ on (_alias_nullifier_, _new account spending key nullifier_)

  * circuit constraints
    * CHECK non-existence merkle proof of _account viewing key nullifier_
    * VERIFY signature by _account viewing private key_ on (_alias_nullifier_, _account spending key nullifier_)

    * // if this is first registration,
      * CHECK existence merkle proof of _account viewing key_ 
      * CHECK non-existence merkle proof of _account viewing key nullifier_
      * VERIFY signature from _account viewing private key_ on (_alias_nullifier_, _account spending key nullifier_)
    * // else: if not the first registration,
      * CHECK existence merkle proof of exsiting _account spending key_
      * CHECK non-existence merkle proof of exsiting _account spending key nullifier_
      * VERIFY signature from _account spending key_ on (_alias_nullifier_, _new account spending key nullifier_)

    * CHECK _alias_nullifier_ is from _alias_
    * CHECK non-existence merkle proof of _alias_nullifier_

    * CHECK _account spending key nullifier_ is from _account spending key_
    * CHECK non-existence merkle proof of this new _Account Note_ on _data tree_
    * CHECK non-existence merkle proof of _account spending key nullifier_

_NOTE:_ Anomix Network does not ensure one _account spending keypair_ is registered in Accounts of different _alias_. i.e. if a user might create multiple accounts(of seperate _alias_) based on different L1 wallets, then he might register the same spending keypair for his different accounts. But this is not really suggested!


## Account Migration
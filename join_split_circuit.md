In Anomix Network, `Anomix Rollup Processor Contract` store all the funds from L1. Each fund deposit or withdraw operation will trigger `Anomix Rollup Processor Contract` to recieve funds from L1 account or withdraw funds from `Anomix Rollup Processor Contract` account to recipient's L1 account.

And regarding fund transfer operations, funds circulation only happens inside L2 internally.

# Deposit funds from L1
In general, one always deposit funds into his own L2 account. But in fact, You could deposit funds to any L2 account.

User deposits L1 funds to `Anomix Rollup Processor Contract` account, and will be watched by `Anomix Sequencer` to generate an L2 `value note` with corresponding fund amount. 

User Journey as blow:
1. user execute method about fund deposits on `Anomix Rollup Processor Contract` locally, and then construct an _L1 tx_ with specified L1 fund amount.
2. circuit pseudo code:
   * circuit inputs (_highlighted fields are public inputs_)
        * user construct a new _value note_ : <br>
            {<br>
                secret, <br>
                owner_pubkey: recipient's viewing pubkey or an spending pubkey <br>
                account_require: 0 or 1, <br>
                creator_pubkey: zero or _L1 address_, <br>
                value: `depositing amount` <br>
                asset_id,  <br>
                input_nullifier: //TODO ??  <br>
            }

        * calculate the `VALUE_NOTE_COMMITMENT`
        * non-existence merkle proof of _VALUE_NOTE_COMMITMENT_

        * calculate the `Value Note Nullifier`
        * non-existence merkle proof of _Value Note Nullifier_

        * `depositing amount`
        * `depositing fee`:  //TODO should fetch the dynamic fee rate to calculate ??

    * circuit constraints
       * CHECK `VALUE_NOTE_COMMITMENT` is from the new _value note_
       * CHECK non-existence merkle proof of _VALUE_NOTE_COMMITMENT_
       * CHECK `Value Note Nullifier` is from the new _value note_
       * CHECK non-existence merkle proof of _Value Note Nullifier_
       * `depositing amount` + `depositing fee` < L1 depositing tx's fund // TODO ??

3. When _L1 tx_ is confirmed, the emited event will be listened to by `Anomix Sequencer` and `Anomix Sequencer` will maintain `data tree` with `VALUE_NOTE_COMMITMENT`

# Transfer funds within L2




# Withdraw funds to L1
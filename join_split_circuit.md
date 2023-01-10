In Anomix Network, `Anomix Rollup Processor Contract` store all the funds from L1. 

Each fund deposit or withdraw operation will trigger `Anomix Rollup Processor Contract` to recieve funds from operator's L1 account or withdraw funds from `Anomix Rollup Processor Contract` account to recipient's L1 account.

And regarding fund transfer operations, funds circulation only happens inside L2 internally.

NOTE: Due to the determinacy requirement of Arithmetic circuit, wihin Joint-Split pattern, Anomix Network currently support up to two _value notes_ as input and up to two _value notes_ as output. You could see that inside the flows below.

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

        * `depositing amount`
        * `depositing fee`:  //TODO should fetch the dynamic fee rate to calculate ??

    * circuit constraints
       * CHECK `VALUE_NOTE_COMMITMENT` is from the new _value note_
       * CHECK non-existence merkle proof of _VALUE_NOTE_COMMITMENT_
       * `depositing amount` + `depositing fee` < L1 depositing tx's fund // TODO ??

3. When _L1 tx_ is confirmed, the emited event will be listened to by `Anomix Sequencer` and `Anomix Sequencer` will maintain `data tree` with `VALUE_NOTE_COMMITMENT`
4. Finally, user should encrypt this new _value note_ with recipient's (user himself normally) _account viewing key_

# Transfer funds within L2
Due to the determinacy requirement of Arithmetic circuit, wihin _fund transfer_ of Joint-Split pattern, Anomix Network currently support up to two _value notes_ as circuit input and up to two _value notes_ as circuit output.

That's to say: 
    ```
    value_note_inputA + value_note_inputB --> value_note_outputC + value_note_outputD
    ```

*Transfer funds within L2* flow covers the scenarios as blow:
1. input one _value note_ -> output one _value note_
   * value_note_inputB == null & value_note_outputD == null
2. input one _value note_ -> output two _value note_ 
    * value_note_inputB == null
3. input two _value note_ -> output one _value note_
   * value_note_outputD == null
4. input two _value note_ -> output two _value note_
   
// TODO ?? to align with the above, how to implement the constraints inside circuit below??
  * ?? Solution1: 
    * each L2 account initially is distributed with a special **_zero value note_** for each asset,
    * Then 2 input value notes and 2 output value notes are all not null, but keep the extra ones as _zero value note_.

User Journey as blow:
1. user choose two unspent _value notes_ for inputs.
2. user construct two new _value notes_.
     * Note: asset_id in four _value_notes_ above shoule aligned! 
3. circuit pseudo code:
   * circuit inputs (_highlighted fields are public inputs_):
     * value_note_inputA
     * _value_note_inputA commitment_
     * existence merkle proof of _value_note_inputA commitment_ on _data tree_
     * `value_note_inputA nullifier`
     * non-existence merkle proof of _value_note_inputA nullifier_ on _nullifier tree_
     * signature of spending value_note_inputA
 
     * value_note_inputB
     * _value_note_inputB commitment_
     * existence merkle proof of _value_note_inputB commitment_ on _data tree_
     * `value_note_inputB nullifier`
     * non-existence merkle proof of _value_note_inputB nullifier_ on _nullifier tree_
     * signature of spending value_note_inputB
   
     * value_note_outputC :<br>
            {<br>
                secret, <br>
                owner_pubkey: recipientC's viewing pubkey or an spending pubkey <br>
                account_require: 0 or 1, <br>
                creator_pubkey: zero or _L2 sender's alias_, <br>
                value, <br>
                asset_id,  <br>
                input_nullifier: //TODO ??  <br>
            }
     * `value_note_outputC commitment`
     * non-existence merkle proof of _value_note_outputC commitment_ on _data tree_
     * non-existence merkle proof of _value_note_outputC.owner_pubkey_  on _nullifier tree_
     * if _value_note_outputC.account_require_ == 1: 
       * existence merkle proof of _value_note_outputC.owner_pubkey_  on _data tree_

     * value_note_outputD :<br>
            {<br>
                secret, <br>
                owner_pubkey: recipientD's viewing pubkey or an spending pubkey <br>
                account_require: 0 or 1, <br>
                creator_pubkey: zero or _L2 sender's alias_, <br>
                value, <br>
                asset_id,  <br>
                input_nullifier: //TODO ??  <br>
            }
     * `value_note_outputD commitment`
     * non-existence merkle proof of _value_note_outputD commitment_ on _data tree_
     * non-existence merkle proof of _value_note_outputD.owner_pubkey_  on _nullifier tree_
     * if _value_note_outputD.account_require_ == 1: 
       * existence merkle proof of _value_note_outputD.owner_pubkey_  on _data tree_

     * _transfer fee_

   * circuit constraints:
     * CHECK value_note_inputA.value + value_note_inputB.value == value_note_outputC.value + value_note_outputD.value + _transfer fee_

     * CHECK _value_note_inputA commitment_ is from _value_note_inputA_
     * CHECK existence merkle proof of _value_note_inputA commitment_ on _data tree_
     * CHECK _value_note_inputA nullifier_ is from _value_note_inputA_
     * CHECK non-existence merkle proof of _value_note_inputA nullifier_ on _nullifier tree_
     * VERIFY signature of spending value_note_inputA with value_note_inputA.owner_pubkey

     * CHECK _value_note_inputB commitment_ is from _value_note_inputB_
     * CHECK existence merkle proof of _value_note_inputB commitment_ on _data tree_
     * CHECK _value_note_inputB nullifier_ is from _value_note_inputB_
     * CHECK non-existence merkle proof of _value_note_inputB nullifier_ on _nullifier tree_
     * VERIFY signature of spending value_note_inputB with value_note_inputB.owner_pubkey
     
     * CHECK _value_note_outputC commitment_ is from _value_note_outputC_
     * CHECK non-existence merkle proof of _value_note_outputC commitment_ on _data tree_
     * CHECK non-existence merkle proof of _value_note_outputC.owner_pubkey_  on _nullifier tree_
     * if _value_note_outputC.account_require_ == 1: 
       * CHECK existence merkle proof of _value_note_outputC.owner_pubkey_  on _data tree_

     * CHECK _value_note_outputD commitment_ is from _value_note_outputD_
     * CHECK non-existence merkle proof of _value_note_outputD commitment_ on _data tree_
     * CHECK non-existence merkle proof of _value_note_outputD.owner_pubkey_  on _nullifier tree_
     * if _value_note_outputD.account_require_ == 1: 
       * CHECK existence merkle proof of _value_note_outputD.owner_pubkey_  on _data tree_

     * CHECK 4 asset_id equals each other
     * //TODO CHECK 4 input_nullifier ??
4. At last, user construct a L2 tx with the witness and broadcast it to `Anomix Sequencer`. 
5. user encrypt _value_note_outputC_ with recipientC's _account viewing key_
6. user encrypt _value_note_outputC_ with recipientD's _account viewing key_
7. And later soon, `Anomix Sequencer` will maintain _data tree_ and _nullifier tree_.
   * `value_note_inputA nullifier` will be added into _nullifier tree_
   * `value_note_inputB nullifier` will be added into _nullifier tree_
   * `value_note_outputC commitment` will be added into _data tree_ 
   * `value_note_outputD commitment` will be added into _data tree_ 

# Withdraw funds to L1
Similar to _Transfer funds within L2_ flow, _Withdraw funds to L1_ flow align with : 
    ```
    value_note_inputA + value_note_inputB --> value_note_outputC + value_note_outputD
    ```

However, _value_note_outputD_ is totally different with normal _value note_, for tracing withdraw info , seen as below in circuit.

Besides, _value_note_outputD_ will be finally encrypted by sender's own _account viewing key_.

User Journey as blow:
1. user choose two unspent _value notes_ for inputs.
2. user construct two new _value notes_.
     * Note: asset_id in four _value_notes_ above shoule aligned! 

User Journey as blow:
1. user choose two unspent _value notes_ for inputs.
2. user construct two new _value notes_.
     * Note: asset_id in four _value_notes_ above shoule aligned! 
3. circuit pseudo code:
   * circuit inputs (_highlighted fields are public inputs_):
     * value_note_inputA
     * _value_note_inputA commitment_
     * existence merkle proof of _value_note_inputA commitment_ on _data tree_
     * `value_note_inputA nullifier`
     * non-existence merkle proof of _value_note_inputA nullifier_ on _nullifier tree_
     * signature of spending value_note_inputA
 
     * value_note_inputB
     * _value_note_inputB commitment_
     * existence merkle proof of _value_note_inputB commitment_ on _data tree_
     * `value_note_inputB nullifier`
     * non-existence merkle proof of _value_note_inputB nullifier_ on _nullifier tree_
     * signature of spending value_note_inputB
   
     * value_note_outputC :<br>
            {<br>
                secret, <br>
                owner_pubkey: recipientC's viewing pubkey or an spending pubkey <br>
                account_require: 0 or 1, <br>
                creator_pubkey: zero or _L2 sender's alias_, <br>
                value, <br>
                asset_id,  <br>
                input_nullifier: //TODO ??  <br>
            }
     * `value_note_outputC commitment`
     * non-existence merkle proof of _value_note_outputC commitment_ on _data tree_
     * non-existence merkle proof of _value_note_outputC.owner_pubkey_  on _nullifier tree_
     * if _value_note_outputC.account_require_ == 1: 
       * existence merkle proof of _value_note_outputC.owner_pubkey_  on _data tree_

     * value_note_outputD :<br>
            {<br>
                secret, <br>
                **owner_pubkey: recipientD's L1 address,** <br>
                **account_require: 0,** <br>
                creator_pubkey: zero or _L2 sender's alias_, <br>
                value, <br>
                asset_id,  <br>
                input_nullifier: //TODO ??  <br>
            }
     * `value_note_outputD commitment`
     * non-existence merkle proof of _value_note_outputD commitment_ on _data tree_
     * `value_note_outputD nullifier`

     * _transfer fee_

   * circuit constraints:
     * CHECK value_note_inputA.value + value_note_inputB.value == value_note_outputC.value + value_note_outputD.value + _transfer fee_

     * CHECK _value_note_inputA commitment_ is from _value_note_inputA_
     * CHECK existence merkle proof of _value_note_inputA commitment_ on _data tree_
     * CHECK _value_note_inputA nullifier_ is from _value_note_inputA_
     * CHECK non-existence merkle proof of _value_note_inputA nullifier_ on _nullifier tree_
     * VERIFY signature of spending value_note_inputA with value_note_inputA.owner_pubkey

     * CHECK _value_note_inputB commitment_ is from _value_note_inputB_
     * CHECK existence merkle proof of _value_note_inputB commitment_ on _data tree_
     * CHECK _value_note_inputB nullifier_ is from _value_note_inputB_
     * CHECK non-existence merkle proof of _value_note_inputB nullifier_ on _nullifier tree_
     * VERIFY signature of spending value_note_inputB with value_note_inputB.owner_pubkey
     
     * CHECK _value_note_outputC commitment_ is from _value_note_outputC_
     * CHECK non-existence merkle proof of _value_note_outputC commitment_ on _data tree_
     * CHECK non-existence merkle proof of _value_note_outputC.owner_pubkey_  on _nullifier tree_
     * if _value_note_outputC.account_require_ == 1: 
       * CHECK existence merkle proof of _value_note_outputC.owner_pubkey_  on _data tree_

     * CHECK _value_note_outputD commitment_ is from _value_note_outputD_
     * CHECK non-existence merkle proof of _value_note_outputD commitment_ on _data tree_
     * CHECK _value_note_outputD nullifier_ is from _value_note_outputD_

     * CHECK 4 asset_id equals each other
     * //TODO CHECK 4 input_nullifier ??
4. At last, user construct a L2 tx with the witness and broadcast it to `Anomix Sequencer`. 
5. user encrypt _value_note_outputC_ with recipientC's _account viewing key_
6. user encrypt **_value_note_outputD_** with **user itself's _account viewing key_**
7. And later soon, `Anomix Sequencer` will maintain _data tree_ and _nullifier tree_.
   * `value_note_inputA nullifier` will be added into _nullifier tree_
   * `value_note_inputB nullifier` will be added into _nullifier tree_
   * `value_note_outputC commitment` will be added into _data tree_ 
   * `value_note_outputD commitment` will be added into _data tree_ 
   * **`value_note_outputD nullifier` will be added into _nullifier tree_**

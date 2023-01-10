# Account Note
When a new user _register_ her/his _account alias_ and _spending keypair_, she/he first need to construct a new `Account Note` locally.

An **Account Note** associates a spending key with an account. It consists of the following field elements. See the dedicated [account_circuit.md](./account_circuit.md) for more details.

- `alias_hash`: pedersen hash of `account alias`
- `account_public_key`: the account viewing public key
- `spending_public_key`: the spending key that is been assigned to this account via this note.

An account note commitment is:
- `pedersen::compress(alias_hash, account_public_key, signing_pub_key)`
  - Pedersen GeneratorIndex: `ACCOUNT_NOTE_COMMITMENT`

# Value Note
Consists of the following:

- `secret`: a random value to hide the contents of the
  commitment, just for more randomness.
- `owner_pubkey`: i.e. _to_. if recipient never registers account, it must be _account view public key_ of recipient, or it must be one _account spending public key_ of recipient .
- `account_required`: 0--`owner_pubkey` is _account view public key_ of recipient; 1--`owner_pubkey` one _account spending public key_ of recipient.
- `creator_pubkey`: i.e. _from_. Optional, can be zero. Allows the sender of a value note to inform the recipient of who the note came from.
- `value`: the value contained in this note.
- `asset_id`: unique identifier for the 'currency' of this note. Currently we place '`Mina`' token as `0`. On the future, we will add more assets from Mina ecology.
- `input_nullifier`: In order to create a value note, another value note must be nullified (except when depositing, where a 'gibberish' nullifier is generated). We include the `input_nullifier` here to ensure the commitment is unique (which, in turn, will ensure this note's nullifier will be unique).


**partial commitment**

- `pedersen::compress(secret, owner_pubkey, account_required, creator_pubkey)`
  - Pedersen GeneratorIndex: `VALUE_NOTE_PARTIAL_COMMITMENT`
  - `creator_pubkey` can be zero.

> _Note:_ The `secret` is to construct a hiding Pedersen commitment to hide the note details.

**complete commitment*

- `pedersen::compress(value_note_partial_commitment, value, asset_id, input_nullifier)`
  - Pedersen GeneratorIndex: `VALUE_NOTE_COMMITMENT`
  - `value` and `asset_id` can be zero

# Note encryption and decryption
First of all, no need to encrypt `Account Note`, Because within all _fund deposit_ or _fund transfer_ scenarios, senders always need to fetch recipient's _viewing public key_ for the encryption of newly generated `Value Notes`.

`Value Note` encryption is key for Anomix Network to make fund operations anonymous and private, Since `Value Note` encryption only happens on fund operators local devices without exposing any sentitive info.

_Note:_ All leaves nodes of `data tree`, `nullifier tree` and `root tree` is based on Hash of plain notes instead of encrypted notes, further meaning that all circuits require plain fields as inputs. 

During all flows, it's enough for users just to provide _plain info_ aligned with circuits. Who provides exact _plain info_ has the key to decrypt corresponding notes. Except that, users encrypt the _plain info_ as encrypted notes for privacy.

_Note:_ Regarding encrypted notes, users could place it anywhere, like the common repo or just user's local own devices. But users has to guarantee being able to find it back and decrypt it into _plain info_ before spending it.

# Account Note Nullifier
`Account Note` construction takes place on _account registration_ section.
Account Note Nullifier covers three scenarios:
* alias nullifier
  * when a new user register a new alias, she/he has to provide _non-existence proof_ on `nullifier tree` for the new alias. _That's to say: everyone would have a unique alias!_
  * calculation: `alias nullifier = pedersen::compress(alias)`

  _Note:_ Please don't forget you alias, otherwise you cann't get it back! Animix Network only store `alias nullifier` on _nullifier tree_.

* account viewing key nullifier
  * when user make a `Account Migration` due to viewing key exposure, it has to make the old viewing key into a nullifier, to anounce that the old key is invalid.
  * Then in _fund transfer_ or _fund deposit_ scenarios, sender would check if recipient's viewing key is invalid (i.e. on `nullifier tree`) first before _value note_'s encryption. //TODO should in circuit?
  * calculation: `account_viewing_key_nullifier = pedersen::compress(alias nullifier, account_public_key)`

* account spending key nullifier
  * when user carelessly expose one spending key, then he is supposed to transfer unspent value notes associated with this spending key into a new value notes associated with another spending key! Then he needs to make the exposing _spending public key_ into nullifier, to anounce it's invalid now.
  * Then in _fund transfer_ or _fund deposit_ scenarios, sender would check if recipient's spending key is invalid (i.e. on `nullifier tree`) first.  //TODO should in circuit?
  * calculation: `account_spending_key_nullifier = pedersen::compress(account_viewing_key_nullifier, spending_public_key)`

_NOTE:_ If a user make existing `account viewing key` invalid by nullifier, all `account spending key` registered to this account will be 

# Value Note Nullifier

Nullifier is key for Anomix Network cash system to avoid `Double Spending` inside the UTXO account model.

Each Value Note would derive its unique note nullifier when being spent. `Value Note Nullifier`, as a commitment, will be recorded on the `nullifier tree`.

**Objectives** of this nullifier:

- Only the owner of a note may be able to produce the note's nullifier.
- No collisions. Each nullifier can only be produced for one value note commitment. Duplicate nullifiers must not be derivable from different note commitments.
- No double-spending. Each commitment must have one, and only one, nullifier.
- The nullifier must only be accepted and added to the nullifier tree if it is the output of a join-split circuit which 'spends' the corresponding value note.

**Calculation**
We set out the computation steps below, with suggestions for changes:
- `hashed_pk = account_private_key * G` (where G is a generator unique to this operation).
- `pedersen::compress(value_note_commitment, hashed_pk)`
  - Pedersen GeneratorIndex: `VALUE_NOTE_NULLIFIER_COMMITMENT`



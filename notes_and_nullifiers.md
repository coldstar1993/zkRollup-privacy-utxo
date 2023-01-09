# Account Note
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
- `owner_pubkey`: the public key of the owner of the value note. 
- `account_required`: Is the note linked to an existing account or can the note be spent without an account, by directly signing with the owner key
- `creator_pubkey`: Optional, can be zero. Allows the sender of a value note to inform the recipient about who the note came from.
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
First of all, no need to encrypt `Account Note`, Because within all _fund deposit_ or _fund transfer_ scenes, senders always need to fetch recipient's _viewing public key_ for the encryption of newly generated `Value Notes`.

`Value Note` encryption is key for Anomix Network to make fund operations anonymous and private, Since `Value Note` encryption only happens on fund operators local devices without exposing any sentitive info.

# Account Note Nullifier
When a///////////////////////////////////////////////////

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



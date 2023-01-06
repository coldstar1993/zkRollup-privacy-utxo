# zkRollup-privacy-utxo
DRAFT: a zkRollup design Based on UTXO &amp; Protecting Privacy

# Functionality
**Register a Unique name**: 
layer1 shows you register, but cannot know who you will be in layer2.

**Deposit ETH/ERC20/ERC721**:
layer1 shows you doposit money into Contract, but cannot know who will recieve it.

**Withdraw ETH/ERC20/ERC721**:
layer1 shows a fund from Contract to you, but cannot know from who in layer2 to withdraw.

**Transfer ETH/ERC20/ERC721**:
transfer to anyone by its unique name.<br>
non-interactive layer2 tx.<br>
no one know all detail insides a layer2 tx(from, to, amount, etc.), and no related side-info exposed for people to make further anylasis.

**Tx history viewer**:
could retrieve all tx historys any time when log on.

# Privacy
Anyone knows my name txt, but cann’t trace any info about me;
Account & Note are all encrypted. 
No links between Account & Notes.

**Questions 1**: 
how to efficiently pick out encrypted Notes belonging to me?<br>
**Answer1**: 
Easily to pick out SendNotes by recorded EncryptedSendNoteHashes, but still need to decrypt all RecvNotes’ encrypted AES key to identify all the owner’s RecvNotes.

**Questions 2**: 
how to encrypt note when transferring to the other one and how he/she could decrypt it?<br>
**Answer2**: 
pick their exposed viewPubkey to encrypt the random AES key.


# Data Structure
### Account
```
{
nameHash, 
ipfsGitCommitId,
  viewPubkey[32] <-> EncryptedViewPrivKey[32],
  EncryptedSendNoteHashes:[...], 
}
```
generate new key pairs based on signed msg from metamask.

### SendNote
```
{from, amount, to, timestamp, refId}
```
offchain storage: hash(encrypted SendNote) --> {随机AES密钥的密文(自己viewPubkey)}+viewPubkeyIndx

### RecvNote
```
{from, amount, to, timestamp, refId}
```
offchain storage: hash(encrypted RecvNote) --> {随机AES密钥的密文(对方viewPubkey)}+viewPubkeyIndx


### Account Tree: 
SMT, 记录全量 AccountNames, 仅以AccountName 作为leaf.

### AccountDetail Tree: 
SMT, 记录全量 Account{nameHash, ipfsGitCommitId}.

### SendNote Tree: 
SMT, 记录全量encrypted send Notes.<br>
leaf -> hash encrypted SendNote{from, amount, to, timestamp, refId}

### RecvNote Tree: 
SMT, 记录全量 encrypted recv Notes.<br>
leaf -> hash encrypted RecvNote{from, amount, to, timestamp, refId}

### Nullifier Tree: 
SMT, 仅仅记录已经销毁的encrypted RecvNote.<br>
leaf -> hash encrypted RecvNote{from, amount, to, timestamp, refId}


# Members
### Users: 
events makers.

### Rollup Providers: 
permissionless POS network, broadcast layer2 tx by gossip internally, awards & punishment by work.<br>
select one to verify tx, maintain offchain storage, batch tx, generate recursive proof, generate layer1 Tx and broadcast.<br>

### Rollup Accelerators:
hold the entire Note Tree & Nullifier Tree in memory,<br>
help to calculate (non-)existence proof.<br>

### Rollup Contracts:
Register,<br>
Deposit, Withdraw,<br>
Normal Layer2 Transfer,<br>
verify function for layer1-Tx.<br>

### Storage:
IPFS: Git Version Control, <br>


# Key sections
1)In order to be more efficient, when the selected rollup provider verify one layer2-tx, then the layer2-tx could be seen as confirmed, thus rollup provider will update the off-chain data at once. when accumulating the verified layer2-tx enough, rollup provider will generate a layer1-tx attached with recursive proof of all verified layer2-tx within this window, and send it to layer1 network. <br>
2)To improve privacy and anti-trace & anti-analysis, layer2 client fetch all SendNotes itself via common IPFS Gateway by SendNoteHash. not via IPFS Gateway specified for This Rollup. <br>
3)Client fetch all RecvNotes back, decrypt all by viewPubkey specifed by viewPubkeyIndx. <br>
any RecvNotes that could be decrypted belongs to current viewPubkey owner. So there is no need to add ‘to’ field within RecvNote . <br>
As the same, no need to add ‘from’ field within SendNotes. <br>
all kinds of notes and accounts stored offchain via common IPFS storage. IPFS provide Git Version Control feature for Rollup Provider to maintain the data of exactly right version even though any one could update the data. <br>

# Scenario
## Scenario Register
user connects metamask; <br>
user inputs a unique name & passcode; <br>
client triggers metamask to make a sign, and derives 32 view-keypairs based on the signed msg. <br>
init Account entity: <br>
&ensp;&ensp;encrypt 32 view-privKeys by hash(passcode) as AES key; <br>
&ensp;&ensp;compose Account entity and hash it to get AccountHash; <br>
fetch non-existence merkle proof back by AccountHash from Rollup Accelerators; <br>
calculate witness by circuit; <br>
compose a layer1 tx to Rollup Providers for further verification and final comfirmation. //TODO  how/who to pay gas? <br>


## Scenario Deposit
user connects metamask;<br>
user chooses ETH/ERC20 and inputs the amount;<br>
client construct a RecvNote{from: coinbase, amount, to: current user, timestamp, refId} <br>
generate a random number as AES key and encrypt the RecvNote.<br>
select one viewPubKey to encrypt the AES key and append the encrypted key and viewPubkeyIndx to the encrypted RecvNote as a whole.<br>
fetch respective non-existence merkle proof on RecvNote Tree and Nullifier Tree back by hash from Rollup Accelerators;<br>


## Scenario Withdraw
// TODO <br>


## Scenario transfer
// TODO <br>



## Scenario Logon
// TODO <br>




# Emergency Treatment
1)partial user & partial rollup providers do sth maliciously<br>
2)rollup providers do sth maliciously<br>

# Blocking me:
### 1)can we make a encryption & decryption within a circuit?
I wish to put an encrypted txtA and plain txtB as signal inputs for a circuit. Within circuit, txtA is decrypted and force equals to plain txtB. <br>
could this be done in a circuit??<br>
why circom does not provide such a template for encryption & decryption?<br>

### 2)how to verify in layer2-circuit a tx that has been confirmed on ETH layer1? 
I think it disabled. Since circuit doesnot (should not) have network IO capablity. Besides, ETH layer1 is not seriously absolutely deteministic, so that Circuit are not supposed to depend on such a non-stable stuff.<br>

### 3)if a tx has been confirmed on eth layer1 (confirmed by eyes), how to prove it’s related to the signal inputs for a circuit.
Within Scenario-Deposit, Rollup providers need to verify one deposit notes is of one confirmed layer1-Tx??<br>
**Solution1**:  <br>
user put aesEncrypt(hash(the signal inputs), aesKey) as payload within a layer1_tx. <br>
Within Rollup Provider circuit, force the signal inputs to equal to the decrypted aesEncryptHash. Then we could guarantee the layer1_Tx related to the signal inputs.<br>
But to avoid malicious rollup providers, how to guarantee in circuit the amount of layer1_Tx equals to or is greater than that of layer2 deposit note??<br>

**Solution2**:<br>
could we construct the tx as output within a client circuit?? This method could prove relationship directly for Rollup Provider’s verification.<br>



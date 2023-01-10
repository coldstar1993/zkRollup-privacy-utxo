临时想法：

value_note.`owner_pubkey`: if recipient never registers account, it must be _account view public key_ of recipient, or it must be one _account spending public key_ of recipient .

funding deposit & funding transfer中，电路不会检查 要花费的value note中的owner_pubkey 是否有效, 只要签名复合指定的owner_pubkey，那么value note 就可以被消费掉。

因此，如果你怀疑 某spending private key丢失了，那么最好尽快转移对应的value notes!

可是电路会要求 tx中新创建的value notes中指定的owner_pubkey 有效（即如果account_required=1则要求"account viewing key 有效（即不可以在on nullifier tree）" 和 “owner_pubkey在data tree, 而不可以在on nullifier tree“；如果account_required=0则只要求owner_pubkey不可以在on nullifier tree），主要是防止sender故意作恶使用recipient声明失效的owner_pubkey从而令recipient的资金更容易被盗。

如果恶意或粗心sender在recipient 未register account时，指定错误的account viewing key作为owner_pubkey，那么即使加密采用了正确的account viewing key，对于recipient而言，也同样无法使用此笔utxo，即senders转账入黑洞了。

电路只关注明文input, 至于明文怎么解密来的和之后怎么加密存储，电路不关注的，这是senders转帐成功后决定的。所以电路更加不需要关注viewing key是否失效！

如果你暴露了viewing key, 那么你之前的tx history 都会暴露。你所能做的是转移 unspent value notes，再以新viewing key 加密之，(当然也要重新加密tx history以保持一致性)。
更换viewing key（即existing viewing key失效）, 并不会影响existing spending key的有效性。
{
    alias,
    (invalid) viewing public key,
}
依然在data tree, 但也在nullifier tree了。


{
    alias,
    (invalid) viewing public key,
    spending public key
}
依然在data tree. _spending public key_依然有效。


{
    alias,
    (new) viewing public key,
}
在data tree，而不在nullifier tree.


总结：
* 用户如果暴露了 spending key, 则容易资金丢失。此时需要尽快transfer相关value notes!
* 如果同时暴露了 viewing key 和 spending key, 则可能被恶意者添加新spending key! 此时需要尽快令 viewing key 失效！
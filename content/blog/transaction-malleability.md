+++
title = "Transaction Malleability"
date = 2022-04-25
image = "https://res.cloudinary.com/vladimirfomene/image/upload/v1653069616/blog/addresses.png"
description = "These addresses are like single-use (they could be used multiple times but it is not advised for privacy reasons) invoices that are issued by a receiver to a sender in a transaction"
url = "https://www.vladimirfomene.com/blog/bitcoin-addresses/"
+++

### Introduction

Welcome back to the blog! This week we are going to be looking into Bitcoin transaction malleability. The goal will be to understand the transaction malleability and how it can be used 
by an attacker to defraud a Bitcoin user of their funds. 

###  Bitcoin Transaction Data Structure

Before we can understand transaction malleability, we need to have a good grasp of the Bitcoin
transaction data structure. Bitcoin transactions can either be represented in a serialized format (hexadecimal) or a deserialized format. To understand the serialized format of Bitcoin transactions, let's look at the following serialized transaction. TxId: [c1b4e695098210a31fe02abffe9005cffc051bbe86ff33e173155bcbdc5821e3](https://learnmeabitcoin.com/explorer/transaction/c1b4e695098210a31fe02abffe9005cffc051bbe86ff33e173155bcbdc5821e3)

```hex
01000000017967a5185e907a25225574544c31f7b059c1a191d65b53dcc1554d339c4f9efc010000006a47304402206a2eb16b7b92051d0fa38c133e67684ed064effada1d7f925c842da401d4f22702201f196b10e6e4b4a9fff948e5c5d71ec5da53e90529c8dbd122bff2b1d21dc8a90121039b7bcd0824b9a9164f7ba098408e63e5b7e3cf90835cceb19868f54f8961a825ffffffff014baf2100000000001976a914db4d1141d0048b1ed15839d0b7a4c488cd368b0e88ac00000000
```

To understand this format, we are going to divide it into fields made up of one or many hexadecimal characters. The table below shows the name, size, data and description of each field in the transaction above.

![Transaction Data Structure](/images/transaction-datastructure.png)

[Image Source](https://learnmeabitcoin.com/technical/transaction-data)

> The purple icon at the end of a field's data means the field is represented in little-endian. 
> As you might have suspected from the `input count` and `output count` fields, you can have 
> more than one input or output in a transaction.

On the other hand, the same transaction can be represented in a deserialized format like JSON. Here is the JSON representation of the transaction we just saw above.


```json
{
    "txid": "c1b4e695098210a31fe02abffe9005cffc051bbe86ff33e173155bcbdc5821e3",
    "hash": "c1b4e695098210a31fe02abffe9005cffc051bbe86ff33e173155bcbdc5821e3",
    "version": 1,
    "size": 191,
    "vsize": 191,
    "weight": 764,
    "locktime": 0,
    "vin": [
        {
            "txid": "fc9e4f9c334d55c1dc535bd691a1c159b0f7314c54745522257a905e18a56779",
            "vout": 1,
            "scriptSig": {
                "asm": "304402206a2eb16b7b92051d0fa38c133e67684ed064effada1d7f925c842da401d4f22702201f196b10e6e4b4a9fff948e5c5d71ec5da53e90529c8dbd122bff2b1d21dc8a9[ALL] 039b7bcd0824b9a9164f7ba098408e63e5b7e3cf90835cceb19868f54f8961a825",
                "hex": "47304402206a2eb16b7b92051d0fa38c133e67684ed064effada1d7f925c842da401d4f22702201f196b10e6e4b4a9fff948e5c5d71ec5da53e90529c8dbd122bff2b1d21dc8a90121039b7bcd0824b9a9164f7ba098408e63e5b7e3cf90835cceb19868f54f8961a825"
            },
            "sequence": 4294967295
        }
    ],
    "vout": [
        {
            "value": 0.02207563,
            "n": 0,
            "scriptPubKey": {
                "asm": "OP_DUP OP_HASH160 db4d1141d0048b1ed15839d0b7a4c488cd368b0e OP_EQUALVERIFY OP_CHECKSIG",
                "hex": "76a914db4d1141d0048b1ed15839d0b7a4c488cd368b0e88ac",
                "address": "1LzZJkQfz9ahY2SfetBHLcwyWmQRE9CwfU",
                "type": "pubkeyhash"
            }
        }
    ]
}
```

Though the two representations represent the same transaction, you might have noticed that you don't find inputs and outputs in this representation (at least they are not named like that). This is because they are represented by the `vin` and `vout` fields. This particular representation also has the hash of the transaction represented by the `txid` and `hash` fields. Now that we understand the transaction data structure, let's look at transaction malleability.

### What is Transaction Malleability

Transaction malleability is the ability to modify certain parts of a Bitcoin transaction that has been broadcasted to the network. When this happens, the transaction ID (TxId) of the transaction changes because the transaction ID is simply the hash of the transaction. Does signing a Bitcoin transaction not make it tamper-proof? Signatures in Bitcoin only protect the integrity of the inputs, outputs and other data used in a transaction but not the signature. This is because trying to sign the signature will create a circular dependency. Circular dependency because you need to sign the signature during the signing process but this signature is generated as a product of the signing process.  With this in mind, there are ways of modifying the ScriptSig field of the transaction thereby changing the TxId without invalidating the transaction. In the next section, we are going to be exploring different mechanisms to do that. 

### How can a Bitcoin Transaction be malleated?

The potential sources of Bitcoin transaction malleability described in [BIP 62 ](https://github.com/bitcoin/bips/blob/master/bip-0062.mediawiki) can be grouped into three categories. 

##### From Signature Format

Bitcoin Core uses OpenSSL to validate signatures and OpenSSL accepts more serializations than just those encoded using the DER standard (this is the encoding used by Bitcoin signatures). This could be a source of malleability as the serialization encoding of the signature could be changed thereby changing the hash of the transaction.

##### From ECDSA

Elliptic curve signatures are made up of two numbers, (r, s) where r is the random part of the signature and s is the part generated from the private key. Once you have these two numbers, it is possible to generate a complimentary signature denoted (r, -s mod n) where n is the order of the elliptic curve without knowing the corresponding private key of the signature. This complimentary signature is still a valid signature for the transaction. 

In addition to complementary signatures, someone with access to the private key can generate many valid signatures as the elliptic curve digital signatures algorithm uses a random number to create signatures.

##### From ScriptSig

There is the possibility of adding operations to the ScriptSig field that do not change the outcome of executing the script. For example, we can have a ScriptSig that looks like the following.

```
<signature> <pubkey> OP_DUB OP_DROP
```

Adding OP_DUB, duplicates the topmost item on the stack and OP_DROP then removes it. The addition of these two operators will change the hash of the transaction, therefore, malleating it.

Last but not least it is possible to zero pad numbers of bytes specified by push data operations(OP_0, OP_PUSHDATA1, OP_PUSHDATA2, OP_PUSHDATA4). This doesn't change the number of bytes that will be pushed to the stack. For example, reading `32` bytes is the same as reading `0032` bytes to the stack.

### Why is Transaction Malleability an issue?

Transaction Malleability is an issue because a malicious user could take advantage of it to defraud another Bitcoin user. Let's say a Bitcoin user is trying to pay a malicious merchant in bitcoins. The user can broadcast the transaction to the network. The merchant will then monitor the network for this transaction and once he/she gets it, they will then malleate it and rebroadcast it to the network. Now the race is on for which transaction will get mined first. If the malleated transaction gets mined first, it invalidates the user's transaction. Since the user's wallet will not see the transaction on the chain, after a while it might decide to redo the transaction thereby double paying the malicious merchant. One important thing to note here is that if the user's wallet double pays the merchant, it is because it is not tracking transactions correctly. It might only be looking at the transaction Id and not the UTXOs assigned to that user.

An attacker cannot change inputs and outputs of a transaction using malleability as those parts are signed. 

If not fixed, transaction malleability will have also been an issue for the Lightning network as nodes on Lightning track the confirmations on a 2-of-2 multisig transaction (funding transaction) using the transaction Id to declare a channel open.

This attack was claimed to have been used against many Bitcoin exchanges back in 2014. Most notably Mt.Gox, which was the largest Bitcoin exchange at the time claimed to have been rendered insolvent because of this attack. Though the attack was real in 2014, this [paper ](https://arxiv.org/abs/1403.6676) proves that there was no widespread use of this attack before the closing of Mt.Gox in February 2014. 

### Conclusion

Most vulnerabilities were removed from the Bitcoin system with BIP 62 and SEGWIT eventually made this impossible. This might be the subject of a future post. In this post, we reviewed the transaction data structure and saw how parts of it can be modified to malleate the transaction hash (TxId). Last but not the least, we saw how an attacker can use this to get double paid and how this could have been an issue for the Lightning network implementation. 

### Resources

[Bitcoin Transaction Malleability and MtGox](https://arxiv.org/abs/1403.6676)
[Transaction Data](https://learnmeabitcoin.com/technical/transaction-data)
[Transaction Signing](https://learnmeabitcoin.com/technical/ecdsa#signing-a-transaction)
[BIP 62](https://github.com/bitcoin/bips/blob/master/bip-0062.mediawiki)
[Bitcoin Transaction Malleability](https://eklitzke.org/bitcoin-transaction-malleability)
Kalle Rosenbaum - Grokking Bitcoin, Pg 334-336

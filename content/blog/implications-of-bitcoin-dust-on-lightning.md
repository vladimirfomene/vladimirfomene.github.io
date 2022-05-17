+++
title = "Implications of Bitcoin Dust Limit on Lightning Channels"
date = 2022-05-03
+++

### Introduction

Welcome back to the blog! This week, I'm taking you on a debugging journey where we will be learning about Bitcoin dust, how it is calculated and its implication for the Lightning network. Before we dive into the meat of the subject, let me set the stage. This week while trying to open a payment channel from an [LDK](https://lightningdevkit.org/) node to an [LND](https://github.com/lightningnetwork/lnd) node setup in [Polar](https://lightningpolar.com/), I had the following error:

```sh
Channel <channelID> closed due to processingError
{ err: "dust_limit_satoshis (573) is greater than the implementation limit (546)" }

```

On seeing this error, I was intrigued. What is the meaning of dust? What does the `dust_limit_satoshis` represent in Lightning? Why is the dust_limit_satoshis `573` and why is the implementation limit `546`. How are these calculated?

### What is the dust and what does it represent?

In Bitcoin, a UTXO is considered dust when the fee required to spend it is greater than the amount on the UTXO. Bitcoin Core considers a UTXO as dust if it's value is lower than the cost of spending it at the [dustrelayfee](https://github.com/bitcoin/bitcoin/commit/9022aa3). In Bitcoin Core, this `dustrelayfee` can be set by a commandline argument `-dustrelayfee`. Additionally, Bitcoin Core default `dustrelayfee` is 3000 sat/kB. For a P2PKH transaction, the dust limit is calculated as follows:

A P2PKH output is 34 bytes:

- 8 bytes for the output amount
- 1 byte for the script length
- 25 bytes for the script (`OP_DUP` `OP_HASH160` `20` 20-bytes `OP_EQUALVERIFY` `OP_CHECKSIG`). Each opcode is 1 byte in size.

A P2PKH input is at least 148 bytes:

- 36 bytes for the previous output (32 bytes hash + 4 bytes index)
- 4 bytes for the sequence
- 1 byte for the script sig length
- 107 bytes for the script sig:
  - 1 byte for the items count
  - 1 byte for the signature length
  - 71 bytes for the signature
  - 1 byte for the public key length
  - 33 bytes for the public key

The P2PKH dust limit is then `(34 + 148) * 3000 / 1000 = 546 satoshis`. This is just the  `(input-size + output-size) * fee-per-byte` .

Here are the dust limit for the other transaction types in Core:
- pay to script hash (P2SH): 540 satoshis
- pay to witness pubkey hash (P2WPKH): 294 satoshis
- pay to witness script hash (P2WSH): 330 satoshis
- unknown segwit versions: 354 satoshis

Does this mean dust is unspendable? Not really, it can be spent with other input(s) provided this other input(s) can provide value to make the transaction fee greater than the minimum relay fee. In case you are interested in reading more about how these calculations are made, you can read more [here](https://github.com/lightning/bolts/blob/master/03-transactions.md)



### Why is it set at 546 satoshis for the LDK node?

I tracked down the error message above in LDK's [source code](https://github.com/lightningdevkit/rust-lightning/blob/main/lightning/src/ln/channel.rs#L1961)  and this error comes from a method called `accept_channel`. In this function, there is condition that requires the dust limit to be below  `MAX_CHAN_DUST_LIMIT_SATOSHIS` set to `546` in LDK. This makes sense because it is the largest dust limit from the previous section on dust limit calculation in Bitcoin Core. 

Where is the `573` coming from you? In Lightning, channel establement between two nodes is a six step process as shown below

       +-------+                              +-------+
        |       |--(1)---  open_channel  ----->|       |
        |       |<-(2)--  accept_channel  -----|       |
        |       |                              |       |
        |   A   |--(3)--  funding_created  --->|   B   |
        |       |<-(4)--  funding_signed  -----|       |
        |       |                              |       |
        |       |--(5)--- funding_locked  ---->|       |
        |       |<-(6)--- funding_locked  -----|       |
        +-------+                              +-------+

        - where node A is 'funder' and node B is 'fundee'
		
[source](https://github.com/lightning/bolts/blob/master/02-peer-protocol.md)

Following this flow, when I executed the `openchannel` command on the CLI of the LDK node, it sent an `open_channel` message to the LND node. As soon as LND node received the `open_channel` message, it processed it and replied with an `accept_channel` message containing a `dust_satoshis_limit` value of `573`.

### Why is it set at 573 satoshis for the LND node?

According to [Bitcoin Core v0.10 release candidate 3](https://github.com/bitcoin/bitcoin/blob/v0.10.0rc3/src/primitives/transaction.h#L137), an output is considered dust if you will pay 1/3 of its values in fees. Assuming we want to construct a transaction to spend this dust, a typical output is 34 bytes in size and will require an input of 148 bytes to be spend it. According to the comment in Bitcoin Core, the dust limit is 546 satoshis. How did they get that? If we consider a one input and one output transaction, which will be the transaction with the least weight necessary to spend an output, the size of the transaction is `148 + 34 = 182` At a 1 satoshi/byte rate, this transaction will cost 182 sats. What is the fee of the output for which we will pay 182 sats to spend it? 

`182 * 3 = 546 sats`


 According this [Stack Exchange](https://bitcoin.stackexchange.com/questions/1195/how-to-calculate-transaction-size-before-sending-legacy-non-segwit-p2pkh-p2sh), there is always a 10 bytes extra bytes added to the transaction and there is also a "plus or minus 1" byte added for the signature. Given this information, we adjust our calculation of the size of a one input, one output transaction as follows:

`148 + 34 + 10 = 192 bytes ` 

Following the "plus or minus 1" rule we choose to substract one byte and we get `191` bytes. At a 1 satoshi/byte rate, this transaction will cost 192 sats. What is the fee of the output for which we will pay 192 sats to spend it?

`192 * 3 = 573 sats`


This is how LNDv0.12.1-beta's `dust_satoshis_limit` was calculated and set to `573`.

### Final analysis of the bug
The LDK node refused to open a channel with the LND node because LDK's dust limit is 546 and LND's dust limit is 573. As such, if a payment of 550 sats (above the dust limit) is made from LDK, LND will consider it as dust (below the dust limit) and will apply the policy for payments below dust in Lightning, whereas the LDK node is not expecting that payment to be treated as below dust. Therefore having different dust limit values will cause a misalignement on how the payment should be processed.


### What are the implications the dust limit on Lightning channels?

The Lightning protocol does not create an HTLC output for payments below the `dust_satoshis_limit`. To understand what happens to these payments let's look at an example. Let's say Nana and Ama have a payment channel with each other and Nana wants to route a payment below the  `dust_satoshis_limit` through Ama. Assumming the current state of the channel has 10000 sats on Nana's side and 6000 sats on Ama's side.

![Initial Channel State](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/43qhjuv307fn0d1zhyz4.png)

When Nana routes a payment of 200 sats through Ama, the commitment transaction of the channel will be as follow:

![Payment Under Dust limit](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tahzq97aozxn6wmz17eg.png)

From the diagram, you can see that no HTLC output was created for this payment. On the otherhand, Nana's balance has been deducted by 200 sats and that value has been implicitly added to the fee of this transaction. Once Ama gets the preimage for this payment, she will give it to Nana, and they will transition commitment transaction to a state where the 200 sats has been added to Ama's output. 

![Commitment Transaction After Ama receives preimage](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dsyc7e7f8hpzrqwbnidb.png)

The issue with this process is that it requires Nana to trust Ama after putting his 200 sats as transaction fee, but what if Ama was a bad actor. She could decide to forcefully close the channel and Nana will loose his money to a miner on onchain fees. In other words, the protocol does not protect Nana when a payment is below the `dust_satoshis_limit`.

What are the implications? Well this means, channel partners cannot carry out micropayments below the dust limit in a trustless way. Therefore this puts a security risk on anyone paying or routing a payment under the dust limit.


### Conclusion

In this article, we explored the bug I had while trying to open a channel from an LDK node to an LND node. This bug made us explore an important concept in Bitcoin and Lightning called dust limit. We also saw how payments under the dust limit don't create HTLC output as transaction with outputs under the dust limit are not viable on-chain as they will not be relayed by nodes. 

### Resources

[Lightning Peer Protocol Bolt](https://github.com/lightning/bolts/blob/master/02-peer-protocol.md)
[Transaction Bolt](https://github.com/lightning/bolts/blob/master/03-transactions.md)
[Rene Pickhardt's Bitcoin StackExchange](https://bitcoin.stackexchange.com/questions/87735/how-can-i-create-htlcs-below-dust-limit-satoshi-in-lightning-network)
[David Harding's Bitcoin StackExchange](https://bitcoin.stackexchange.com/questions/85650/htlcs-dont-work-for-micropayments)
[Lightning dirty little secrets](https://medium.com/@peter_r/visualizing-htlcs-and-the-lightning-networks-dirty-little-secret-cb9b5773a0)
[What is Bitcoin's dust limit, precisely?](https://www.reddit.com/r/Bitcoin/comments/2unzen/what_is_bitcoins_dust_limit_precisely/)
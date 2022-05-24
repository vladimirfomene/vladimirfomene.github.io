+++
title = "Coinjoin"
date = 2022-05-16
description = "Coinjoin is a protocol or mechanism for increasing the privacy of Bitcoin users by mixing inputs from multiple parties in a transaction"
[extra]
    image = "https://res.cloudinary.com/vladimirfomene/image/upload/v1653223611/blog/coinjoin.jpg"
    url = "https://www.vladimirfomene.com/blog/coinjoin/"
+++

## Introduction

Despite being public, the Bitcoin blockchain provides tools for users to transact while maintaining their privacy. This article will look at how the Coinjoin protocol can help users transact more privately on the Bitcoin network today. The inputs in a Bitcoin transaction usually come from one user but there is no restriction on the protocol which stipulates that it has to be so. Chain analysis tools utilize this for tracking transactions under the name of the common input ownership heuristics. This just means that when a chain analysis tool sees a transaction with many inputs, it assumes those inputs are coming from the same Bitcoin user. To break this heuristic, Coinjoin utilizes the fact there is no restriction on Bitcoin transaction inputs stipulating that they have to come from the same owner. With the stage set, let's look more into what is Coinjoin.

## What is CoinJoin?

Coinjoin is a protocol or mechanism for increasing the privacy of Bitcoin users by mixing inputs from multiple parties in a transaction. Coinjoin uses the fact that Bitcoin does not restrict ownership of inputs in a transaction to one user to create a transaction which is made up of inputs from many participants to make it hard for any third party to map an input to a particular output. Why does this increase the privacy of Bitcoin transactions? It does because it is now difficult for a chain analyst to distinguish between a transaction spending UTXOs tied to multiple addresses owned by one user to a Coinjoin transaction. In addition, Coinjoin is also resistant to transaction graph analysis as it makes it hard for someone to map any of the inputs to a particular output. This makes it hard for chain analysis to track the flow of UTXOs from one party to another.



## Coinjoin Considerations

If your order of outputs is the same as your order of inputs, it will still be leaking information about your input/output mapping. Also if your anonymity set (the set of participants in the Coinjoin transaction) is too small it will make it easy for a third party to figure out the input/output mapping of the transaction. Like a Coinjoin of two participants. Increasing the anonymity set in Coinjoin will increase the inputs of the transaction. Therefore, the anonymity set is limited by the transaction size of the bitcoin network. 



## What are the different implementation mechanisms?

I made up the following classifications to better understand various implementations of Coinjoin. 

## Manual Naive Implementation 

Here the transaction is being created by trusted parties over an online communication channel that they agree on. Participants could also meet in person. This communication channel could be a forum, an online group chat or any other meeting app on the internet. Over this medium, participants collaborate by sharing their inputs and outputs to facilitate the process of transaction creation. They can create a transaction template using a partially signed Bitcoin transaction. Each party then verifies that his/her inputs and outputs are correct in the final transaction before signing. Then one of the participants can broadcast this transaction to the Bitcoin network. Though I haven't mentioned it here, these participants could collaborate by relying on one of the participants as the coordinator or they could all act as peers in a decentralized protocol where there is no leader.

## DrawBacks

Since the transaction is between trusted parties, every participant will learn the input-output mapping of every other participant. Therefore one of the participants could leak the privacy of other participants. Given that the transaction has to be done between trusted parties, it might not be easy to find parties willing to participate with you in a Coinjoin transaction. If participants are meeting over a third-party service like a forum, this service will keep a log of their activities. This implementation also assumes that participants or one of the participants will have the necessary technical knowledge for creating, signing and broadcasting the transaction.

## Automated Naive Implementation

In this implementation, participants meet over a server who is in charge of coordinating the transaction creation, signing and broadcasting between participants. This server is often called the coordinating server or the coordinator. This server could be run by a service or one of the participants in the Coinjoin transaction. If the coordinating server is a service it might be easier to get a bigger anonymity set if you are participating in the transaction

## DrawBacks

The coordinating server knows the input and output mapping of every participant. This is a risk because the server could leak this information to parties external to the transaction. In addition to learning the transaction information, the server also learns the IP address of participants. Participants can join via Tor but that is assuming that the service offers a possibility to connect via Tor. In terms of service reliability, the coordinating server is a single point of failure.

## Decentralised Implementation: With CoinShuffle

Implementing Coinjoin with CoinShuffle removes the need for a central coordinator to facilitate the transaction. Coinshuffle happens in three steps. At the beginning of the protocol execution, each participant generates a public encryption scheme key pair. They will then announce their public keys alongside their private keys to all other participants in the group. The next step is to share the outputs without participants learning the input/output mapping for each participant. This is done through shuffling. To understand shuffling let's assume that we have A, B, C, and D participating in a Coinjoin transaction and they have already passed the key exchange step.  

A will then perform layered encryption of her outputs. Assuming their output is A', they will first encrypt their output for D, this can be represented as `enc(Ekd, A')`  where `Ekd` is the public encryption key of D. This means this ciphertext can only be decoded by D. They will then encrypt again for C, `enc(Ekc, enc(Ekd, A'))`. Finally, they encrypt for B, `enc(Ekb, enc(Ekc, enc(Ekd, A')))` and share this with B.

B takes this cypher and decodes it to `enc(Ekc, enc(Ekd, A'))`. Following the same pattern as A, B will encrypt their output to be consumed for D and C. This will produce, `enc(Ekc, enc(Ekd, B'))`. B nows shuffle the list of cyphers and send it to C. 

Once C receives these cyphers, they decode them and get, `enc(Ekd, A')` and `enc(Ekd, B')`. C will also encrypt their output for D, `enc(Ekd, D')`, shuffle these ciphertexts, and send it to D.

On receiving the ciphertext from C, D will decrypt it and get all the outputs. They will add their output to it and shuffle it. They will then proceed to share all these outputs (D', B', C', A') with other participants. Shuffling makes it hard for a participant to be able to map output to another participant.

In the final step, participants will create a transaction and each will verify their input/output in the transaction before signing.

One interesting thing about Coinshuffle is that if a participant’s address gets lost in this process (i.e doesn’t make it to one of the participants), the protocol can know who cheated in the process. The cheating party will be removed from the process and they will start all over again. This protocol can be implemented over a central communication server or in a peer-to-peer fashion. 

## Drawbacks

This implementation needs at least two honest participants to work. Let's assume you only have one honest participant, the dishonest participants could easily collaborate to de-anonymize the output of the honest participant.

## Chaumian Coinjoin

In Chaumian Coinjoin, you don’t get rid of the central coordinator, you just use blind signatures and a secured communication medium like Tor to ensure that the coordinator doesn’t get a knowledge of the input-output mapping. 

![Chaumian Coinjoin](https://res.cloudinary.com/vladimirfomene/image/upload/v1652887301/blog/chaumian.png)


From the diagram, Nana connects to the server and provides their inputs and blinded outputs. The server signs the outputs and gives a blind signature to Nana. Nana then disconnects from the server and connects again as a different user called Ama. This time he gives the signature and the unblinded output to the server. The server has no way of knowing that they had connected before as Nana but the server can use the signature to verify if they had seen these outputs before. Every participant in the Coinjoin transaction follows the same procedure as Nana to submit their inputs and outputs to the server. Once the server has all the input and output, it constructs a transaction and sends it to all participants for verification. If verification fails for any of the participants, the server will have to start the process all over again. The transaction is then broadcasted to the network by the server. Chaumian Coinjoin is implemented in the Wasabi wallet.

## Drawbacks

Participants will be charged a fee for the coordinating service provided by the central coordinator. Users who want to use Chaumian Coinjoin may need Tor knowledge. Last but not least, finding participants and coordinating for the Coinjoin transaction may take some time.

## Benefits of Coinjoin

Coinjoin transactions provide more privacy for all Bitcoin transactions because aside from protecting the privacy of the users participating in a transaction they also make it hard to distinguish a transaction with inputs from different addresses controlled by the same user to Coinjoin transaction. On the other hand, for a user participating in a Coinjoin transaction, the fee is cheaper as it is shared across multiple participants. Coinjoin transactions provide less pressure to the Bitcoin network as users can send one transaction instead of multiple transactions. Last but not the least, Coinjoin doesn’t require any change in the existing Bitcoin system to work.

## Cons of Coinjoin

Coinjoin transactions can be tracked on-chain by a chain analyst. Using chain analysis, certain exchanges can track and refuse to accept Coinjoin bitcoins. There is currently no implementation of the peer-to-peer Coinjoin. 

## Alternative Privacy Approaches

## What is CoinSwap?

Coinswap is a protocol that allows two or more participants to exchange coins with each other using a set of transactions. For example, let's have a look at a coinswap transaction between Nana and Ama. 

Ama's Address 1 ----> CoinSwap Address 1 ----> Nana's Address 1
Nana's Address 2 ----> CoinSwap Address 2 ----> Ama's Address 2
The coinswap addresses here will either be a 2-of-2 multisig or a 2-of-3 multisig address. The utility of coinswap is that it breaks the transaction linkability for Ama as there is no one transaction linking here fund transfer from address 1 to address 2. Compared to Coinjoin, Coinswap
will be more expensive as the protocol needs more than one transaction for its execution. There is transaction is more costly compared to a Coinjoin transaction.

## ZeroCash

This is a code fork of the Bitcoin protocol which enables private transactions using zero-knowledge proofs. The main concern with this approach is that it adds a computation overhead to transaction validations and also uses cryptography that is fairly new and so has not yet been battle-tested in cryptographic systems. To implement ZeroCash's approach to privacy, Bitcoin will have to change through a fork which is not the case for Coinjoin.

## Confidential transactions

You might have realized that our Coinjoin transactions don't hide the value of the outputs. As such, using statistical analysis chain analysts could still be able to figure out the input/output mapping. Some Coinjoin implementations produce outputs with equal value to fight this privacy leak. This mechanism makes it even easier to easily identify Coinjoin transactions. Confidential transactions is a proposal to use a type of zero-knowledge proof called bulletproofs to encrypt the output values of a transaction while at the same making sure users don't spend more on outputs than they have in inputs. Confidential transactions need a change in Bitcoin in order to be possible.

## Conclusion

Despite the fact that the Bitcoin protocol lacks a direct method for constructing private transactions, it is possible to use the current transaction structure to create transactions that safeguard user privacy while keeping the protocol intact. Despite the fact that there have been many different privacy protocols proposed, Coinjoin has seen a lot of development and wallet integration since it is very easy to deploy because it does not require a change to the Bitcoin protocol.

## References

[Greg Maxwell's Coinjoin BitcoinTalk](https://bitcointalk.org/index.php?topic=279249.0)
[Coinshuffle Bitcoin talk](https://bitcointalk.org/index.php?topic=567625.0)
[Chaumian Coinjoin](https://www.youtube.com/watch?v=yJFCRNCa6s0)
[Confidential Transaction](https://www.youtube.com/watch?v=EDaM8A-tAck)
[Coinswap Bitcoin Optech](https://bitcoinops.org/en/topics/coinswap/#)
[Coinjoin Bitcoin Optech](https://bitcoinops.org/en/topics/coinjoin/)

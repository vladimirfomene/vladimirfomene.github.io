+++
title = "Coinjoin"
date = 2022-05-16
+++

## Introduction

The inputs in a Bitcoin transaction usually come from one user but there is no restriction on the protocol which stipulates that it has to be so. Chain analysis tools utilize this for tracking user's transaction under the name of the common input ownership heuristics. This just means that when a chain analysis tool sees a transaction with many input, it assumes those input are coming from thesame Bitcoin user. To break this heuristic, Coinjoin utilizes the fact there is no restriction on Bitcoin transaction inputs stipulating that they have to come from the same owner. With the stage set, let's look more into what is Coinjoin.

## What is CoinJoin?

Coinjoin is a protocol or mechanism for increasing the privacy of Bitcoin users by mixing inputs from multiple parties in a transaction. Coinjoin uses the fact that Bitcoin does not restrict inputs in a transaction to come from one user to create a transaction which is made up of inputs from many participants with the goal of making it hard for any third party to map an input to a particular output. Why does this increase the privacy of Bitcoin transactions? This is because now it is difficult for a chain analyst to distinguish between a transaction spending UTXOs tied to multiple addresses owned by one user to a Coinjoin transaction. In addition, coinjoin is also resistant to transaction graph analysis as it makes it hard for someone to map any of the inputs to a particular output. This makes it hard for chain analysis to track the flow of UTXOs from one party to another.

## Benefits of Coinjoin

Coinjoin transactions provide more privacy for all Bitcoin transactions because aside from protecting the privacy of the users participating in a transaction they also make it hard to discussion a transaction with inputs from different addresses controlled by thesame user to coinjoin transaction. On the otherhand, for a user participating in a coinjoin transaction the fee is cheaper as it is shared across multiple participants. Coinjoin transactions provide less pressure to the Bitcoin network as users can send one transaction instead of multiple transactions. Last but not the least, Coinjoin doesn’t require any change in the existing Bitcoin system to work. 



## Coinjoin Considerations
If your order of outputs is the same as your order of inputs, it will still be revealing your input/output mapping.
Also if your anonymity set is too small it will make it easy for a third party to figure out the input/output mapping of the transaction. Like a coinjoin of two participants.The anonymity set is limited by the transaction size of the bitcoin network.
Coinjoin transactions are easily detectable on the blockchain by blockchain analysts. 



## What are the different implementation mechanisms? 

## Manual Naive Implementation 
Between trusted party over online communication channel
Where participants could connect over an online forum and create a transaction template which is shared among themselves and each participant will add their respective inputs and outputs and the transaction will be signed by each participant and a selected participant can broadcast it over the Bitcoin network. 

#### DrawBacks
Every participant knows the mapping between input and outputs. 

## Automated Naive Implementation 
Participants meet over a coordinating server. In this particular scenario, you have a server who acts as coordinator for facilitating the Coinjoin transaction between these participants. 

#### DrawBacks
The coordinating server has knowledge of the input and output mapping of every participant.
In term of service reliability, the coordinating server is a single point of failure.

## Decentralised Implementation: With CoinShuffle 
Implementing Coinjoin with CoinShuffle removes the centralised server used for coordination. In coinshuffle the outputs are gotten to each participant through a process of key exchange, encryption and shuffling that makes it hard for any participant to tell from whom a particular output is coming from. (Maybe we can show the process with a map)
One interesting thing about Coinshuffle is that if a participant’s address gets lost in this process (i.e doesn’t make it to one of the participants), it is possible for the protocol to know who cheated in the process. The cheating party will be removed from the process and they will start all over again. This protocol can be implemented over a central communication server or in a peer-to-peer fashion. 

### Drawbacks

Chaumian Coinjoin: Smart Implementation with a central meeting server
In Chaumian Coinjoin, you don’t get rid of the central coordinator, you just use blind signatures and a secured communication medium like Tor to ensure that the coordinator doesn’t get a knowledge of the input output mapping. 



Chaumian coinjoin is implemented in the Wasabi wallet. 
To prevent that coordinator from learning which inputs fund which outputs, users anonymously commit to the outputs they want to create, receiving a chaumian blinded signature over the commitment. Later, each user connects under another anonymous identity and submits each output along with its unblinded signature. The signature provably came from the coordinator but the unblinded signature can’t be connected to the specific user who received the blinded signature. This allows constructing the transaction template without the coordinator learning which inputs funded which outputs.



What are the trade-offs and risks? 
In the naive implementations, the input and output mapping is known by either the central meeting server or the other participants in the transactions.
Coinjoin also has a negative side effects for participants as they are not accepted by many exchanges or other platforms requiring KYC.
These services might be leaking user’s Ip address, and so it is recommended to be used over a privacy network like Tor. Tor doesn’t provide the best experience for most users.

## Cons of Coinjoin
CoinJoin has the potential to taint our Bitcoin with other Bitcoin derived from criminal acts. Here is a value, freedom, and moral judgement on the part of each user, with the caveat that if you join CoinJoin, people who have obtained that bitcoin unlawfully may join you.
It is possible that user data will be exposed. This is achievable if you link your CoinJoin managed balances to other managed balances associated with your identity. Even if you use CoinJoin, it is conceivable at this stage to conduct a data analysis that will identify your identity.

## Alternative Privacy Approaches

## What is CoinSwap?
Coinswap is a protocol that allows two or more participants to exchange coins with each other using a set of transactions. For example, let's have a look at a coinswap transaction between Nana and Ama. 

Ama's Address 1 ----> CoinSwap Address 1 ----> Nana's Address 1
Nana's Address 2 ----> CoinSwap Address 2 ----> Ama's Address 2
The coinswap addresses here will either be a 2-of-2 multisig or a 2-of-3 multisig address. The utility of coinswap is that it really breaks the transaction linkability for Ama as there is no one transaction linking here fund transfer from address 1 to address 2.

## How is this different from Coinjoin?
Unlike Coinjoin which uses a single transaction, coinswap uses more than one transaction. 
Coinswaps are also arguably hard to detect by a blockchain analyst on the blockchain compared to Coinjoins. 
Last but not the least, Coinswap transactions could be performed across blockchains in the form of atomic swaps.
Drawbacks
This will be potentially more expensive for a user as they might have to pay fees for more than one transaction.



Zerocoin/ZeroCash
Confidential transactions - uses zero knowledge proof, 

## Conclusion

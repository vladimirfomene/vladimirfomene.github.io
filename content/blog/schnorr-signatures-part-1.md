+++
title = "The ABCs of Schnorr Signatures: Part 1"
description = "Welcome back to the blog! This week, we are going to be learning about Schnorr Signatures, we will start by explaining its building blocks such as hash functions, commitment scheme and challenge/response scheme. Then, we will build the Schnorr identity protocol and derive the signature scheme from it."
date = 2022-05-26
[extra]
	image = "https://res.cloudinary.com/vladimirfomene/image/upload/c_scale,w_1200/v1653570051/blog/open-source.jpg" 
	url =  "https://www.vladimirfomene.com/blog/contributing-to-bitcoin-open-source/"
+++

## Introduction

Welcome back to the blog! This week, we are going to be learning about Schnorr Signatures. We will start by explaining its building blocks such as hash functions, commitment scheme and challenge/response scheme. Then, we will build the Schnorr identity protocol and derive the signature scheme from it.

## Motivation

Most online platforms use passwords for authentication, despite their security weaknesses. The most important of which is allowing an online platform to learn about your secret aka password. How do we prove our identity online without necessary revealing our password? To solve this problem, cryptographers created a branch of cryptography in the 1970s called the public key cryptography. In public key cryptography, you have a key pair, one private and one public. You can share the public key with anyone but your private key should be kept secret almost like your password. This new branch of cryptography also gave rise to digital signature algorithms (**DSA**). A DSA is proof of knowledge of a private key for a corresponding public key. In an online interaction, one user will produce a DSA to show the other party they know the private key associated with their public key. There are different signature schemes, for example, [RSA](https://cryptobook.nakov.com/digital-signatures/rsa-signatures), [ECDSA](https://learnmeabitcoin.com/technical/ecdsa) and Schnorr signatures. Now that you know the motivations behind DSAs, let's start discussing the building blocks for building the Schnorr signature scheme.

## Building Blocks for Schnorr Signatures

### Commitment Schemes

While building DSAs, we need a mechanism for a party in interaction to commit to a value without revealing it to the other party in the interaction. This requirement is also referred to as the  **hiding** property of the interaction. The second property we want from this interaction is for it to be **binding**. That is, if a party commit to a particular value and later changes it, the other party in the interaction should be able to prove that the value changed.  Cryptographic protocols usually use hash functions for commitments schemes because, given the hash of a message, you can't tell what the message was. This is because hash functions are one-way functions. That is, given x, you can compute `h(x)` but it is impractical to compute x given `h(x)`. This gives hash functions a `hiding` property. Given a hash, if the message was to change, the hash will also change. Therefore the hash function is `binding`.  It is also possible to commit to a value by multiplying it with the generator point of an elliptic curve. In Bitcoin, we commit to the private key by multiplying it with the generator point like so:

$$
P = pG 
$$
where p is the private key and P is the public key.

This has a hiding property because it is computationally impractical to derive the private key from the public key and it also has the binding property because changing the private key will change the value you get as the public key. 


### Challenge / Response Strategy

Consider an online interaction between Alice and Bob, where Alice is the prover (i.e Alice has to prove that she knows the private key associated to some public key) and Bob, is the verifier (Bob has to verify Alice's claim). Alice has to prove that she knows the private key, `x` associated with her public key, `P` without revealing the private key to Bob. To aid Alice, Bob will generate a random number, `c` and share it with Alice, and challenge Alice to produce a number `s` such that:

$$
s * G = c * P
$$

Alice will then generate `s` by multiplying Bob's random number, `c` with her private key, `x`.  So we have:

$$
s = x * c
$$

With this, if Bob multiplies the right-hand side of the equation with `G`, the elliptic curve's generation point, he will get `x * G * C` which is just `P * c`. Since multiplication is commutative, this is also equal to `c * P`. This solves it right? Alice has been able to prove to Bob that she knows the private key associated with `P` her public key. Well, not really, with this solution, Bob can easily calculate Alice's private key by computing `s / c` where the division symbol represents the modulus inverse. Therefore this solution is binding but it is not hiding.

### Blinding Approach

Given that the above approach does not succeed in hiding Alice's private key. She decides to blind her private key, `x` adding it to a large randomly generated number, `r`. With this in place, `s` can now be written as:

$$
s = x + r
$$

Alice commits to `r` by multiplying it with the generator point, `G` of the elliptic curve. This gives us:

$$
R = r * G
$$

The pair `(R, s)` becomes the proof Alice gives to Bob. To verify this proof, Bob will have to prove the equality of the following equation given `s`, `R` and `P`.

$$
s * G = P + R
$$

It turns out this second approach also has its loophole, an attacker can pick  `s` to be anything and  `R = G * s - P`. If you substitute `R` in the verification equation above, the `P`'s will cancel out and the equation will pass verification. 

### Combined Approach

To solve the shortcomings of each approach, we will combine both approaches. Here Alice chooses a random number, `r` and commits to it by multiplying it with the generator point of the elliptic curve, `R = G * r`. Alice then shares `R` with Bob who will generate a challenge, `c` and share it with Alice. Alice will then create the following `s`:

$$
s = x * c + r
$$

When Bob receives, `s` from Alice, he has to verify the equality of the right-hand side and the left-hand side of the following equation:

$$
s * G = P * c + R
$$

With this Alice has successfully proven to Bob that she knows the private key associated with `P` securely.

The protocol explained above is the Schnorr Identity Protocol not not the Schnorr signature scheme. We have shown how Alice could generate proof of knowledge for a secret (private key) without revealing the secret to Bob. It might also be important to note that the above protocol was interactive (both parties have to be online for this to work).

To make the process non-interactive, we will replace our challenge with the following hash:

 $$
 H(R||P||m)
 $$ 

 This process is known as the **Fiat-Shamir transform**. Once we have applied the Fiat-Shamir transform, we now have the **Schnorr signature scheme**.

## Conclusion

In this article, we were able to build the Schnorr signature scheme starting from commitments and the challenge/response scheme to form the Schnorr identity protocol and finally used the Fiat-Shamir transform to derive the Schnorr signature scheme from the Schnorr identity protocol. While discussing we also proved that the protocol is sound, complete and provable.

## References

While working on this article, I learned a lot from the following resources:

* [Introduction to Schnorr Signatures with Elichai Turkel](https://www.youtube.com/watch?v=XKatSGCZ-gE)
* [How to design Schnorr Signatures with Adam Gibson](https://www.youtube.com/watch?v=wjACBRJDfxc)
* [CryptoShorts Schnorr Signatures](https://www.youtube.com/watch?v=r9hJiDrtukI)
* [Schnorr Signature BIP340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki)
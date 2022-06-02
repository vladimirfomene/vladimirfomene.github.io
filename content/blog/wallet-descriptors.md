+++
title = "Wallet Descriptors"
date = 2022-06-01
description = "Wallet descriptors or output script descriptor are plain-text language that helps wallet move from public/private keys to scripts and addresses"
[extra]
    image = "https://res.cloudinary.com/vladimirfomene/image/upload/c_scale,w_1039/v1653223416/blog/malleability.jpg"
    url = "https://www.vladimirfomene.com/blog/wallet-descriptors/"
+++

Welcome back to the blog! This week we are going to be looking at the issues that led to the creation of wallet descriptors, look at their syntax and talk about their implications for wallets.

## Introduction

To the everyday Bitcoin user, wallets seem to be working fairly well and can easily be backed up thanks to BIP39 mnemonic phrases. Though the tree of keys wallet or the wallet generated from the mnemonic phrase often called the legacy wallet works well, it has a couple of shortcomings. Due to these shortcomings, BIP380 was proposed to resolve some of the issues faced by
the current wallet architecture.

## Why  Wallet Descriptors

As mentioned in the introduction, with the old wallet we only have keys and it is not clear to the 
wallet software which type of address types (P2PKH, P2WPKH, P2WSH, etc) to generate from these keys.
As a consequence, wallet restoration is very computationally intensive, for each key, wallets have to scan the whole chain for different ScriptPubKey types to calculate a user's balance.

Although, BIP44, BIP49 and BIP84 provide standard derivation paths to be used by all wallets, not every wallet supports these BIPs. As a direct consequence, some wallets will not be able to
derive a user's balance given a BIP39 mnemonic phrase as they might be using a different derivation path from the wallet that produced the backup. This is will create an illusion of a loss of funds for a user thereby providing a very poor user experience.

Last but not the least, this current architecture also doesn't provide the mechanism for wallets to easily watch funds in a multisig. 

Last but not the least, with the advent of SEGWIT, wallets created new version bytes (version bytes are prepended to the 20 bytes of a public key to create different address types) to denote native SegWit addresses. With SEGWIT, wallets used `0x049d7cb2` for `ypub` which is an extended public key for wrapped SEGWIT addresses and `0x04b24746` for `zpub` which is an extended public key for native SegWit. Using prefixes like `xpub`, `ypub` and `zpub` are not sustainable in the long term as they are only so many letters in the alphabet.

With all these problems, we need a way to help wallet software easily generate the right addresses and scripts given a particular key, extended key and derivation path. This is where wallet descriptors come to the rescue.


## What are descriptors?

Wallet descriptors often called output descriptors or output script descriptors were proposed as a new specification (BIP380) in Bitcoin which provides a plain text language that describes how outputs scripts (ScriptPubKey) and addresses are derived from keys. Wallet descriptors also contain all the information needed to spend from a particular script if the wallet software has the private key. By information, I mean the script type (P2PKH, P2WPKH, etc) and public keys necessary to create a scriptsig/witness to spend an output. 

## Structure of wallet descriptors?

Now that we understand what descriptors are, let's look at the structure of descriptors. Here is a descriptor for an address generated in my local Bitcoin core running in regtest. 
`wpkh([3bc81a2f/84'/1'/0'/0/0]02c6577ab6080b2ebce1eb86d8ce5c8061cb029a82ae4ef9d49e1ed37e3d0d3896)#tcgk59uj`

This string has the following format: 

`function([derivation-path]key)#checksum`

* The function, in this case, is `wpkh` which means you can derive a P2WPKH address or ScriptPubKey from this descriptor.
* `[3bc81a2f/84'/1'/0'/0/0]` is the derivation path but which doesn't start with M or m but instead the first four bytes of the hash of the master key. 
* `02c6577ab6080b2ebce1eb86d8ce5c8061cb029a82ae4ef9d49e1ed37e3d0d3896` is the key in this case.
* `tcgk59uj` is the checksum which makes it easy for transcription errors to be detected when descriptors are moved from one wallet to another.

## Drawbacks of Wallet descriptors

Though descriptors are great at what they do, they will potentially make the import/export user experience for wallets not very friendly. This is because descriptors are easily readable to someone
who is technical but it looks like code to someone who is not technical. It seems much easier for an average user to remember a mnemonic than to write down an output descriptor. It is important to note that wallet descriptors can be used alongside mnemonics when it comes to backing up a wallet. For example, the mnemonic could be used to restore the HD wallet and the descriptor could be used to restore a wallet with complex scripts like multisig.  There have been talks to convert these descriptors to base64 for backup/restore or even to create a new mnemonic system that works for them.

## Wallet descriptors in Bitcoin Core

Core has had support for output descriptors since v17.0. In this section we are going to explore 
a few Bitcoin Core commands with support for output descriptors. From Bitcoin Core v23.0.0, if you create a wallet with the CLI you will get a descriptor wallet unless `descriptors=false` is set 
while using `createwallet` to create the wallet. Assuming you already have a wallet, we will start
by creating an address, I'm running v23.0.0 but any version above v17.0 should be okay.

Run the following command to create an address:
`bitcoin-cli -rpcwallet=<wallet-name> getnewaddress`

This should get you a bech32 bitcoin address. Using `getaddressinfo` let's see what Core offers us 
apart from the address.

`bitcoin-cli -rpcwallet=<wallet-name> getaddressinfo <previous-generated-address>`

After running this command you should have a JSON object as result with a `desc` which contains your output descriptor. It should look similar to this:

```json
{
    "address": "bcrt1q2e4g8cetupxes7l95qstpxq8qnpml9ev6gjle2",
    "scriptPubKey": "0014566a83e32be04d987be5a020b0980704c3bf972c",
    "ismine": true,
    "solvable": true,
    "desc": "wpkh([dc711bf8/0'/0'/1']0303207b2fd5b151397949aa9d6f9ff8eccd3bb97aa05192ced942b17bbd45e9d9)#yr975l8h",
    "iswatchonly": false,
    "isscript": false,
    "iswitness": true,
    "witness_version": 0,
    "witness_program": "566a83e32be04d987be5a020b0980704c3bf972c",
    "pubkey": "0303207b2fd5b151397949aa9d6f9ff8eccd3bb97aa05192ced942b17bbd45e9d9",
    "ischange": false,
    "timestamp": 1654093495,
    "hdkeypath": "m/0'/0'/1'",
    "hdseedid": "50b54a741dbc3b120280391ecba28d657e137bce",
    "hdmasterfingerprint": "dc711bf8",
    "labels": [""]
}
```

We can get more information about the descriptor in the result above by running the following command:

`bitcoin-cli getdescriptorinfo <descriptor>`

Using the descriptor in the JSON output above, I get the following result.

```json
{
  "descriptor": "wpkh([dc711bf8/0'/0'/1']0303207b2fd5b151397949aa9d6f9ff8eccd3bb97aa05192ced942b17bbd45e9d9)#yr975l8h",
  "checksum": "yr975l8h",
  "isrange": false,
  "issolvable": true,
  "hasprivatekeys": false
}
```

A script is solvable according to Core, if given the private key and the descriptor, the wallet software can generate a scriptsig/witness to spend its funds. `isrange` denotes if it is a range descriptor meaning it is possible to generate other descriptors from it. Range descriptors usually end their derivation path with `/*`. For example, `wpkh(tprv8ZgxMBicQKsPdy6LMhUtFHAgpocR8GC6QmwMSFpZs7h6Eziw3SpThFfczTDh5rW2krkqffa11UpX3XkeTTB2FvzZKWXqPY54Y6Rq4AQ5R8L/84'/0'/0'/0/*)` is a range descriptor.

The `getdescriptorinfo` command can also be used to derive the checksum of a descriptor if there you get one without a checksum. If we run the previous command, this time without adding the checksum at the end, Core will compute the checksum for us in the outputs.

`bitcoin-cli getdescriptorinfo "wpkh([dc711bf8/0'/0'/1']0303207b2fd5b151397949aa9d6f9ff8eccd3bb97aa05192ced942b17bbd45e9d9)"`

The output derives the checksum for us. See the JSON result below:

```json
{
  "descriptor": "wpkh([dc711bf8/0'/0'/1']0303207b2fd5b151397949aa9d6f9ff8eccd3bb97aa05192ced942b17bbd45e9d9)#yr975l8h",
  "checksum": "yr975l8h",
  "isrange": false,
  "issolvable": true,
  "hasprivatekeys": false
}
```

Aside from getting information about a descriptor, we can derive an address from a
descriptor. To do that, use the following command:

`bitcoin-cli deriveaddresses <descriptor>`

Using the descriptor above, we get the following address:

```json
[
  "bcrt1q2e4g8cetupxes7l95qstpxq8qnpml9ev6gjle2"
]
```

Finally, it is possible to import descriptors into another wallet using the `importdescriptors` command. 

`bitcoin-cli -rpcwallet=vladitest importdescriptors '[{ "desc": "wpkh([0ee8ad5c/84'"'"'/1'"'"'/0'"'"'/0/0]0384fa1d3495ae000e6da98daef1b68dfb0730bd1275f2ede4342b013ed8cd15a2)#y8j0e2j6", "timestamp":"now", "label": "" }]'`

Notice the `'"'"'` used to denote the hardened paths in the descriptor, consider this as an escape character. It is also worth mentioning that if you are importing a descriptor with public keys, you 
will need to create a wallet with private keys disabled by passing `disable_private_keys=true` as an argument.

## Conclusion

With the advent of wallet descriptors it will become very easy for wallet software to generate address or spend from a particular output script. These descriptors are extensible which means that if we suddenly have a new address/script type tomorrow, it will be easy to add it to this general structure.


## Resources

* [What's Coming To The Bitcoin Core Wallet in 0.21](https://achow101.com/2020/10/0.21-wallets)
* [Zpub (Extended Public Key)](https://river.com/learn/terms/z/zpub-extended-public-key/)
* [Ypub (Extended Public Key](https://river.com/learn/terms/y/ypub-extended-public-key/)
* [Output Descriptors](https://bitcoinops.org/en/topics/output-script-descriptors/)
* [Wallet Descriptors Stack Exchange](https://bitcoin.stackexchange.com/questions/99540/what-are-output-descriptors)
* [Rethinking Wallet Architecture: Native Descriptor Wallets](https://www.youtube.com/watch?v=xC25NzIjzog)
* [Understanding the Descriptor](https://github.com/BlockchainCommons/Learning-Bitcoin-from-the-Command-Line/blob/master/03_5_Understanding_the_Descriptor.md)
* [Output Script Descriptors for Bitcoin](https://www.youtube.com/watch?v=kGM3DNtd_iw)


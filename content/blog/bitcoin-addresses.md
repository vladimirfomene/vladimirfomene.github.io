+++
title = "Bitcoin Addresses"
image = "https://res.cloudinary.com/vladimirfomene/image/upload/v1653069616/blog/addresses.png"
description = "These addresses are like single-use (they could be used multiple times but it is not advised for privacy reasons) invoices that are issued by a receiver to a sender in a transaction"
url = "https://www.vladimirfomene.com/blog/bitcoin-addresses/"
date = 2022-04-18
+++



Welcome back to the blog! Today we will explore how Bitcoin Addresses are formed and how they are represented. You can find the repository for the code at the bottom of the article.

### Introduction

What are Bitcoin addresses anyway? Bitcoin is a decentralized network of computers which secures the funds of all the users of the network. Where do these funds live you might wonder? They live at your Bitcoin address. These addresses are like single-use (they could be used multiple times but it is not advised for privacy reasons) invoices that are issued by a receiver to a sender in a transaction. The best way to think about a Bitcoin address is to see it as a place on the network where your funds can live. As a user of the Bitcoin network, you can have more than one Bitcoin address.


### Address Utility

In this article, we are going to be focusing on a particular group of Bitcoin Addresses called Pay-to-Public-Key-Hash addresses(P2PKH). In a Bitcoin transaction where you are receiving funds, you have to give your Bitcoin address to the sender and the sender's wallet software will generate and sign a transaction that assigns some bitcoins to your address. This transaction will then be broadcasted to the Bitcoin network. Now that we have a good understanding of the utility of addresses, let's see how we can generate one.

### Generating Private Key

Before generating an address, we will first have to generate its public key. Additionally, before generating a public key, we will first have to generate a private key. So how do we generate a private key? First, we start by generating a cryptographically secure random number, in most systems, this means generating a key using your operating system as your source of randomness. To do that, we will generate 256 bytes (I choose this arbitrarily, just make sure the randomness is bigger than 256 bits) of randomness from the OS' cryptographically secure random number generator. After generating this entropy, we will generate a SHA256 digest from it. SHA256 because it produces keys which are 256 bits and the Bitcoin System requires our private key to be 256 bits. We have to make sure that this private key is less than the order of Bitcoin's elliptic curve, denoted in the code as `n`. This process will look like this in Python.

```python
import os
import hashlib


# Constants

n = 115792089237316195423570985008687907852837564279074904382605163141518161494337

  
# 1. Generate randomness and hash it with sha256 algorithm.

private_key = None

while True:

	entropy = os.urandom(256)

	private_key = hashlib.sha256(entropy).hexdigest()

	if int(private_key, 16) < n:
		break

break
```

After this process, your `private_key` variable will now contain a cryptographically secure private key in hexadecimal. 

> Please don't use this private key for real Bitcoin transactions. The code snippet above is strictly for educational purposes.

### Generating Public Key

Now that we have a private key, we need to generate our public key from this private key. To do that, we multiply our private key with the generator point of the [secp256k1  elliptic curve](https://www.secg.org/sec2-v2.pdf). Since our private key is just a big random number, we can mathematically multiply with a point on this curve.  For the elliptic curve, we pulled [James D'Angelo's implementation](https://github.com/wobine/blackboard101/blob/master/EllipticCurvesPart4-PrivateKeyToPublicKey.py) and just ported it to Python 3. Once we do that, we can import it in our project.

```python
...
import elliptic # Elliptic curve from James D'Angelo

# Constants

.....

generator = (

55066263022277343669578718895168534326250603453777594175500187360389116729240,

32670510020758816978083085130507043184471273380659243275938904335757337482424

)

# 2. Calculate the Public key from the Private key with Elliptic Curve.

public_key = elliptic.EccMultiply(generator,int(private_key, 16))

```

With this in place, your public key should now be in the `public_key` variable. This is just a point with X and Y coordinates on the elliptic curve. The `generator` used in the variable was copied from the secp256k1 standard document above and converted from hexadecimal to decimal.

### Private Key Format

Although our hexadecimal private key is valid, it is not the recommended format expected by wallet software when instantiating a wallet from the private key. The format expected by this software is called Wallet Import Format (WIF for short). This format is divided into four parts.
**version : payload : suffix : checksum**

- The version informs wallet software which type of key this is. For private keys, the version number is **0x80**.
- The payload is the hexadecimal form of the key as we generated it in the previous section.
-  The suffix indicates whether this key is compressed or not compressed. **0x01** for compressed keys and it is left empty for uncompressed keys.
-  The checksum helps the wallet software detect if the key has been transcribed correctly into the software.
	
WIF requires these four parts to be Base58 encoded. Once concatenated and encoded using Base58 encoding, the uncompressed private keys start with the number 5 while the compressed private keys start with K or L. Here is what implementing from our hexadecimal private key looks like:

```python
..........
import base58
from binascii import unhexlify as decode_hex

# 3. Generate a compressed and uncompressed private key

def generate_base58_format(payload, prefix, suffix = ""):
	hash256 = hashlib.sha256(decode_hex(prefix + payload)).hexdigest()
	checksum = hashlib.sha256(decode_hex(hash256)).hexdigest()
	checksum = checksum[:8]
	formatted_key = prefix + payload + suffix + checksum
	return base58.b58encode(decode_hex(formatted_key))


wif_key_compressed = generate_base58_format(private_key, "80", "01")
wif_key_uncompressed = generate_base58_format(private_key, "80")

print("WIF Private key Compressed:", wif_key_compressed)
print("WIF Private key Uncompressed:", wif_key_uncompressed)

```

Notice the importation of two new libraries at the top of the script. You will have to download `base58` for the encoding. `binascii` is part of the system, you don't have to download it. Did you notice that the compressed version of the key is bigger than the uncompressed version? What is that all about? We use compressed here to mean that compressed public keys can be generated from the compressed private keys and uncompressed public keys can be generated from uncompressed private keys. This reduces the number of public keys a wallet creates when you import a WIF private key as it can use this suffix to determine whether to generate a compressed public key or uncompressed public key. In the original Bitcoin software, the wallet supported Pay-To-Pubkey (P2PK) and Pay-To-Pubkey hash (P2PKH) addresses. If the suffix was not added, it means the wallets will have to search funds using four types of public keys: uncompressed_P2PK, compressed_P2PK, uncompressed_P2PKH and compressed_P2PKH. With the WIF suffix, wallets will reduce the types of keys used for fund search by 3. With the default key pool size of 1000 in Bitcoin Core, this will reduce the key types searched by 3000. This is a huge performance boost for large wallets with hundreds of thousands of addresses because it reduces the amount of processing required for funds associated with the wallet's key when a new block is received. 

### Base58 and Base58Check Encoding

The keys above are Base58Check encoded. What does that even mean? To represent long numbers compactly, many computer systems used alphanumeric representations. For example, `1000000` in decimal is `F4240` in hexadecimal, we went from seven characters to five characters. To represent Bitcoin addresses compactly, Bitcoin encodes them using Base 58 characters. Base58 encoding is just a subset of Base64 encoding with characters that might appear identical when displayed removed. Characters removed are 0 (zero), O (capital o), I (capital i), l (lower case L), + and /.

Base58 means characters in Bitcoin addresses can be between 0 and 57. Here are the characters in the base58 alphabet: `123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz`. **0** is represented by **1** and **57** by **z** in Base58. For Base58Check, it means instead of just encoding the number for which we want a compact representation, we will append a checksum to it before encoding. In Bitcoin, the checksum is the first four bytes of the `SHA256(SHA256(prefix + payload))` where the prefix is the version and the payload is the private key. In the code above, we are using the first 8 characters because the hash is in hexadecimal and a byte is equivalent to two hexadecimal characters. Since the checksum is derived from the hash of the encoded data, it can be used to detect errors in Bitcoin addresses. For example, when a user mistypes an address in a wallet.

### Public Key Format

Now that we started talking about public keys, you might be wondering how to format them. They have the same format as the private key except that their suffix is always empty. The payload is also different depending on whether it is a compressed or uncompressed key. 

#### Uncompressed Public Keys

- Version number is **0x04**
- Payload is a concatenation of the X and Y coordinate of the point.
- Suffix is empty
- Checksum is calculated as in the case of private keys.


#### Compressed Public keys

To move from uncompressed keys to compressed keys, we discard the Y coordinate of our point. This is because we can always re-compute it from the elliptic curve. 

- The version number of the compressed public key is **0x02** if the Y coordinate is even and **0x03** if the Y coordinate is odd. 
- The payload is just the X coordinate
- Suffix is empty 
- Checksum is calculated as in the case of private keys.


### Generating Bitcoin Address

With all this knowledge, let's now generate a compressed and uncompressed public key. Then, we can generate their Bitcoin addresses. The following code does exactly that:

```python
# we slice from index 2 because we don't want the string "0x" to be part of the key. Same 
# reason everywhere else.
uncompressed_public_key = "04" + hex(public_key[0])[2:] + hex(public_key[1])[2:]
# we compute prefix which will be used for our public key
prefix_compressed_public_key = "02" if public_key[1] % 2 == 0 else "03"

compressed_public_key = prefix_compressed_public_key + hex(public_key[0])[2:]
print("Uncompressed Public key: ", uncompressed_public_key)
print("Compressed Public key: ", compressed_public_key)

# When generating Bitcoin addresses from public keys, we have to add a "00" prefix during the #encoding.
print("Uncompressed Bitcoin Address: ", generate_base58_format(uncompressed_public_key, "00"))
print("Compressed Bitcoin Address: ", generate_base58_format(compressed_public_key, "00"))

```

### Other Address Types

There is another class of addresses that we haven't looked at called Pay-To-Script-Hash addresses. These addresses are generated by calculating the following hash:

`hash = RIPEMD160(SHA256(script))` 

Then encoding this hash with base58 checksum with a version prefix of 5. They usually start with "3" after encoding. Aside from this, SegWit also introduced two new address types, Pay-To-Witness-Script-Hash (P2WSH) and Pay-To-Witness-Public-Key-Hash (P2WPKH).

### Conclusion

In this post, we explored how to generate a private key from a cryptographically secure source of entropy, we used this private key to generate public keys and finally saw how we could format both private and public keys. The addresses shown in this post are considered legacy addresses as most Bitcoin wallets no longer use them. Nowadays, most wallets use SegWit addresses which are encoded using Bech32 instead of the Base58 encoding used above. These might be the subject of a future post. Stay tuned!

### Resources

- [Elliptic Curve Standard documentation](https://www.secg.org/sec2-v2.pdf)
- [Mastering Bitcoin, Andreas Antonopoulos](https://github.com/bitcoinbook/bitcoinbook/blob/develop/ch04.asciidoc)
- [Learn Me a Bitcoin](https://learnmeabitcoin.com/technical/public-key)
- [Python Bitcoin Addresses Repo](https://github.com/vladimirfomene/python-addresses)
- [Why do WIF-compressed private keys exist?](https://www.reddit.com/r/Bitcoin/comments/g46nvw/why_do_wifcompressed_private_keys_exist/)

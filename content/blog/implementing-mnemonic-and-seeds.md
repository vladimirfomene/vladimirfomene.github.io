+++
title = "Implementing Mnemonics & Seeds in Rust : BIP 39"
date = 2022-05-13
+++

![Mnemonic Words](https://res.cloudinary.com/vladimirfomene/image/upload/v1653067145/blog/mnemonic.jpg)

## Introduction

A mnemonic is a system/device that helps with retention. This could be a pattern of letters, words, a phrase or even ideas. In our particular scenario we are trying to remember a very large number called a seed. This seed is used by wallets to derive all your keys and therefore calculate your balance. When you download a wallet software, most wallets make you store a list of either 12 or 24 words as a backup for the wallet. With this setup, if you decide to suddenly change wallet software you can use this word list on the new wallet to recover your keys and calculate your balance. This list of words is called a mnemonic because it helps us remember or recover the seed. 

Wallets that implement BIP 39 are able to recover your keys from the mnemonic. Most of them use 12 or 24 words, but it is also possible to use 15, 18 and 21 words.  Next, we will be looking at how we can generate our own mnemonic. To follow along download the codebase for the article [here](https://github.com/vladimirfomene/bip39).

## Generate entropy

Before we can generate a mnemonic phrase, we need to first generate cryptographically secure entropy (randomness). To do that, in the `entropy.rs` file, we create a new struct called `Entropy` with a field called `entropy` which has a vector of bytes as value. In the implementation of the struct, we created a new function which calls a cryptographically secured random number generator to generate the entropy. The code looks like this:

```rust
impl Entropy {

  

	//create new entropy (large random number)
	//entropy size here is in bits.
	pub fn new(size: usize) -> Entropy {

		//Probably change this to a better error handling mechanism if this has

		//to be a library.



		//check if size is between 128 and 256

		assert!((size >= util::MIN_NUM_BITS) && (size <= util::MAX_NUM_BITS));



		//check if size is a multiple of 32

		assert!((size % util::ENTROPY_MULTIPLE) == 0);



		//create a vector filled with 0 bytes

		let mut entropy: Vec<u8> = vec![0u8; size / util::BYTE_SIZE];



		//generate cryptographically secure random bytes and store it in entropy vector

		rand::rngs::OsRng.try_fill_bytes(&mut entropy).unwrap();



		return Entropy {

		entropy

		}

	}

}
```

In the function, we make sure the bits used to generate entropy have are between 128 and 256 in size. This is enough bits to make sure that two people don't generate thesame entropy. There is also a check to make sure the size of these bits is a multiple of 32. Thirty two because it allows us to easily split the entropy in pieces and convert them to words. There after, we fill the vector of bytes by calling our cryptographically secured random number generator. Since we have a vector of bytes and not bits, our vector's length has to be `size` divided by the `BYTE_SIZE` . You can check the values of constants in the `util.rs` module.

## From Entropy Generate Checksum

According to the BIP, assuming the length in bits of the entropy is `ENT`. The checksum is the first `ENT / 32` bits of the SHA256 hash of the entropy.  For all our possible entropy len, we can have the following checksum size:

<pre>
Let checksum size be, CS = ENT / 32.
The checksum size for all entropy length is as follows:

|  ENT  | CS | 
+-------+----+
|  128  |  4 | 
|  160  |  5 | 
|  192  |  6 | 
|  224  |  7 | 
|  256  |  8 | 
</pre>

[Table Source](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)

On the table we see that irrespective of the entropy length we choose, our checksum size will never be more than 8 (a byte). So to get the checksum, we generate a SHA256 hash of the entropy and take the first Byte.  Here is the code to generate the checksum from the entropy:

```rust
fn generate_checksum(ent: Entropy) -> Vec<u8> {

	let mut hasher = Sha256::new();

	hasher.update(&ent.entropy);

	let entropy_hash = hasher.finalize();

	return entropy_hash[0..1].to_owned();

}
```

## Get Mnemonic from entropy and checksum

Now that we have an entropy and its checksum, we can calculate a mnemonic. Let the word count of a mnemonic be `WC`. The word count for each entropy length is calculated like `WC = (ENT + CS) / 11` where `ENT` is the entropy length and `CS` is the checksum's length. This can be represented with the following table:

<pre>
WC = (ENT + CS) / 11

|  ENT  | CS | ENT+CS |  WC  |
+-------+----+--------+------+
|  128  |  4 |   132  |  12  |
|  160  |  5 |   165  |  15  |
|  192  |  6 |   198  |  18  |
|  224  |  7 |   231  |  21  |
|  256  |  8 |   264  |  24  |
</pre>

[Table Source](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)

Base on the formula for `WC` above we have divided our concatenated entropy and checksum bits into chunks of **11** bits. Therefore, our words are represented by 11 bits numbers. With 11 bits, we can represent 2048 (2^11) numbers. These numbers can be from 0 to 2047. Each of these numbers is an index in our array of 2048 words found in `src/language/english.rs`. This works because the word list is defined in the [spec](https://github.com/bitcoin/bips/blob/master/bip-0039/english.txt). If everyone created their own word list wallets won't be compatible. We used the following snippet of code from the `src/mnemonic.rs` module to generate our mnemonic words.

```rust
impl Mnemonic {

  

	pub fn from_entropy_checksum(entropy: Vec<u8>, checksum: Vec<u8>) -> Mnemonic {

		let checksum_size = (entropy.len() * util::BYTE_SIZE) / util::ENTROPY_MULTIPLE;

		let entropy_size = entropy.len() * util::BYTE_SIZE;

		let mnemonic_word_count = (entropy_size + checksum_size) / util::WORD_BITS;



		//create a vector of bits where

		let mut bits = vec![false; entropy_size + checksum_size];



		//add entropy bits to bits array

		for (index, bit) in bits[..(entropy.len() * util::BYTE_SIZE)].iter_mut().enumerate(){

		*bit = util::get_index_bit(entropy[index / util::BYTE_SIZE], index % util::BYTE_SIZE);

		}



		//add checksum bits to bits array

		for (index, bit) in bits[(entropy.len() * util::BYTE_SIZE)..].iter_mut().enumerate() {

		*bit = util::get_index_bit(checksum[0], index);

		}



		//create vector for mnemonic words.

		let mut mnemonic_list = Vec::with_capacity(mnemonic_word_count);

		for chunk in bits[..(checksum_size + entropy_size)].chunks(11) {

		//convert 11 bit chunk to word index

		let word_index = util::bits_to_usize(chunk, 11);



		//use word index to get word from wordlist

		let word = english::WORDS[word_index];



		//add word to mnemonic list

		mnemonic_list.push(word.to_string());

		}


		return Mnemonic {

			lang: language::Language::English,

			words: mnemonic_list

		}

	}

}
```

## Generate Seed from Mnemonic

With the above code, we have been able to generate our list of mnemonic words from our english wordlist. Now we are going to pass our mnemonics through a password based key derivation function (PBKDF2 for short).  This key derivation function uses HMAC-SHA512 as its pseudo-random function. The `password` that we are going to hash is the mnemonic sentence. The salt is the  `"mnemonic" + passphrase` where the passphrase can be empty. This algorithm(PBKDF2) will produce different seeds from the same mnemonic if given different passphrases. We are going to use 2048 iterations in our PBKDF2. This just refers to the number of times we are going to be hashing. One important thing to mention here is that the `salt` and `password` have to be UTF-8 normalized before passing it to this function. Normalization is important because it prevents our program from treating words looking thesame differently because they have different Unicode representation. See more [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/normalize). Here is the code snippet for generating a seed from a mnemonic.

```rust
pub (crate) fn compute_seed(mnemonic: Vec<String>, passphrase: &str) -> Vec<u8>{

  
	//join our mnemonic words into one sentence
	let mnemonic_sentence = mnemonic.join(" ");

	//use the unicode-normalization crate to normalize string
	let normalized_mnemonic = mnemonic_sentence.nfkd().collect::<String>();

	//convert normalized string to bytes
	let normalized_mnemonic = normalized_mnemonic.as_bytes();

	//create salt from prefix and passphrase
	let salt = format!("{}{}", SALT_PREFIX, passphrase);
	
	//normalize salt string
	let normalized_salt = salt.nfkd().collect::<String>();

	//convert salt string to bytes
	let normalized_salt = normalized_salt.as_bytes();

	//create a seed vector to hold seed bytes
	let mut seed = [0u8; PBKDF2_BYTES];

	//call PBKDF2 to generate seed and store it in seed vector
	pbkdf2::pbkdf2::<Hmac<Sha512>>(normalized_mnemonic, normalized_salt, PBKDF2_ITERATIONS as u32, &mut seed);

	return seed.to_vec();
}
```


## Conclusion

In this article, we have shown how to move from entropy, to checksum, to mnemonic and from mnemonic to seed. A BIP39 Mnemonic is a list of words used as backup by wallets. By backup, I mean it can be used by any BIP39 compatible wallet software to recover your keys and therefore calculate your balance.

## References

While working on this article, I learned a lot while reading the following implementations alongside the BIP 39 Wiki. 

- [BIP 39 Wiki](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)
- [Article Code](https://github.com/vladimirfomene/bip39)
- [Alternative Implementation](https://github.com/koushiro/bip0039)
- [Alternative Implementation](https://github.com/rust-bitcoin/rust-bip39)

# Slot Machine
Category: MISC
Points: 453
Solves: 76

### TLDR

Bitcoin go brrr.

### Solution

We aim to find a hex value such that its SHA-256 hash starts with *k* identical characters.

To achieve this, we use Bitcoin block hashes, which start with many zeros due to proof-of-work mining.


After some googling we can find the lowest bitcoin block hash:
`0000000000000000000000005d6f06154c8685146aa7bc3dc9843876c9cefd0f`
That's 24 zeros in a row! (This perfectly covers the length of the flag)
(https://bitcoin.stackexchange.com/questions/65478/which-is-the-smallest-hash-that-has-ever-been-hashed)

Using online APIs (e.g., https://blockchain.info/rawblock/#hashvalue) and some coding, we can retrieve the block header, which serves as the preimage (unhashed value) for the hash:

```py
# functions are in "Full Script" section below

# get JSON data of block_hash
block = load_block_data(block_hash)

# calculate block hash
block_header = block_hash_preimage(
    version=block['ver'],
    prevblock=block['prev_block'],
    merkle_root=block['mrkl_root'],
    time=block['time'],
    bits=block['bits'],
    nonce=block['nonce'],
)
```

We then reverse the block header, as the chal requires reversing the input for some reason:


```py
# chal
hash = sha256(bytes.fromhex(lucky_number)[::-1]).digest()[::-1].hex()
```

```py
# script
payload = block_header[::-1].hex()
```

This leads to the payload:
`93ccd10e30712e566e0bc0189c791e609b11fc17190b00eb50d6fa8b4909b2f5`

Submitting this to the server with a length of 24 yields the flag!

![Slot Machine Submit](/images/SlotMachineSubmit.png)

### Full Script
```py
import hashlib
import struct

import requests
from pwn import remote


# modified from https://gist.github.com/Ironpark/9c9a10040eecb2e6ac071ae6f30b1560
def load_block_data(block_hash):
    r = requests.get('https://blockchain.info/rawblock/' + block_hash)
    return r.json()


def block_hash_preimage(version: int, prevblock, merkle_root, time: int, bits: int, nonce: int):
    prevblock_little = bytearray.fromhex(prevblock)[::-1]
    merkle_root_little = bytearray.fromhex(merkle_root)[::-1]

    # pack binary little endian
    header_bin = struct.pack(
        # < : little endian
        # i : signed int (32 bit) - +
        # I : unsigned int (32 bit) +
        # 32s : 32 byte (char)

        # <i32s32sIII = little endian | int | byte[32] | byte[32] | unsigned int | unsigned int | unsigned int
        '<i32s32sIII',
        version,
        prevblock_little,  # little -> big
        merkle_root_little,  # little -> big
        time,
        bits,
        nonce,
    )

    return hashlib.sha256(header_bin).digest()


def main():
    # https://www.blockchain.com/explorer/blocks/btc/756951
    block_hash = "0000000000000000000000005d6f06154c8685146aa7bc3dc9843876c9cefd0f"
    block = load_block_data(block_hash)

    # calculate block hash
    block_header = block_hash_preimage(
        version=block['ver'],
        prevblock=block['prev_block'],
        merkle_root=block['mrkl_root'],
        time=block['time'],
        bits=block['bits'],
        nonce=block['nonce'],
    )

    assert block_hash == hashlib.sha256(block_header).digest()[::-1].hex()

    prefix_len = max(i for i in range(1, len(block_hash) + 1) if set(block_hash[:i]) == {'0'})

    # chall reverses it for some reason
    payload = block_header[::-1].hex()


    print(payload)
    print(prefix_len)


if __name__ == '__main__':
    main()
```
### False Starts, Dead Ends, and Other Exploration

**Understanding the Challenge**

Starting out by reading the challenge script, we see that it asks us for a hex input which gets hashed using SHA-256:

```py
lucky_number = input("What's your lucky (hex) number: ").lower().strip()
...
hash = sha256(bytes.fromhex(lucky_number)[::-1]).digest()[::-1].hex()
```

We are then asked for a *length* value:
```py
length = min(32, int(input("How lucky are you feeling today? Enter the number of slots: ")))
```

The next part of the code, while daunting at first, simply prints out the prefix of our hash using our provided length and makes it look fancy:

```py
print("=" * (length * 4 + 1))
print("|", end="")
for c in hash[:length]:
    print(f" {hex_alpha[hex_alpha.index(c) - 1]} |", end="")
print("\n|", end="")
for c in hash[:length]:
    print(f" {c} |", end="")
print("\n|", end="")
for c in hash[:length]:
    print(f" {hex_alpha[hex_alpha.index(c) - 15]} |", end="")
print("\n" + "=" * (length * 4 + 1))
```

The real part of the challenge seems to be getting this prefix of the hash to be all the same character, at which point we will be awarded a prefix of the flag of length length:

```py
if len(set(hash[:length])) == 1:
    print("Congratulations! You've won:")
    flag = open("flag.txt").read()
    print(flag[:min(len(flag), length)])
else:
    print("Better luck next time!")
```

As expected, testing out a length of 1 reveals the first character 'u':

![Testing Length 1](/images/SlotMachineTest1.png)

**Brute Force**

The first idea that popped into my head was to brute-force random hex strings until I found one with a long same-character prefix. Obviously, this would take a long time, but maybe I could get *EXTREMELY* lucky.


```py
from hashlib import sha256
import random

hex_alpha = "0123456789abcdef"
best = 0

# generate random starting string
lucky_number = ''.join(random.choice(hex_alpha) for i in range(32))

for i in range(1000000000000):
    
    # compute hash
    hash = sha256(bytes.fromhex(lucky_number)[::-1]).digest()[::-1].hex()
    
    # compute prefix
    cnt = 0
    for x in hash:
        if x == hash[0]: cnt+=1
        else: break

    # keep track of longest prefix
    if cnt > best:
        best = cnt
        print(lucky_number, best)
        print(hash)
        print("-"*64)

    # set to old hash to reduce time it takes to generate a random string
    lucky_number = hash

    # counter to keep track of brute force
    if i % 10000000 == 0:
        print("on iter", i)
```

After only a few seconds, I found a prefix of length 7:

![brute forced hashes](/images/SlotMachineBruteForce.png)

Excitedly, I tested this value:

![brute force test](/images/SlotMachineBruteForceTest.png)

Darn! I didn't even get past the flag wrapper.

After 4 hours of running the script, the longest prefix I ever got was 9:
`13848321aa72e41e0210f13dbf9217abb4e004755c69077c8d4a715d755d4e0b`
hashed
`66666666643b696b89ff4ef4734fa7ee3dc1dffa5bb565914989e99e3c5382e4`

**Negative Length**

Upon closer inspection of the code, I realized that the input for *length* wasn't sanitized, allowing for negative integers. Given that Python supports negative string indexing and slicing, I explored this idea.

I theorized that if the flag was longer than 32 characters, we could never get the full flag since *length* is capped at 32. By entering -1, we might get the whole flag minus the '}'. However, this would require the hash to have a 63-character long prefix, and I couldn't even reach 10.

In summary, while the concept of negative slicing was valid, it wasn't useful because `string[:negative_num]` still returns a prefix, which doesn't change the problem.

```py
>>> "prefix"[:4]
'pref'
>>> "prefix"[:-2]
'pref'
```
Since the flag length was probably less than 32, negative slicing would just cut off more of the flag.

![Negative Length Test](/images/SlotMachineNegativeLengthTest.png)

**Bitcoin** (Solution)

After a while, my teammate suggested using Bitcoin hashes since they start with many zeros (https://www.blockchain.com/explorer/blocks/btc).

After some research and copy-pasting, we learned that Bitcoin block hashes are generated using SHA-256 through a complex process (https://www.gemini.com/cryptopedia/what-is-block-in-blockchain-bitcoin-block-size#section-bitcoin-block-headers-and-mining, https://gist.github.com/Ironpark/9c9a10040eecb2e6ac071ae6f30b1560).

In short, instead of brute-forcing our own hash, we leveraged the years of brute-forcing done by Bitcoin miners to find a hash key.

Starting out with recently mined hashes we used one with 20 zeros:
`00000000000000000000a9bb06a63163965b97b584a5119d278b7cf696250099`
yielding: `uiuctf{keep_going!_3`

Googling for long zero-prefix hashes eventually led us to a prefix of 24, the size of the flag (https://blog.bitmex.com/bitcoins-lowest-block-hash-values/, https://bitcoin.stackexchange.com/questions/65478/which-is-the-smallest-hash-that-has-ever-been-hashed).


### Flag
```uiuctf{keep_going!_3cyd}```
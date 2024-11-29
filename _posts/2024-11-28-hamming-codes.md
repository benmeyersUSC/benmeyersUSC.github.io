## Hamming Codes 

[my Python script]([hamming/HammingCodes.py](https://github.com/benmeyersUSC/RandomScripts/blob/main/hamming/HammingCodes.py#L61))

Hamming codes are a category of error-correcting codes that enable durable and dynamic sending of packets of data. Suppose I wanted to sent a byte of information over a network
to another computer: there could be any number of interferences that may cause a bit to be flipped along the way. 
Do I send it every couple seconds until I get a reply confirming
the reception? What if *that* message has a bitflip? The same goes for storing data on a drive or disk or several other states in which we want binary information to stay in tact.
How can we embed within the data some mechanism to check and correct any errors that may arise in its transit or storage?

### Enter Claude Shannon and John Hamming
While working at Bell Laboratories, these two pioneered not only information theory, but Error-Correcting Codes. 
**They essentially sought to answer the question: *can we devise a protocol to encode and decode messages in binary with some mechanism that can detect and then fix any unintended errors?***


If we wanted to send, for example, the four-bit code ```1011``` , how could we embed within it a minimal set of additional bits to aide in error-correction? We could definitely imagine ways
to stuff error-correcting (EC) bits between each data (D) bit and somehow use those to ensure fidelity, right? What if we, say, prepended each bit with an EC bit that is just a copy of
the D bit that follows, so our message would become ```11001111``` ? Could certainly work to add redundancy. But doing so doubles the size of our message! At scale, this is a costly
mechanism. 

What about instead adding the binary sum at the end of the four-bit piece of data, so our example would become ```101111``` , where the first four bits are the same old D bits and the 
three EC bits at the end correspond to the sum of the digits (3, or 11 in binary)? As our D-bit count grows in larger messages, this sum at the end would grow logarithmically, achieving
efficiency at scale. This method could certainly tell us *whether or not* the data was messed with in any way, to a relatively high degree of certainty. But what about remedying the error?
Do we really want to have to retrieve another copy through another transmission? What about data that is corrupted in storage? We need a way to not only detect, but fix!




As with many of the revolutionary insights at this time and place, reaching far and deep into the future, the answer lay in the powerful structure of binary digits themselves. 
In Hamming-encoded EC code, for a four-bit message, we could instead add 3 EC bits, which we will now call Parity (P) bits to our message and achieve some incredible results. 

Instead of ```D1 D2 D3 D4```, we can construct ```P1 P2 D1 P3 D2 D3 D4```

What is going on here? Why 3 P bits? Why that order?

First, let us refresh with some counting in binary from 1 to 7 (the range of total bits in our new message): 
>- 001 = 1
>- 010 = 2
>- 011 = 3
>- 100 = 4
>- 101 = 5
>- 110 = 6
>- 111 = 7

Notice that certain number have a 1 in the far right (least significant, or "first") place of the number. Some have a 1 in the 2 spot or the 3 (far left, most significant) spot. Some have
1s in multiple places. Let's organize them visually...

"1" in the __:
>- 1st place: [1, 3, 5, 7]
>- 2nd place: [2, 3, 6, 7]
>- 3rd place: [4, 5, 6, 7]

There is certainly some overlap between these groups, but they are all less than 7 in length. And in fact, the overlap becomes supremely useful. Suppose we implemented a similar sum-checking mechanism as discussed before, but with
respect to these groups. Say P1 can somehow correspond to that first group, P2 to the second, and P3 to the third. Recognize again that these groups have a very specific affinity with each other: they share the same bit (or weight) in a given index. Notice that binary (and even decimal notation) is simply a weighted sum notation, where weights are either 0 or 1 and the elements being weighted are powers of 2. 

What can we do here? Let's revisit our proposed encoded message once more, 
but with the D bits filled in:

```python
(P1) (P2) 1 (P3) 0 1 1
```

Hamming devised a scheme whereby, starting with P1, we would assign these P bits a value that ensures that the sum of their "group" is even. This way, we know *something* about certain parts of our message when it is sent and thus we have expectations for this property maintaining itself in transit and storage. If we recieve a Hamming-coded message and any of these parity bit groups do not add to an even number, then we will have known something went wrong. Let us see it in action.

#### Parity Bit Assignment
>- P1: b1(=P1) + b3(=1) + b5(=0) + b7(=1) = P1 + 2.....P1 must then be 0
>- P2: b2(=P2) + b3(=1) + b6(=1) + b7(=1) = P2 + 3.....P2 must then be 1
>- P3: b4(=P3) + b5(=0) + b6(=1) + b7(=1) = P3 + 2.....P3 must then be 0

Now we know that our message, encoded, should be:

```python
(0) (1) 1 (0) 0 1 1
```

#### Decoding the Message
When we receive a Hamming-coded message, we need to now verify that it made it here in tact. To do so, we need to verify these sums are even, because we know they were when the message was encoded. If they are all even, nothing more is needed and we just extract from the encoded message the bits at indices of powers of two (in other words, the P bits). 

If any of the sums are not even, then we have some work to do. Let's take a look at the code in [my Python script]([hamming/HammingCodes.py](https://github.com/benmeyersUSC/RandomScripts/blob/main/hamming/HammingCodes.py#L61)) to see how we fix an erroneous message. 

```python
# p_bit_keys = ["P1", "P2", "P3"]
# msg = [0, 1, 1, 0, 0, 1, 1]

flipped_bit_index = 0                       # where we will accumulate the index of the flipped bit
for k in p_bit_keys:                        # check each parity bit
    indices = parities[k]["indices"]            # bits that THIS P bit will check
    sm = 0                                      # sum accumulator for inner loop (indices within a given P bit)
    for j in indices:                           # for each bit index (in the message) that THIS P bit checks
        sm += msg[j - 1]                            # add to sum the bit at the given index
    if sm % 2 != 0:                             # if the sum is not even
        ordinal = int(k[1:]) - 1                    # index in (a binary number) that THIS P bit corresponds to
        flipped_bit_index += 2 ** ordinal           # add 2 ^ P bit's corresponding index
```

Diving into the code, we see that we are actually accumulating the answer for which bit may have been flipped on the fly. For each P bit and its corresponding set of bits it checks (always including itself), we calculate the sum. If that sum is odd, we have an error in this group and **we thus know that the index of the flipped bit will have a weight of 1 in the binary representation of its index**. For instance, if we see, in the simple case, that P*3* alone has an odd sum, we know that the *3*rd bit (from the left) in the binary number of the index in the message is flipped. That is, we know it is the 4th, 5th, 6th, or 7th bit (from the left...I know it is confusing) that is flipped. Well, which is it? Doing some deduction, we can see that only one of the values in this group is not in any others, 4. If it were any of the other numbers in this group that triggered the failed check, then other Parity checks would have failed! If only one Parity check fails, we know that it is the *Parity bit itself* that flipped. In the code we see that the only thing we ever accumulate in *flipped_bit_index* is the index of the P bit we are checking. This is actually more complicated to appreciate than the case where multiple checks fail...

Supposing instead that both P1 and P3 failed their checks, the code provides a much more clear picture of the elegance of this method. If these checks failed, we know that we are adding to *flipped_bit_index* both (2 ^ 0) and (2 ^ 2), which equals 5! And you guessed it, 5 is the only number that is in both P1 and P3's 'jurisdiction', if you will. If P1 and P2 failed, we would get (2 ^ 1) + (2 ^ 2), which gives 6. 

#### The Beauty of the Encoding

Beautifully, the algorithm just checks and accumulates the powers of 2, the basis vectors, of the groups that fail the check. If 7 is the bit that is flipped, we know that every single check needs to have failed because 7 contains in its sum: 2^0, 2^1, and 2^2. The only way to reach 7 is with this unique combination of powers of 2. The only way to reach 3 is with the precise mix of 2^0 and 2^1, the P bits 1 and 2 (again, the indexing is frustrating). 

The code may look naive and incomplete, in that a given failed check can be so confident as to what to add to our accumulated answer. However, *by construction*, we know that this method will work, because we encoded this message with P bits that specifically correspond to powers of 2. The only way to reach a certain number is with a certain combination of powers of 2 and certain powers of two can only reach certain numbers.


This process, as you can see in [my script]([hamming/HammingCodes.py](https://github.com/benmeyersUSC/RandomScripts/blob/main/hamming/HammingCodes.py#L61)), which exhaustively does this process for a random message of any length, will work for all lengths of messages. The only drawback in Hamming codes is that it only works to find a single flipped bit. If a message is corrupted in more than one place, we would be out of luck. Thank god for Hadamard's Error Correcting Codes!


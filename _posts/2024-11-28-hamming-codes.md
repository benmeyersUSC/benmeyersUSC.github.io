## Hamming Codes 

[HammingCodes.py](https://github.com/benmeyersUSC/RandomScripts/tree/main/error_correcting_codes)

Hamming codes are a category of error-correcting codes that enable both detection and reversion of errors in binary data. Suppose I wanted to sent a byte of information over a network
to another computer; suppose I have data being stored on a disk in my computer; while technology today seems to do these things effortlessly, it was not always so. If you could just send binary data through wires or through the air, how would you go about efficiently ensuring that messages come through cleanly? There are many different ways in which a bit, being represented somehow physically in transit or storage, can be flipped or shut to 0. Data can become somewhat garbled in translation. 
Do I send it every couple seconds until I get a reply confirming
the clean reception? What if *that* message has a bit-flip on its way to me?
How can we embed within the data some mechanism to check and correct any errors that may arise in its transit or storage and do so efficiently?

### Enter Claude Shannon and John Hamming
While working at Bell Laboratories, these two pioneered Error-Correcting Codes in binary data that could first just detect errors, then eventually finding ways to precisely locate and thus fix errors in binary. 
**Shannon and Hamming sought to answer the question: *can we devise a protocol to encode and decode messages in binary with some mechanism that can detect and then fix any unintended errors?***


If we wanted to send, for example, the four-bit code ```1011``` , how could we embed within it a minimal set of additional bits to aide in error-correction? We could definitely imagine ways
to stuff error-correcting (EC) bits between each data (D) bit and somehow use those to ensure fidelity, right? What if we, say, prepended each bit with an EC bit that is just a copy of
the D bit that follows, so our message would become```EC D EC D EC D EC D``` or ```1 1 0 0 1 1 1 1``` and have the receiving machine just check in pairs and get extra assurance of the data? Could certainly work to add redundancy. But doing so doubles the size of our message! At scale, this is a costly
mechanism. 

What about instead adding the binary sum at the end of the four-bit piece of data, so our example would become ```D D D D EC EC...``` or ```1 0 1 1 1 1``` , where the first four bits are the same old D bits and the three EC bits at the end correspond to the sum of the digits (1 + 0 + 1 + 1 = 3, or 11 in binary)? As our D-bit count grows in larger messages, this sum at the end would grow logarithmically, achieving size efficiency at scale. This method could certainly tell us *whether or not* the data was messed with in any way. What about these sum bits flipping, though? What about finding and remedying the error?
Do we really want to have to retrieve another copy through another transmission if there is any error detected? We need a way to not only detect, but fix!


#### Parity

As with many of the revolutionary insights at this time and place, reaching far and deep into the future, the answer lies in the powerful structure of binary digits themselves. The powerful and fundamental nature of a bit, literally having only two possible values, means that finding the location of an error means fixing it. We can use them to represent so much more than numbers in their most plain form. 
Similar to the approach of counting and checking a sum of the binary in the message, we can use the concept of Parity to verify whether our data was corrupted. However, while Hamming adopts all the best qualities of our original sum idea, it does so in an incredibly intelligent way that allows the uncorrupted message to be recovered from one where a bit has been flipped. (To be clear, Hamming Codes only work for a single bit flipping...it is however a conceptual jumping-off point and foundational for error correcting codes in general.)
Using Hamming Codes, for a four-bit message, we could instead add 3 EC bits, which we will now call Parity (P) bits to our message and achieve some incredible results. 

Instead of 
```python
D1 D2 D3 D4
```
we can construct 
```python
(P1) (P2) D1 (P3) D2 D3 D4
```

What is going on here? Why 3 P bits? Why that order?

First, let us refresh with some counting in binary from 1 to 7 (the number, or indices, of digits in our encoded message...we'll need these): 
>- 001 = 1
>- 010 = 2
>- 011 = 3
>- 100 = 4
>- 101 = 5
>- 110 = 6
>- 111 = 7

**Binary represents the weight or amount of powers of 2 starting at 2^0 on the far right (the 1's place) then 2^1 next (2's place) then 2^2 (4's place), and so on. We will be referring to these places in order from left to right, where the 1's place will the index 1, the 2's place will be index 2, and the 4's place will be index 3.**


Notice that certain number have a 1 in index 1. Some have a 1 in index 2 or 3. Some have
1s in multiple indices. Let's organize them accordingly...

"1" in the __:
>- 1's place, index 1: [1, 3, 5, 7]
>- 2's place, index 2: [2, 3, 6, 7]
>- 4's place, index 3: [4, 5, 6, 7]


There is certainly some overlap between these groups, but they're clearly well defined. And in fact, the overlap becomes crucially useful. For a number to be in one of these groups, it must have a 1 in the right index; it must include a specific *power* of 2 in its sum. Specifically, for any of these numbers to be created by adding powers of 2 (**ie represented in binary!**) it can only be done in one distinct way. 


Suppose we implemented a similar sum-checking mechanism as discussed before, but with respect to these groups. Say P1 can somehow correspond to that first group (binary index 1), P2 to the second, and P3 to the third. Recognize again that these groups have a very specific affinity with each other: they share the same bit (or weight) in a given index. 

What can we do here? Let's revisit our proposed encoded message once more, 
but with the D bits filled in:

```python
(P1) (P2) 1 (P3) 0 1 1
```

Hamming devised a scheme whereby we would assign these P bits a value that ensures that the sum of their index group is even. This way, we know *something* about certain parts of our message when it is sent and thus we have expectations for this property maintaining itself in transit and storage. If we recieve a Hamming-coded message and any of these parity bit groups do not add to an even number, then we will have known something went wrong And even more than that...Let us see it in action.

#### Parity Bit Assignment
>- P1: b1(=P1) + b3(=1) + b5(=0) + b7(=1) = P1 + 2.....*P1 must then be 0*
>- P2: b2(=P2) + b3(=1) + b6(=1) + b7(=1) = P2 + 3.....*P2 must then be 1*
>- P3: b4(=P3) + b5(=0) + b6(=1) + b7(=1) = P3 + 2.....*P3 must then be 0*

Now we know that our message, encoded, should be:

```python
(0) (1) 1 (0) 0 1 1
```

#### Decoding the Message
When we receive a Hamming-coded message, we need to now verify that it made it here in tact. To do so, we need to verify these sums are even, because we know they were when the message was encoded. If they are all even, nothing more is needed and we just remove from the encoded message the bits at indices of powers of two (in other words, the P bits). 

If any of the sums are not even, then we have some work to do. Let's take a look at the code in [my Python script](https://github.com/benmeyersUSC/RandomScripts/tree/main/error_correcting_codes) to see how we fix an erroneous message. 

```python
# In our example:
# parities = {
    {"P1": "indices": [1, 3, 5, 7], ...},
    {"P2": "indices": [2, 3, 6, 7], ...},
    {"P3": "indices": [4, 5, 6, 7], ...}
}
# msg = [0, 1, 1, 0, 0, 1, 1]

flipped_bit_index = 0                       # final answer accumulator
for k in parities.keys():                       # for each parity bit ("P1", "P2", "P3")
    indices = parities[k]["indices"]            # indices in the message that a P bit corresponds to (in P1's loop this would be [1, 3, 5, 7]
    sm = 0                                      # sum accumulator for inner loop
    for j in indices:                           # for each bit index in the indices we are checking
        sm += msg[j - 1]                            # add to sum the bit at the given index

    if sm % 2 != 0:                             # if the sum is not even
        p_number = k[1:]                            # chop of "P" from the P bit key (in P1's loop this is literally "P1" without the first character = "1")
        ordinal = (p_number) - 1                    # P1's p_number is 1, which is in index 0 of the message string...P2 in 1, P3 in 3, P4 in 7, P5 in 15...
        flipped_bit_index += 2 ** ordinal           # add 2^P to the current sum


# flipped_bit_index now contains the index of the bit that was messed up in the message we are decoding
```

Diving into the code, we see that we are actually accumulating the answer for which bit may have been flipped on the fly. For each P bit and its corresponding set of bits it checks (always including itself), we calculate the sum. If that sum is odd, we have an error in this group and **we thus know that the index of the flipped bit will have a weight of 1 in the binary representation of its index**. The more rare cases where P bits are flipped (only rare because there will be ~log2(K) P bits, where K is the total bits in the encoded message), only their own Parity check will fail (ie have an odd sum). If any other number in their list causes the failure of a given Parity check, it means by definition that it has other powers of 2 as a part of its sum, so other checks will fail too. 

By the end of looping through each P bit and their group, we do not even need any more calculation to *know* exactly where our error was and thus how to fix it. We harness the fact that a given power of two provides an otherwise unreachable dimension of addition, of a number. Whereas multiples of 2 are every other number, the only power of 2 that oscillates that frequently is 2^0. While we can add 2 some amount of times to itself to calculate any even number, certainly any power of 2, we can only represent a number in *one* way with binary. You cannot say that 8 has 3 times 2^0, we can at most say that it has 1 times 2^0. Then we have to talk about 2^1. Binary is strict, but such strictness later allows powerful information. "Restriction today gives certainty tomorrow" shows itself everywhere in programming.

#### The Beauty of the Encoding

Beautifully, the algorithm just checks and accumulates the powers of 2, the basis vectors, of the groups that fail the check. If 7 is the bit that is flipped, we know that every single check needs to have failed because 7 contains in its sum: 2^0, 2^1, and 2^2. The only way to reach 7 is with this unique combination of powers of 2. The only way to reach 3 is with the precise mix of 2^0 and 2^1, the P bits 1 and 2 (again, the indexing is frustrating). 

The code may look naive and incomplete, in that a given failed check can be so confident as to what to add to our accumulated answer. However, *by construction*, we know that this method will work, because we encoded this message with P bits that specifically correspond to powers of 2. The only way to reach a certain number is with a certain combination of powers of 2 and certain powers of two can only reach certain numbers.


This process, as you can see in [my script](https://github.com/benmeyersUSC/RandomScripts/tree/main/error_correcting_codes), which exhaustively does this process for a random message of any length, will work for all lengths of messages. The only drawback in Hamming codes is that it only works to find a single flipped bit. If a message is corrupted in more than one place, we would be out of luck. Thank god for [The Hadamard Code](https://benmeyersusc.github.io/2024/11/29/hadamard-code.html)!


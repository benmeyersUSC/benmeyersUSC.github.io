## Hamming Codes 

[(my Python script)]([hamming/HammingCodes.py](https://github.com/benmeyersUSC/RandomScripts/blob/main/hamming/HammingCodes.py#L61))

Hamming codes are a category of error-correcting codes that enable durable and dynamic sending of packets of data. Suppose I wanted to sent a byte of information over a network
to another computer: there could be any number of interferences that may cause a bit to be flipped along the way. 
Do I send it every couple seconds until I get a reply confirming
the reception? What if *that* message has a bitflip? The same goes for storing data on a drive or disk or several other states in which we want binary information to stay in tact.
How can we embed within the data some mechanism to check and correct any errors that may arise in its transit or storage?

### Enter Claude Shannon and John Hamming
While working at Bell Laboratoris, these two pioneered not only information theory, but Error-Correcting Codes. 
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

There is certainly some overlap between these groups, but they are all less than 7 in length. Suppose we implemented a similar sum-checking mechanism as discussed before, but with
respect to these groups. Say P1 can somehow correspond to that first group, P2 to the second, and P3 to the third. What can we do here? Let's revisit our proposed encoded message once more, 
but with the D bits filled in:

```python
P1 P2 1 P3 0 1 1
```

Hamming devised a scheme whereby, starting with P1, we would assign these P bits a value that ensures that the sum of their "group" is even.  

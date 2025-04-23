---
title: "CRC By Hand: A Guided Example"
excerpt: "A guide to doing CRC without stepping into the theory too much."
permalink: /posts/2025/05/01/crc-by-hand.md/
author: 'Austin Barton'
---

This is a practical guide to the Cyclic Redundancy Check (CRC). For a deep dive into the theory and math behind CRC, check out my the post Math Behind Cyclic Redundancy Check.

This post might be particularly helpful for those learning about CRC in a computer networking course.

# CRC with Example

The data is $$D = 1101011$$ and the generator is $$G = 1001$$.

## Sender and Receiver...

Both have access to the generator polynomial: $$G$$.

Both have access to the degree of $$G$$. The degree is simply the number of bits minus 1(starting from the left most nonzero bit).

$$G$$ is a polynomial over $$GF(2)$$, which means it's coefficients are either $$1$$ or $$0$$.

For example, $$1001 \to x^3 + 1$$. So, when we see that $$G$$ is a 4 bit string, we say it is of degree $$r = 3$$.

## Sender

Need to transmit data $$D$$ encoded in a message $$T$$ so that the receiver can use $$G$$ on the message it receives, which may or may not be the same as $$T$$, to check if there were any bit errors over transmission.

IDEA: Transform the data-polynomial $$D$$ into a message-polynomial $$T$$ that is divisible by the generator-polynomial $$G$$ and if any errors occur (changing the coefficients of the message-polynomial) then the received message-polynomial won't be divisible by $$G$$ (to some extent).

If we use a generator $$G$$ with $$r+1$$ bits, then we can detect burst errors less than or equal to $$r$$ bits. In our example, this means we can detect up to $$3$$ bit burst errors. This is what I mean by to some extent.

Walkthrough...

$$D = 1101011 \to x^6 + x^5 + x^3 + x + 1$$.

Now shift $$D$$ $$r$$ times:

$$D\cdot 2^r = 1101011000 \to x^9 + x^8 + x^6 + x^4 + x^3$$.

And now we divide $$D\cdot 2^r$$ by $$G$$ to calculate the remainder $$R$$. Note we are working over $$GF(2)$$, so we can simplify this by treating it like bitwise XOR operations.

```text
1 1 0 1 0 1 1 0 0 0

1 0 0 1

----------

0 1 0 0                                    <- XOR result. Left most is 0, so drop it and carry down next bit.

  1 0 0 0                                  <- leftmost bit is 1. Now XOR with G again.

  1 0 0 1                                  <- and rinse and repeat...

  --------

  0 0 0 1

    0 0 1 1 

      0 1 1 1 

        1 1 1 0

        1 0 0 1 

        --------

        0 1 1 1 

          1 1 1 0

          1 0 0 1

          --------

          0 1 1 1

              1 1 1 0

              1 0 0 1

              --------

              0 1 1 1
```

Our remainder's degree will be one less than the degree of the generator, or 1 less bit. Take $$r$$ bits from $$R$$ and we have $$R = 111 \to x^2 + x + 1$$.

Now craft the message-polynomial as such: 

```math
T = D || R = D\cdot 2^r + R = 1101011111 \to x^9 + x^8 + x^6 + x^4 + x^3 + x^2 + x + 1
```

$$T$$ is now divisible by $$G$$ and through some things explained only by mystical abstract algebra stuff, if you alter the coefficients of $$T$$, it won't be divisible by $$G$$ (to some extent).

Receiver

Simply take your received message, $$T'$$ we will call it, and divide by $$G$$. If $$T'/G = 0$$, then we detect no errors and it's very likely that $$T' = T$$.

If $$T'/G \neq 0$$ then we KNOW that $$T' \neq T$$, and thus, there was an error.

If there is no error, you retrieve $$D$$ from $$T$$ through simply stripping off the $$r$$ most bits. In our case, you strip off the $$3$$ most bits.



### No Errors

```text
1 1 0 1 0 1 1 1 1 1

1 0 0 1

----------

0 1 0 0 

  1 0 0 0

  1 0 0 1

  --------

  0 0 0 1 

    0 0 1 1 

      0 1 1 1

        1 1 1 1 

        1 0 0 1

        --------

        0 1 1 0

            1 1 0 1 

            1 0 0 1

            -------

            0 1 0 0

              1 0 0 1

              1 0 0 1

              --------

              0 0 0 0 
```

$$R = 0$$. No error detected.



### Errors

```text
1 0 1 1 0 1 1 1 1 1

1 0 0 1

--------

0 0 1 0  

  0 1 0 0  

    1 0 0 1  

    1 0 0 1  

    --------

    0 0 0 0  

        0 0 0 1  

            0 0 1 1 

              0 1 1 1 

                  1 1 1 1

                  1 0 0 1

                  --------

                  0 1 1 0
```

$$R = 110$$. Error detected.

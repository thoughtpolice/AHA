% CRYPTOGRAPHY OMG
% Austin Seipp
% AHA 0x0043 - April 26th, 2012

# Quick Prep

## What this talk is:

 * A quick intro to two cryptographic libraries
 * Comparisons between them from the perspective of speed,
   ease-of-use, and robustness

## What it isn't:

 * An introduction to cryptography
 * Theoretical
 * Boring (I hope)

# What's with cryptographic libraries, anyway?

You basically want 3 things from a cryptographic library:

> 1. Security and robustness - Not secure? You're screwed
> 2. Ease-of-use - Not easy to use? Easy to mess up - you're screwed
> 3. Speed - Not fast? Too computationally expensive - you're screwed

All of these are crucially important for mainstream cryptographic use.

# Case study: OpenSSL - Security

Is OpenSSL secure? Well, it's certainly *battle tested*.

# Case study: OpenSSL - Ease-of-use

Is OpenSSL easy to use? Fuck. No.

# Case study: OpenSSL - Speed

Is OpenSSL fast?

... kinda. More on this shortly

# A new hope: NaCl

# Case study: NaCl - Security

# Case study: NaCl - Ease-of-use

# Case study: NaCl - Speed

# Conclusion

 * Cryptographic APIs can be drastically simpler, safer and faster
    * NaCl raises the bar for ease, security and speed

 * Don't fuck with crypto unless you do lots, and lots, and *lots* of research.
    * Or use an idiot-proof API (`crypto_box` wins again!)

# el fin

Any questions? No? Didn't think so.

XOXOXOXOXOs, death threats, hot chicks: <mad.one@gmail.com>

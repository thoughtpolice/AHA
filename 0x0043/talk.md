% CRYPTOGRAPHY OMG or, rot13(rot13("CRYPTOGRAPHY OMG"))
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

1. Security and robustness - Not secure? You're screwed
2. Ease-of-use - Not easy to use? Easy to mess up - you're screwed
3. Speed - Not fast? Too computationally expensive - you're screwed

All of these are crucially important for mainstream cryptographic use.

# Case study: OpenSSL - Security

Is OpenSSL secure? Well, it's certainly *battle scarred*.

Already 9 vulnerabilities in 2012. Most recent: Tavis Ormandy's recent
integer conversion err/heap overflow in ASN.1 BIO/FILE parsing routines;
user-supplied RSA public keys or X.509 certificates could lead to RCE.

Attack surface: small (`libssl` doesn't use BIO/FILE parsing for its
certificate API; RSA key parsing occurs in-memory, not via BIO/FILE.)
Only affects a small subset of ASN.1 functions.

Still a "textbook heap overflow" by most means.

<http://lists.openwall.net/full-disclosure/2012/04/19/4>

Vulnerability list (not yet updated with Tavis' flaw): <http://www.openssl.org/news/vulnerabilities.html>

# Case study: OpenSSL - Security (pt. 2)

Side-channel attacks are quickly becoming the predominate way to break
cryptographic *implementations* (not *primitives* - an important
distinction.)

It's speculated `<insert three-letter organization>` has means of
breaking much mainstream cryptography; but it's likely these are
through implementation flaws and side channels, not primitive failures
in the algorithms (otherwise AES-256 wouldn't be used for TOP SECRET
docs.)

Dan Bernstein demonstrated complete key recovery for OpenSSL's AES
implementation *over a network link*, via timing attacks & statistical
correlation: <http://cr.yp.to/antiforgery/cachetiming-20050414.pdf>

Be afraid - be very, very afraid.

# Case study: OpenSSL - Ease-of-use

Is OpenSSL easy to use? Not particularly, a lot of the
`libssl`/`libcrypto` APIs have horrible documentation, and the APIs
themselves are complex. Design seems to be mostly unspecified.

Read em' and weep: <http://h71000.www7.hp.com/doc/83final/ba554_90007/ch04s03.html>

How many failure paths and error conditions? Too many - humans err
frequently in any case.

Given enough choice, you'll invariably choose to shoot yourself in the
face whether you want to or not.

# Case study: OpenSSL - Speed

Is OpenSSL fast?

... kinda. More on this shortly.

# A new hope: NaCl

NaCl, the **N**etworking **a**nd **C**ryptography **l**ibrary
(pronounced "salt") is a new library for cryptographic primitives
designed by Dan J. Bernstein (qmail/djbdns fame.) It uses elliptic
curve cryptography.

Home page: <http://nacl.cace-project.eu/>

Designed to raise the bar on cryptographic software APIs on all
fronts. Provides authenticated encryption, decryption, streaming
encryption, (one time) authentication, signatures, etc. Stronger tools
and protocols can be built using it.

# Case study: NaCl - Security

NaCl is designed with security in mind:

 * Systematic avoidance of all branch conditions/loads/stores on secret data.
 * All operations are constant time; none proportional to length of input or 
   proportional to key differences.
 * No dynamic memory allocation.
 * Immune to timing attacks
 * This immunity is verified mechanically, using a variant of valgrind
   known as 'ctgrind' (<http://github.com/agl/ctgrind>)

Read this casual, epic paper:
<http://cr.yp.to/highspeed/coolnacl-20111201.pdf>

# Case study: NaCl - Ease-of-use

~~~~~~{.c}
// generate keys
int crypto_box_keypair(uint8_t* pk,uint8_t* sk);
// authenticated encryption
int crypto_box(uint8_t* cipher_out, uint8_t* msg,
               unsigned long mlen, uint8_t* nonce,
			   uint8_t* pubkey, uint8_t* seckey);
~~~~~~

All functions either succeed and return 0, or fail and return non
0. No trinary error modes, complex interfaces. Pretty much every API
looks just like this and everything has predetermined lengths.

# Case study: NaCl - Speed

Much of the optimized NaCl code is machine generated, and most of it
verified as previously explained.

The library benchmarks itself at build time, in order to select the
fastest primitives for your machine on a per-CPU basis (corollary: not
in your package manager.)

# Case study: NaCl - Speed (pt 2)

Benchmark: Streaming encryption - <http://thoughtpolice.github.com/salt/bench/results.html#b28>

 * XSalsa/20 (NaCl) vs AES-128-CTR (OpenSSL)
 * 1 gigabyte file of random data pulled from `/dev/urandom` (sits in disk cache during benchmark)

TL;DR version: NaCl's **4x faster** than OpenSSL: 3s vs 12s. That's a
difference of **88 megabytes per second** vs **350 megabytes per
second**.

NB: that's using *one core*. My MBP has 8 hyperthreaded cores @ 2.2 gHz, meaning
a saturated CPU could theoretically achieve about 16 gigabits per
second, or 2 gigabytes per second. That'll saturate your fancy link
~reality:\ I/O\ or\ memory\ performance\ is\ going\ to\ tank\ long\ before\ this.~

NB: benchmarks are statistically accurate, using statistical bootstrap to provide confidence
intervals in the face of outliers like process creation/context switches.

# Pitfalls

 * Most cryptographic APIs require you generate the nonces. There is
   no way to abstract this generically enough behind the API to be
   useful to everyone. Nonce creation should be carefully thought of,
   as it's a matter of cryptographic concern.
 * Most of the encryption modes supported may not potentially abide by
   e.g. FIPS requirements. For example, there currently isn't an
   authenticated encryption mode that uses the NIST P-256 curve or any
   NIST curves; so you may not be able to use these if you work for
   the government or elsewhere. Tread carefully.
 * Most of the optimized implementations are machine generated
   assembly; they also aren't provided in a relocatable form; thus,
   unsuitable for dynamic libraries. The C reference implementations
   will work however, at the expense of speed.

# Conclusion

 * Cryptographic APIs can be drastically simpler, safer and faster
    * NaCl raises the bar for ease, security and speed
 * Should go without saying but, don't fuck with crypto unless you do
   lots, and lots, and *lots* of research.
    * Or use an idiot-proof API (`crypto_box` wins again!)

Go write bindings for `<my favorite programming language>`

# el fin

Any questions? No? Didn't think so.

Slides online: <http://hacks.yi.org/rants.html>

XOXOXOXOXOs, death threats, et cetera: <mad.one@gmail.com> or tweet me `@stdlib`

# el fin

Thanks for listening


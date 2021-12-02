# Erlang based HMAC-based Extract-and-Expand Key Derivation Function (HKDF)

See [RFC 5869](https://tools.ietf.org/html/rfc5869) for details. This
is a very straigth forward implementation, including the tests from
RFC5869 as eunit tests. It supports the hash functions md5, sha,
sha224, sha256, sha384, sha512.

Updated to Erlang/OTP 24 and rebar3.

## Usage

- Hash = md5 | sha | sha224 | sha256 | sha384 | sha512,
- IKM = input keying material,
- salt = optional salt value (a non-secret random value),
- info = optional context and application specific information,
- L = length of output keying material in octets.

```
~/hkdf$ rebar3 shell
===> Verifying dependencies...
===> Analyzing applications...
===> Compiling hkdf
Erlang/OTP 24 [erts-12.1.5] [source] [64-bit] [smp:2:2] [ds:2:2:10] [async-threads:1] [jit]

Eshell V12.1.5  (abort with ^G)
```
Derive a 4096 bytes long key from the secret `some secret` (11 bytes) using the hash256 algorithm:
```
1> hkdf:derive_secrets(<<"some sercret">>, 4096). 
<<79,20,73,77,43,236,55,35,153,91,214,230,196,243,154,42,
  202,253,10,124,228,154,130,235,132,49,153,104,9,...>>
```
The maximum derived key length depends on the hash algorithm used:

| Hash Alg.  | Max Length (L) |
|---|---|
| md5 | 4080 |
| sha | 4080 |
| sha224 | 7140 |
| sha256 | 8160 |
| sha384 | 12240 |
| sha512 | 16320 |

Now use some other hash algorithm:
```
2> hkdf:derive_secrets(sha, <<"some sercret">>, 4096).   
{error,max_derived_length_exceeded}
3> hkdf:derive_secrets(sha, <<"some sercret">>, 2048).
<<141,159,239,143,169,156,148,94,117,93,76,97,232,102,45,
  163,245,17,144,234,186,78,169,12,1,140,191,236,27,...>>
```
Now, with some info added. Info is used to set a context such as
`encryption`, `message authentication`, `WhisperMessageKeys` or whatever you like.
```
4> hkdf:derive_secrets(sha512, <<"some sercret">>, <<"encryption">>, 2048).
<<81,235,47,134,224,128,85,22,18,245,67,75,151,30,104,103,
  55,19,238,148,95,253,8,0,56,139,136,105,207,...>>
5> hkdf:derive_secrets(sha512, <<"some sercret">>, <<"UsedInMyApplication">>, 2048).
<<97,51,54,191,163,55,231,156,65,248,186,24,46,201,234,
  178,98,121,37,55,93,243,214,27,136,7,181,120,52,...>>
```
Add some salt:
```
6> hkdf:derive_secrets(sha512, <<"some sercret">>, <<"MyApplication">>, <<"lots of salt here">>, 2048).
<<86,124,101,141,121,180,89,23,115,176,45,80,60,10,88,157,
  32,249,52,19,231,142,32,74,103,55,161,243,207,...>>
```
Input key material, info and salt all are binaries, i.e. a number of bytes.

NB: do read [Understanding
HKDF](https://soatok.blog/2021/11/17/understanding-hkdf/) on the
proper use of the `Info` argument and why and when to use salt. If the
IKM is not random and/or has some structure as it has when taken from
[Elliptic Curve] Diffie-Hellman for example, you must either hash the
IKM yourself before using it or add some salt.

You can also directly call the underlying functions `extract/2-3` and
`expand/3-5`, which is done in the tests to verify the intermediate
results.

```
PKR = hkdf:extract(Hash, IKM).
```
or 
```
PKR = hkdf:extract(Hash, IKM, Salt).
```
Next, expand the PKR to the required length

```
OKM = expand(Hash, PRK, Info, L)
```

## Tests

[Appendix A in RFC 5869](https://datatracker.ietf.org/doc/html/rfc5869#appendix-A) lists some tests that you can run:
```
~/hkdf$ rebar3 eunit
===> Verifying dependencies...
===> Analyzing applications...
===> Compiling hkdf
===> Performing EUnit tests...
.......
Finished in 0.150 seconds
7 tests, 0 failures

```

## Further reading

- [Understanding
  HKDF](https://soatok.blog/2021/11/17/understanding-hkdf/)
- [RFC 5869 - HMAC-based Extract-and-Expand Key Derivation Function (HKDF)
](https://datatracker.ietf.org/doc/html/rfc5869)

# Erlang based HMAC-based Extract-and-Expand Key Derivation Function (HKDF)

See [RFC 5869](https://tools.ietf.org/html/rfc5869) for details. This is a very straigth forward implementation, 
including the tests from RFC5869 as eunit tests. It supports the hash functions md5, sha, sha224, sha256, sha384, sha512.

## Usage

- Hash = md5 | sha | sha224 | sha256 | sha384 | sha512.
- IKM = input keying material.
- salt = optional salt value (a non-secret random value)
```
PKR = hkdf:extract(Hash, IKM).
```
or 
```
PKR = hkdf:extract(Hash, IKM, Salt).
```
Next, expand the PKR to the required length

- info = optional context and application specific information,
- L = length of output keying material in octets.

```
OKM = expand(Hash, PRK, Info, L)
```

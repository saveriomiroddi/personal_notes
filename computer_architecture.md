# Compute architecture

- [Compute architecture](#compute-architecture)
  - [Data types](#data-types)
  - [IEEE754](#ieee754)

## Data types

The `word` size depends on the architecture!!

Assuming a 16-bit arch, the 128-bit size is `long word`.

## IEEE754

Precision bits (sgn, exp, mant)

- Single precision bits: 1 + 8  + 23
- Double precision bits: 1 + 11 + 52

Exponent: signed
Mantissa: 1.0 + 0.5^(left position 1-based); there are special cases

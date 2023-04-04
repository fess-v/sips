# Preamble

SIP Number: 02X

Title: Standard Trait Definition for Smart Contract Wallet External Calls 

Author: Vladislav Bespalov <https://github.com/fess-v>

Consideration: Technical

Type: Standard

Status: Ratified

Created: 4 April 2023

License: CC0-1.0

Sign-off: -

# Abstract

All smart contract wallet types, including multisignature wallets, DAOs, general purpose smart contract wallets, need to have a possibility to interact with other contracts on the blockchain network.

Clarity language has its own type and static analyzer restrictions. These restrictions include a need to have predefined traits for external contract calls and a need to have a passed trait instead of a contract principal because it can't be up-casted to a trait. 
Therefore, the only way to call any external smart contract from a smart contract wallet is to pass from a user that external contract principal as a trait which was also predefined in the smart contract wallet.

It opens a task which this proposal aims to solve - creation of a standard interface which could be used as a predefined trait which other smart contracts in Stacks could support to be able to be integrated with such smart contract wallets.

# License and Copyright

This SIP is made available under the terms of the Creative Commons CC0 1.0 Universal license, available at https://creativecommons.org/publicdomain/zero/1.0/
This SIP’s copyright is held by the Stacks Open Internet Foundation.

# Introduction

A general purpose smart contract wallet can't be created for now in Stacks because of Clarity restrictions on external smart contract calls. But it's definitely possible to make smart contract wallets which will be secure and covering 99.99% use cases, therefore it's important to make it possible for them to emerge.
Smart contract wallet standards will open this field for developers and make dApps integrations faster and easier.

# Specification

All external dApps which would like to be integrated with smart contract wallets, should have executor interfaces created - either for each function a separate executor interface, or one for all of them with serialized action added into the last buffer argument:

```
(define-trait executor-trait
  (
    (execute ((list 10 <sip-010-trait>) (list 10 <sip-009-trait>) (buff 2000)) (response bool uint))
  )
)
```
`TODO: arguments should be finalized i.e. array sizes, potential traits except token standards`

`execute` takes 3 arguments:
1. The first argument is an array of NFT traits which could be used for further external calls.
2. The second argument is an array of Fungible Token traits which also could be used for further external calls.
3. The third argument is a serialized buffer for possible parameters for this call, for instance, action type or parameters for swapping action on a DEX. Inside executors this buffer can be parsed using standard `from-consensus-buff?` function.

NFT and Fungible Token traits cannot be as well serialized because of the aforementioned Clarity language restriction for up-casting principals to traits.
These function arguments should be enough for covering most current use cases for smart contract wallets.

# Reference Implementations

## Executor contract example

Here's an example of a separate executor interface for stx-transfer from a smart contract wallet.

```
(impl-trait .executor-trait)
(use-trait sip-009-trait .sip-009-trait)
(use-trait sip-010-trait .sip-010-trait)

(define-public (execute (tokens (list 10 <sip-010-trait>)) (nfts (list 10 <sip-009-trait>)) (variables (buff 1000)))
  (let
    (
      (parsed-variables (unwrap! (from-consensus-buff? {amount: uint, recipient: principal} variables) (err u1)))
    )
    (stx-transfer? (get amount parsed-variables) tx-sender (get recipient parsed-variables))
  )
)
```

## Executor contract call from a smart contract

`TODO: add references`

## Full dApp integration example 

`TODO: add references`

# Related Work

- _[MultiSafe](https://github.com/Trust-Machines/multisafe)_ is the first multisignature wallet in Stacks, which has a simple version of the executor concept implemented and was used as an inspiration for the current proposal.
- _[SIP-009](https://github.com/stacksgov/sips/blob/main/sips/sip-009/sip-009-nft-standard.md)_ is a standard trait definition for non-fungible tokens in Stacks.
- _[SIP-010](https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md)_ is a standard trait definition for fungible tokens in Stacks.

# Backwards Compatibility

Not applicable

# Activation

--



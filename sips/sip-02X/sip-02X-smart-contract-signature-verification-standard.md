# Preamble

SIP Number: 02X

Title: Standard Trait Definition for Smart Contract Signature Validation

Author: Vladislav Bespalov <https://github.com/fess-v>

Consideration: Technical

Type: Standard

Status: Ratified

Created: 3 April 2023

License: CC0-1.0

Sign-off: -

# Abstract

Signatures are widely used by many dApps within the blockchain ecosystem.
Mostly signatures are created using externally owned private keys and could be verified by an accepted way in
_[SIP018](https://github.com/stacksgov/sips/blob/main/sips/sip-018/sip-018-signed-structured-data.md)_, but smart contract wallets do not possess a private key.
This proposal provides a standard which could be used by dApps and smart contract wallets for signature verification.


# License and Copyright

This SIP is made available under the terms of the Creative Commons CC0 1.0 Universal license, available at https://creativecommons.org/publicdomain/zero/1.0/
This SIP’s copyright is held by the Stacks Open Internet Foundation.

# Introduction

Lots of decentralized applications utilize message signatures for their logic, mostly for interaction with the backend. For instance, NFT marketplaces, decentralized exchanges, authorization purposes, and many more. Smart contract wallets can be used as a second security layer for controlling and managing assets, DAOs, and other contracts (especially multisignature ones) and therefore integration should be made possible between wallets and dApps. For this we need a standard applied for signatures verification.



# Specification

Every smart contract wallet which has the ability to sign messages in Stacks should implement the following trait:

```
(define-trait sip-02X-signature-verifier-trait
  (
    ;; Arguments: 1. sha256 message hash 2. signature buff (or signatures for multisig cases)
    (is-valid-signature ((buff 32) (buff 2000)) (response bool uint))
  )
)
```
`TODO: calculate the optimal buff size for the signatures argument`

`is-valid-signature` takes a sha256 hashed message following _[SIP018](https://github.com/stacksgov/sips/blob/main/sips/sip-018/sip-018-signed-structured-data.md)_ and a signature buffer, which could be used by multisignature wallets or DAOs for a more complex signature structure with multiple owners.
This function returns a boolean value whether a given signature is valid or not. 
This function must never modify state or return an error response. It must be defined as `define-read-only`.

For reducing the call costs by not increasing the `buff` size to possible thousands, this function accepts a sha256 hashed buffer of the message instead.

Smart contract and normal wallet principals could be easily distinguished therefore it is clear when to call this verification method or to use a wallet's public key for that. 

# Message verification usage

Here's a simple example on how this standard could be used in a Clarity smart contract.

```
(use-trait sip-02X-signature-verifier-trait .sip-02X-signature-verifier-trait)

(define-constant ERR-UNWRAP-OWNER (err u1))
(define-constant ERR-UNWRAP-CALL (err u2))
(define-constant ERR-SIGNATURE-VERIFICATION (err u3))

(define-data-var owner principal tx-sender)

(define-public (change-owner (new-owner principal) (signature (buff 2000)) (verifier <sip-02X-signature-verifier-trait>))
    (let 
        ((hashed-owner (sha256 (unwrap! (to-consensus-buff? new-owner) ERR-UNWRAP-OWNER))))
        (asserts! (unwrap! (contract-call? verifier is-valid-signature hashed-owner signature) ERR-UNWRAP-CALL) ERR-SIGNATURE-VERIFICATION)
        (var-set owner new-owner)
        (ok true)
    )
)
```


# Related Work

- _[EIP1271](https://eips.ethereum.org/EIPS/eip-1271)_ is a standard for EVM compatible chains for Solidity language, which was used as a base for the current proposal.
- _[SIP018](https://github.com/stacksgov/sips/blob/main/sips/sip-018/sip-018-signed-structured-data.md)_  is a standard in Stacks which is applied for signatures verification for non-smart-contract wallets.

# Backwards Compatibility

Not applicable

# Activation

--

# Reference Implementations

## Multisig wallet

`TODO: add Asigna github instead of copypasting code`

`TODO: finalize signature method, select appropriate signature buff size`

```
(define-constant SIGNATURE-SIZE u65)
(define-constant OWNER-SIGNATURE-SIZE u22)
(define-constant FULL-SIGNATURE-SIZE u87)

(define-private (get-signatures-with-owners-from-buff (owner-to-check uint) (state { signature: (buff 2000), owners: (list 20 principal), signatures: (list 20 (buff 65)) }))
    (
        let (
            (start (* owner-to-check FULL-SIGNATURE-SIZE))
            (signature (unwrap! (as-max-len? (unwrap! (slice? (get signature state) start (+ start SIGNATURE-SIZE)) state) u65) state))
            (owner (unwrap! (from-consensus-buff? principal (unwrap! (slice? (get signature state) (+ start SIGNATURE-SIZE) (+ start FULL-SIGNATURE-SIZE)) state)) state))
        )
        (merge state {
            owners: (unwrap! (as-max-len? (append (get owners state) owner) u20) state),
            signatures: (unwrap! (as-max-len? (append (get signatures state) signature) u20) state)
        })
))

(define-read-only (is-valid-signature (message-hash (buff 32)) (signature (buff 2000))) 
    (let (
        (signature-len (len signature))
        (number-of-signers (/ signature-len FULL-SIGNATURE-SIZE))
        (it (unwrap! (slice? ITERATOR u0 number-of-signers) ERR-UNWRAP-ITERATOR))
        (signatures-with-owners (fold get-signatures-with-owners-from-buff it {signature: signature, owners: (list), signatures: (list)}))
        (owners-signed (get owners signatures-with-owners))
        (resulted-owners (get res (fold process-signature owners-signed {message-hash: message-hash, signatures: (get signatures signatures-with-owners), owners-signed: owners-signed, res: (list)})))
        )
        (ok (>= (len resulted-owners) (var-get threshold)))
))
```

## Smart contract wallet

`TODO: add references`

-- 

# Fedimint Lightning Contracts

Fedimint presently uses two specialized lightning smart contracts for moving funds between parties to in a transaction. Funds can move in / out of the federation through a lightning gateway.

## The Outgoing Contract

> **Source**: https://github.com/fedimint/fedimint/blob/7ef972e11b92018899e7e15eb6fc649a13801538/modules/fedimint-ln-common/src/contracts/outgoing.rs#L16-L33

A user locks up funds (timelock) that can be claimed by a lightning gateway that pays the invoice and thus receives the preimage to the payment hash and can thereby prove the payment.

If the gateway is not able to do so before the timelock expires the user can claim back the funds.

```rs
pub struct OutgoingContract {

  /// Hash that can be used to spend the output before the timelock expires

  pub hash: bitcoin_hashes::sha256::Hash,

  /// Block height at which the money will be spendable by the pubkey

  pub timelock: u32,

  /// Invoice containing metadata on how to obtain the preimage

  pub invoice: lightning_invoice::Invoice,

  /// Public key of the user that can claim the money back after the timelock expires

  pub user_key: secp256k1::XOnlyPublicKey,

  /// Public key of the LN gateway allowed to claim the HTLC before the timelock expires

  pub gateway_key: secp256k1::XOnlyPublicKey,

  /// Flag that can be set by the gateway and allows the client to claim an early refund

  pub cancelled: bool,

}
```

> **Issue**: We presently use a full invoice with description. Not good for fedimint level privacy.
> 
> **Improvement**: Need to use pruned, privacy friendly version without description

## The Incoming Contract

> **Source**: https://github.com/fedimint/fedimint/blob/7ef972e11b92018899e7e15eb6fc649a13801538/modules/fedimint-ln-common/src/contracts/incoming.rs#L57-L69

A user generates a private/public keypair that can later be used to claim incoming funds. The pubkey is used as preimage of a payment hash  - which is threshold encrypted to the federations pubkey.

After encryption, the user puts up the encrypted preimage for sale by creating an [`IncomingContractOffer`](https://github.com/fedimint/fedimint/blob/7ef972e11b92018899e7e15eb6fc649a13801538/modules/fedimint-ln-common/src/contracts/incoming.rs#L13)

A lightning gateway can claim an incoming HTLC by locating the offer, and using it to buy the preimage. They need to transfer funds (e-cash) into the incoming contract as the only way to activate threshold decryption process inside the federation. See [`FundedIncomingContract`](https://github.com/fedimint/fedimint/blob/7ef972e11b92018899e7e15eb6fc649a13801538/modules/fedimint-ln-common/src/contracts/incoming.rs#L76)

User could have threshold-encrypted useless data in the contract, so there are two possible outcomes:
- **Data is a valid preimage**:
  - gateway pays the HTLC using valid preimage
  - user claims funds from the contract by signing using the private key corresponding to the pubkey (used as preimage!)

- **Data is an invalid preimage**:
  - gateway cancels the incoming HTLC
  - gateway claims back the money they just locked in the incoming contract

```rs
pub struct IncomingContract {

  /// Payment hash which's corresponding preimage is being sold

  pub hash: bitcoin_hashes::sha256::Hash,

  /// Encrypted preimage as specified in offer

  pub encrypted_preimage: EncryptedPreimage,

  /// Status of preimage decryption, will either end in failure or contain the preimage eventually.
  /// In case decryption was successful the preimage is also the public key locking the contract,
  /// allowing the offer creator to redeem their money.

  pub decrypted_preimage: DecryptedPreimage,

  /// Key that can unlock contract in case the decrypted preimage was invalid

  pub gateway_key: secp256k1::XOnlyPublicKey,

}
```

> **Issue**: Protocol uses a pubkey as preimage in the contract. This is bad for privacy since pubkeys are distinguishable from randomness. Payer would learn the recipient is using a federated mint.
>
> **Improvement**: Hash the pubkey (preimage) before encryption? encrypt preimage to LN gateway?

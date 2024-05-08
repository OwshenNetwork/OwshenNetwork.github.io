# Owshen Core

Owshen core consist of wallet generation and proof generation section and also its has proper HTTP server that serve in local port.

# Key Structures

1. Points on an Elliptic Curve:

```rust=
pub struct Point {
    pub x: Fp,
    pub y: Fp,
}
```

Represents a point on an elliptic curve. Each point consists of two coordinates, x and y, defined over a finite field Fp.

Points are fundamental in ECC and are used to represent public keys and other elements necessary for cryptographic operations

2. Private and Public Keys:

```rust=
pub struct PrivateKey {
    pub secret: Fp,
}
```

```rust=
pub struct PublicKey {
    pub point: Point,
}
```

- PrivateKey: Stores a single scalar value (secret) which acts as a private key.
- PublicKey: Derived from the private key by scalar multiplication of a base point on the elliptic curve. It is essentially a point on the curve.

3. Ephemeral Keys:

```rust=
pub struct EphemeralPrivKey {
    pub secret: Fp,
}
```

```rust=
pub struct EphemeralPubKey {
    pub point: Point,
}
```

- Ephemeral Keys: These keys are temporary and used for a single transaction or session to enhance security. They follow the same structure as permanent keys but are not stored long-term.

## Cryptographic Operations

### Key Derivation and Elliptic Curve Operations:

```rust=
impl PublicKey {
    pub fn derive(&self, r: Fp) -> (EphemeralPrivKey, EphemeralPubKey, PublicKey) {
        let ephemeral = *BASE * r;
        let shared_secret = self.point * r;
        let shared_secret_hash = hash4([shared_secret.x, shared_secret.y, 0.into(), 0.into()]);
        let pub_key = self.point + *BASE * shared_secret_hash;
        (
            EphemeralPrivKey { secret: r },
            EphemeralPubKey { point: ephemeral },
            Self { point: pub_key },
        )
    }
}

```

- The derive method generates ephemeral private and public keys along with a new derived public key. This operation uses a random scalar r, multiplies it with the base point to generate an ephemeral public key, and then uses a hash function to secure the transaction further.

### Shared Secrets and Stealth Addresses:

```rust=
impl EphemeralPrivKey {
    pub fn shared_secret(&self, pk: PublicKey) -> Fp {
        let shared_secret = pk.point * self.secret;
        hash4([shared_secret.x, shared_secret.y, 0.into(), 0.into()])
    }
}

```

- Calculates a shared secret using the ephemeral private key and a public key. This is crucial for generating stealth addresses, allowing for secure and private transactions.

## Entropy and Mnemonic Generation

```rust=
impl Entropy {
    pub fn generate<R: Rng>(rng: &mut R) -> Self {
        Self { value: rng.gen() }
    }

    pub fn to_mnemonic(&self) -> Result<String, bip39::Error> {
        let mnemonic: Mnemonic = Mnemonic::from_entropy(&self.value)?;
        let words: Vec<&str> = mnemonic.word_iter().collect::<Vec<&str>>();
        let phrase: String = words.join(" ");
        Ok(phrase)
    }
}

```

- Handles the generation of entropy and its conversion to a mnemonic phrase, following the BIP39 standard. This feature is critical for user-friendly secure backup and recovery of private keys.

# Dive + Mint & Burn

In the Owshen protocol, the DIVE token is an ERC-20 token with specialized features for minting and burning tokens through privacy-preserving proofs. This functionality is critical for maintaining token supply integrity while supporting various operations such as token recovery and privacy-enhanced transactions.

## Overview of DIVE Token Functionalities

The DIVE token integrates advanced cryptographic features, including mint and burn capabilities, which are controlled through the use of zero-knowledge proofs to ensure transactions are validated without exposing sensitive details on the blockchain.

it has a practical implementation of [EIP-7503](https://eip7503.org/). It's a relatively minimal ERC-20 smart-contract, deployed on Ethereum blockchain, allowing people to provide private proofs of burn and mint Dive tokens in exchange. The minting is done in a 1:1 scale, which means, for each 1 ETH you burn, you'll get 1 Dive.

It uses zkSNARKs under the hood to validate the proof-of-burns. The zero-knowledge protocol argues that there is an account within the state-root of a blockRoot (Which is a public bytes32 value, that can be accessed in smart-contracts by: block.blockRoot or blockroot(idx), and can be fed as a public input to zero-knowledge proof circuits), with an unspendable address (I.e burn-address). The circuit checks the unspendability by checking if the address is in fact equal with the output of a hash-function.

1. Minting Process:

   Minting in the DIVE ecosystem is uniquely tied to the burning of another asset or a previous version of a DIVE token to ensure the total supply is conserved or appropriately regulated. This process is supported by cryptographic proofs that validate the burn without revealing the identity or amount to the public blockchain network.

2. Burning Process:

   Burning tokens is an essential feature for users who wish to exit certain states or convert tokens into other forms while ensuring that the action is recorded and irreversible on the blockchain.

### Cryptographic Proof Integration in Minting

The minting process is intricately designed to require a proof of the previous token's burn, leveraging Ethereum's capabilities to interact with advanced cryptographic proofs.

- `Proof of Burn`: Utilizes zero-knowledge proofs to verify that tokens were burnt without revealing any sensitive information. This method uses the mpt_last_prove and mpt_path_prove functions to generate and verify proofs of the previous token states and their transitions.

- `Interaction with Ethereum Smart Contracts`: The mint function integrates with Ethereum smart contracts through the ethers-rs library, handling blockchain interactions seamlessly.

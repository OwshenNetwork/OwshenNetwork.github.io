Previously said, a Sparse-Merkle-Tree is being maintained in the Owshen platform's smart-contract, where each leaf is:

\\(hash({pub}\_x,{pub}\_y,token,amount)\\)

One can spend/withdraw a coin in the merkle-tree by proving:

> I have a private key \\(s\\) (Private), where there is a leaf in tree with public-key \\(s \times G\\), holding \\(amount\\) of \\(token\\).

After each send, an event will be emitted, providing the data needed for the receiver to recognize his incoming transactions:

```solidity=
event Sent(
    Point ephemeral, // g^r
    uint256 index,
    uint256 timestamp,
    uint256 hint_tokenAddress, // token + hash(g^sr)
    uint256 hint_amount, // amount + hash(g^sr)
    uint256 commitment
);
```

The shared secret between the sender and receiver is \\(hash(g^{sr})\\). We can add the shared-secret to the token-id and amount in order to obfuscate them. (\\(p\\) is the field-size)

\\({token}\_{encoded} = ({token} + hash(g^{sr})) \mod p\\)

\\({amount}\_{encoded} = ({amount} + hash(g^{sr})) \mod p\\)

The receiver may subtract the shared secret from the token/amount to calculate the leaf's actual token/amount and try to calculate the commitment. If the commitment he has calculated is equal to the commitment submitted on-chain, then the coin is for him, and he can derive the private-key needed for spending that coin.

There are another events for Deposit and Spend:

```solidity=
 event Spend(uint256 nullifier);
```

For Deposit, we update tree with new leaf and then initiate new Sent event with proper amount and token address (for deposit token address and amount are not obfuscated)

The Spend event just notify nullifiers that are spend in withdrawal and send mechanism.

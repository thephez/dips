Mobile Wallet PrivateSend support (DRAFT)
=========================================

This document discusses the possibility of having a non-Evolution option for PrivateSend in a mobile wallet.


## Basic case (Dash Core as PrivateSend proxy)

Based on the description [here](https://mydashwallet.org/AboutPrivateSend), this is very similar to what MyDashWallet does (although here a trusted Dash Core node would be used - your own). In this case, the Dash Core full node actually issues the PrivateSend transaction on behalf of the mobile wallet.

Building a service to receive the destination address from the mobile wallet would be the main thing to develop for this (potentially already exists in [MyDashWallet's repo?](https://github.com/DeltaEngine/MyDashWallet)). Current RPC commands already enable monitoring [wallet balance](https://dash-docs.github.io/en/developer-reference#getbalance), [controlling mixing (start/stop)](https://dash-docs.github.io/en/developer-reference#privatesend), and [sending PrivateSend transactions](https://dash-docs.github.io/en/developer-reference#sendtoaddress).

1. Mobile wallet sends enough funds to the Dash Core trusted proxy to cover mixing collateral/fees and the amount to send

2. Mobile wallet also provides a destination address and amount to be sent to it

3. Dash Core trusted proxy sends a PrivateSend transaction:
 - Immediately if it contains a sufficient balance
 - Otherwise, it mixes until a sufficient PrivateSend balance exists for the transaction

Any remaining fund are either refunded or mixed so there is a liquidity balance for a future transaction.


## Advanced Case (Dash Core as PrivateSend mixer only)

This results in the mixed PrivateSend funds being sent directly to the mobile wallet so that PrivateSend transactions can be sent directly from it without requiring access to a full node.

To support this, Dash Core would need to provide a method for inserting outputs not belonging to its wallet into the [`dsi` message](https://dash-docs.github.io/en/developer-reference#dsi).

### Mobile Wallet

1. A multi-account mobile wallet dedicates an account to PrivateSend funds and implements the proper PrivateSend coin selection algorithm for issuing PrivateSend transactions (i.e., no change, large enough fee, etc.). 

2. User sends funds to their trusted Dash Core full node and requests mixing. Request includes:
 - A list of outputs from its PrivateSend account
 - (Optional) The number of rounds to mix

3. (Optional?) Mobile wallet provides a signed message for each provided output to verify that it has the associated private key

### Dash Core (trusted mixing node)

When Dash Core receives a mixing request, it starts mixing for either the default number of rounds or the optionally provided number of rounds in the mixing request. 

On the final round of mixing, outputs from the mixing request are provided in the [`dsi` message](https://dash-docs.github.io/en/developer-reference#dsi) (instead of outputs internal to Dash Core's wallet). This results in  the mixed funds (sent by the [`dstx` message](https://dash-docs.github.io/en/developer-reference#dstx)) being sent directly to the mobile wallet.

At this point, the mobile wallet could send PrivateSend transactions with the same level of privacy/convenience as Dash Core. As mentioned above, it would be necessary for the wallet to properly construct PrivateSend transactions using the rules for fees, change, etc. that Dash Core does.


## Credits

This document was derived in part from a discussion on Discord with _TheDesertLynx_ and _hashengineering_.


# Introduction
An HTLC is a conditional transfer of value from "depositor" to "recipient" where two distinct conditions prevent immediate execution. 1) The hashlock requires presenting the proper "secret" to the blockchain prior to the 2) timelock expiration, else the value automatically returns to "depositor".

This allows two parties to exchange assets on independent platforms trustlessly and securely and thus enables Atomic Cross-Chain Swaps (ACCS) among other useful functionalities.

# hashed-timelock-contract-lc-ethereum
This is to demostrate an atomic swap of two blockchain networks using HTLC. 
Two participants: Peter and Han
Two blockchain networks:  Learning Coin (LC) and Ethereum Ropsten Testnet (ETH)
Transaction: Peter will send 1 LC to Han's LC wallet, in exchange, Han will send 2 ETH to Peter's ETH wallet. 

[Hashed Timelock Contracts](https://en.bitcoin.it/wiki/Hashed_Timelock_Contracts) (HTLCs) for Ethereum:

- [HashedTimelock.sol](contracts/HashedTimelock.sol) - HTLC for native tokens exchange

Use these contracts for creating HTLCs on the Ethereum side of a cross chain atomic swap (for example the [xcat](https://github.com/chatch/xcat) project).

# Process flow
## Creation Phase: 
having agreed on the parameters, Peter and Han create the HTLCs.

```
  1. Peter creates an HTLC on LC network with the receiver as Han. 1 LC is locked into the LC's HTLC. 
     He shares the smart contract ID and Hash(the hashlock).
  2. Han waits for the HTLC to confirm on the LC blockchain and validates all parameters, including 
     but not limited to, that she is the receiver, that the amount is appropriate, and the HTLC 
     hash-lock matches the agreed upon Hash.
  3. Han creates an HTLC on Ethereum with the receiver as Peter. 2 ETH are locked in the Ethereum HTLC.
  4. Peter waits for the HTLC to confirm on the Ethereum blockchain and validates that he is the 
     receiver and the HTLC hash-lock matches the agreed upon Hash.
```

## Redemption Phase: 
with the HTLCs created on both blockchains, Peter and Han can now proceed to the settlement.

```
  1. Peter redeems the ETH on Ethereum by redeeming the HTLC by submitting the Secret(pre-image) in
     a transaction. In doing so, he exposes the Secret(pre-Image).
  2. Han waits for Peter to expose the Secret, and then Han redeems the HTLC on LC using the same Secret.
```

## Done: 
the swap between Peter and Han is now complete.

## Run Tests
* Install truffle
* Install ganache [https://truffleframework.com/ganache](https://truffleframework.com/ganache)
* Launch and set the network ID to `4447`

```
$ npm i
$ truffle test
Using network 'test'.

Compiling ./test/helper/ASEANToken.sol...
Compiling ./test/helper/EUToken.sol...


  Contract: HashedTimelock
    ✓ newContract() should create new contract and store correct details (92ms)
    ✓ newContract() should fail when no ETH sent (84ms)
    ✓ newContract() should fail with timelocks in the past (78ms)
    ✓ newContract() should reject a duplicate contract request (159ms)
    ✓ withdraw() should send receiver funds when given the correct secret preimage (214ms)
    ✓ withdraw() should fail if preimage does not hash to hashX (111ms)
    ✓ withdraw() should fail if caller is not the receiver (162ms)
    ✓ withdraw() should fail after timelock expiry (1243ms)
    ✓ refund() should pass after timelock expiry (1273ms)
    ✓ refund() should fail before the timelock expiry (132ms)
    ✓ getContract() returns empty record when contract doesn't exist (48ms)

  Contract: HashedTimelockERC20
    ✓ newContract() should create new contract and store correct details (214ms)
    ✓ newContract() should fail when no token transfer approved (107ms)
    ✓ newContract() should fail when token amount is 0 (166ms)
    ✓ newContract() should fail when tokens approved for some random account (214ms)
    ✓ newContract() should fail when the timelock is in the past (136ms)
    ✓ newContract() should reject a duplicate contract request (282ms)
    ✓ withdraw() should send receiver funds when given the correct secret preimage (363ms)
    ✓ withdraw() should fail if preimage does not hash to hashX (227ms)
    ✓ withdraw() should fail if caller is not the receiver  (307ms)
    ✓ withdraw() should fail after timelock expiry (2257ms)
    ✓ refund() should pass after timelock expiry (2407ms)
    ✓ refund() should fail before the timelock expiry (283ms)
    ✓ getContract() returns empty record when contract doesn't exist (55ms)

  Contract: HashedTimelock swap between two ERC20 tokens
    ✓ Step 1: Alice sets up a swap with Bob in the AliceERC20 contract (233ms)
    ✓ Step 2: Bob sets up a swap with Alice in the BobERC20 contract (239ms)
    ✓ Step 3: Alice as the initiator withdraws from the BobERC20 with the secret (97ms)
    ✓ Step 4: Bob as the counterparty withdraws from the AliceERC20 with the secret learned from Alice's withdrawal (144ms)
    Test the refund scenario:
      ✓ the swap is set up with 5sec timeout on both sides (3613ms)

  Contract: HashedTimelock swap between ERC721 token and ERC20 token (Delivery vs. Payment)
    ✓ Step 1: Alice sets up a swap with Bob to transfer the Commodity token #1 (256ms)
    ✓ Step 2: Bob sets up a swap with Alice in the payment contract (231ms)
    ✓ Step 3: Alice as the initiator withdraws from the BobERC721 with the secret (95ms)
    ✓ Step 4: Bob as the counterparty withdraws from the AliceERC721 with the secret learned from Alice's withdrawal (132ms)
    Test the refund scenario:
      ✓ the swap is set up with 5sec timeout on both sides (3737ms)

  Contract: HashedTimelock swap between two ERC721 tokens
    ✓ Step 1: Alice sets up a swap with Bob in the AliceERC721 contract (225ms)
    ✓ Step 2: Bob sets up a swap with Alice in the BobERC721 contract (265ms)
    ✓ Step 3: Alice as the initiator withdraws from the BobERC721 with the secret (131ms)
    ✓ Step 4: Bob as the counterparty withdraws from the AliceERC721 with the secret learned from Alice's withdrawal (119ms)
    Test the refund scenario:
      ✓ the swap is set up with 5sec timeout on both sides (3635ms)


  39 passing (27s)
```

## Protocol - Native token exchange between Learning Coin (LC) and Ethereum (ETH)

### Main flow

![](docs/sequence-diagram-htlc-lc-eth-success.png?raw=true)

### Timelock expires

![](docs/sequence-diagram-htlc-eth-refund.png?raw=true)


## Interface

### HashedTimelock

1.  `newContract(receiverAddress, hashlock, timelock)` create new HTLC with given receiver, hashlock and expiry; returns contractId bytes32
2.  `withdraw(contractId, preimage)` claim funds revealing the preimage
3.  `refund(contractId)` if withdraw was not called the contract creator can get a refund by calling this some time after the time lock has expired.

See [test/htlc.js](test/htlc.js) for examples of interacting with the contract from javascript.

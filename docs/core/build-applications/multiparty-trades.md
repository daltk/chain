# Multiparty Trades

## Overview

 This guide demonstrates how to use the Chain Core API to create complex transactions that are:

- **Multi-party**: Multiple accounts--on the same core or different cores--can participate in the same transaction.
- **Multi-asset**: Multiple assets, originating from any core, can be traded and issued in the same transaction.
- **Risk-free**: Within a single transaction, each party can declare exactly what they will pay, and what they expect to receive. All transfers and issuances in the transaction happen atomically: together, at the same instant, and only when all conditions specified by all parties are met.

Please make sure you’ve read [Transaction Basics](../build-applications/transaction-basics.md) before continuing.

### Sample Code

All code samples in this guide are extracted from a single Java file.

<a href="../examples/java/MultipartyTrades.java" class="downloadBtn btn success" target="\_blank">View Sample Code</a>

## Example: Trading within the same core and application

In this example, Alice and Bob both have accounts on the same core. Alice holds Alice Dollars, and Bob holds Bob Bucks. Their accounts are managed by a central application that has access to both of their HSMs.

In this setting, the application can trade Alice’s Alice Dollars for Bob’s Bob Bucks directly, within a single transaction:

$code ../examples/java/MultipartyTrades.java same-core-trade

This is the simplest possible scenario for a multi-asset trade between two accounts. Because the assets and accounts are local to the same core, the application can name all of the relevant accounts and assets in the space of a single `build` call. And since the application has access to both Alice’s and Bob’s HSMs, it can sign for both parties simultaneously.

## Example: Trading between cores

Suppose that we want to conduct a similar trade as above, but Alice’s account and Bob’s account are now on different cores, and managed by separate applications.

Alice initiates the trade by building a transaction that stipulates what she will pay and what she expects to receive. Since the Bob Buck asset is not local to her core, Alice can’t refer to it by its human-readable alias. Instead, she has to know its asset ID from an out-of-band source, like an email from Bob.

$code ../examples/java/MultipartyTrades.java build-trade-alice

Unlike the transaction in the first example, this transaction is _unbalanced_: there are 50 Alice Dollars in the inputs that appear in no output, and 100 Bob Bucks that appear in an output but no input. If Alice were to sign and submit the transaction at this point, it would be rejected by the blockchain.

Instead, Alice provides a signature that commits her to her payment of Alice Dollars only if she receives Bob Bucks in the same transaction. The partial transaction will need more actions stacked on top of it to become complete, so Alice must explicitly allow additional actions to be added before she signs it:

$code ../examples/java/MultipartyTrades.java sign-trade-alice

Now Alice has a signed, partial transaction that she can send to someone who can give her Bob Bucks in exchange for her Alice Dollars:

$code ../examples/java/MultipartyTrades.java base-transaction-alice

Alice takes the raw transaction, `baseTransactionFromAlice`, and emails it to Bob. Now Bob can fill in the rest of the trade, using the partially-signed transaction from Alice as a base transaction:

$code ../examples/java/MultipartyTrades.java build-trade-bob

As was the case with Alice, Bob can’t refer to non-local assets by alias, so he refers to the Alice Dollar asset by its asset ID.

Note that the transaction now has all of the components of the trade described in the first example: it contains payments from Alice and Bob of their respective currencies, and names the other party as the recipients.

With Bob’s addition, the transaction’s incoming and outgoing assets are now balanced. But it’s not a valid transaction without his signature. Since Bob is the last participant in the trade, he does _not_ allow additional actions before he signs the transaction:

$code ../examples/java/MultipartyTrades.java sign-trade-bob

Finally, with the balanced transaction signed by both parties, Bob can submit the transaction to the blockchain network:

$code ../examples/java/MultipartyTrades.java submit-trade-bob

************
Introduction
************


Hyperledger Sawtooth is an enterprise distributed ledger (aka blockchain) project.
Our design philosophy targets keeping distributed ledgers *distributed* and
making smart contracts *safe* - particularly for enterprise use.

In fitting with this enterprise focus, Sawtooth is also highly modular.
This enables enterprises and consortia to make policy decisions that they are
best equipped to make.

We are an open source project under the Hyperledger umbrella. We welcome
working with individuals and companies interested in the advancement of
distributed ledger technology. Please see the community section for ways to
interact with and become a part of the Sawtooth community.


Sawtooth's Ledger
=================
Distributed ledgers are shared databases with the unique feature that they do
not rely on a central authority or intermediary. Instead of a single,
centralized database, each participant operates a copy of the database,
verifies transactions, and engages in a protocol that ensures universal
agreement on the state of the ledger. While Bitcoin is the most well-known
distributed ledger, the technology has been proposed for many different
applications ranging from international remittance, insurance claim
processing, supply chain management and the Internet of Things (IoT).

Distributed ledgers generally consist of three basic components:

    * A data model that captures the current state of the ledger

    * A language of transactions that change the ledger state

    * A protocol used to build consensus among participants around
      which transactions will be accepted by the ledger.

Transaction Families
====================
In Sawtooth, the data model and transaction language are implemented
in a *transaction family*. While we expect users to build custom transaction
families that reflect the unique requirements of their ledgers, we provide
several core transaction families as models\:

    * Validator Registry - Provides a methodology
      for registering ledger services.

    * IntegerKey - Used for testing deployed ledgers.

    * Settings - Provides a reference implementation for storing
      on-chain configuration settings.

    * Identity - Handles on-chain permissioning for transactor
      and validator keys to streamline managing identities
      for lists of public keys.

Additional transaction families provide models for specific areas\:

    * Smallbank - Handles performance analysis for benchmarking
      and performance testing when comparing the performance of
      blockchain systems.
      This transaction family is based on the H-Store Smallbank benchmark.

    * Seth - Enables the creation and execution of smart contracts.
      This transaction family integrates the Hyperledger Burrow
      implementation of the Ethereum Virtual Machine (EVM)
      into the Hyperledger Sawtooth framework using the
      Sawtooth Go SDK.

    * BlockInfo - Provides a methodology for storing information
      about a configureable number of historic blocks.
      This transaction family is used for Seth smart contracts.

We also provide several example transaction families to demonstrate
Sawtooth functionality\:

    * XO - Allows users to play Tic-Tac-Toe, also known as
      "noughts and crosses" or "Xs and Os".

    * Supply Chain - Provides the infrastructure for tracking
      physical objects in a real-world supply chain on the Sawtooth Ledger.

    * Track and Trade - Provides a methodology for users to track goods
      as they move through a supply chain.
      This transaction family includes the history of ownership and
      custodianship, as well as histories for a variety of properties
      such as temperature and location.

Consensus
=========

Consensus is the process of building agreement among a group of mutually
distrusting participants. There are many different algorithms for building
consensus based on requirements related to performance, scalability,
consistency, threat model, and failure model. While most distributed ledgers
operate with an assumption of Byzantine failures (malicious attacker),
other properties are largely determined by application requirements.
For example, ledgers used to record financial transactions often require
high transaction rates with relatively few participants and immediate
finality of commitment. Consumer markets, in contrast, require substantial
aggregate throughput across a large number of participants; however,
short-term finality is less important.

Algorithms for achieving consensus with arbitrary faults generally require
some form of voting among a known set of participants. Two general approaches
have been proposed. The first, often referred to as "Nakamoto consensus",
elects a leader through some form of “lottery”. The leader then proposes a
block that can be added to a chain of previously committed blocks. In Bitcoin,
the first participant to successfully solve a cryptographic puzzle wins
the leader-election lottery. The elected leader broadcasts the new block
to the rest of the participants who implicitly vote to accept the block by
adding the block to a chain of accepted blocks and proposing subsequent
transaction blocks that build on that chain.

The second approach is based on traditional
`Byzantine Fault Tolerance (BFT)
<https://en.wikipedia.org/wiki/Byzantine_fault_tolerance>`_
algorithms and uses multiple rounds of explicit votes to achieve consensus.
`Ripple <https://ripple.com/>`_ and `Stellar <https://www.stellar.org/>`_
developed consensus protocols that extend traditional BFT for open
participation.

Sawtooth abstracts the core concepts of consensus and isolates consensus
from transaction semantics. The interface supports plugging in various
consensus implementations. Sawtooth provides two such implementations:
dev_mode and PoET.

Dev_mode is a simplified random leader algorithm that is useful
for developers and test networks that require only crash fault tolerance.

PoET, short for “Proof of Elapsed Time" is a Nakamoto-style consensus algorithm.
It is designed to be a production-grade protocol capable of supporting large
network populations.


Proof of Elapsed Time (PoET)
============================

For the purpose of achieving distributed consensus efficiently,
a good lottery function has several characteristics:

    * Fairness: The function should distribute leader election
      across the broadest possible population of participants.

    * Investment: The cost of controlling the leader election
      process should be proportional to the value gained from it.

    * Verification: It should be relatively simple for all participants
      to verify that the leader was legitimately selected.

PoET is designed to achieve these goals using new secure CPU instructions
which are becoming widely available in consumer and enterprise processors.
PoET uses these features to ensure the safety and randomness of the leader
election process without requiring the costly investment of power and specialized
hardware inherent in most “proof” algorithms.

We include an implementation which simulates the secure instructions.
This should make it easier for the community to work with the software but
also forgoes byzantine fault tolerance.

PoET essentially works as follows...
Every validator requests a wait time from an enclave(a trusted function).
The validator with the shortest wait time for a particular transaction
block is elected the leader. One function, say “CreateTimer” creates
a timer for a transaction block that is guaranteed to have been created
by the enclave. Another function, say “CheckTimer” verifies that the timer
was created by the enclave and, if it has expired, creates an attestation
that can be used to verify that validator did, in fact, wait the allotted
time before claiming the leadership role.

The PoET leader election algorithm meets the criteria for a good lottery
algorithm. It randomly distributes leadership election across the entire
population of validators with distribution that is similar to what is
provided by other lottery algorithms. The probability of election
is proportional to the resources contributed (in this case, resources
are general purpose processors with a trusted execution environment).
An attestation of execution provides information for verifying that the
certificate was created within the enclave (and that the validator waited
the allotted time). Further, the low cost of participation increases the
likelihood that the population of validators will be large, increasing
the robustness of the consensus algorithm.


Getting Sawtooth
=====================

The Sawtooth platform is distributed in source code form with
an Apache license. You can get the code `here
<https://github.com/hyperledger/sawtooth-core>`_ and start building your own
distributed ledger.

Repositories
============

One repository contains all of the the code needed:

sawtooth-core
    Contains fundamental classes used throughout the Sawtooth project, as well as:

    * The implementation of the validator process which runs on each node
    * SDKs for writing transaction processing or validation logic in a variety
      of languages
    * Tools including a Vagrant environment for easily launching a network of
      validators
    * Dockerfiles to support development or launching a network of validators
    * Source files for this documentation

---
title: IronFleet - Proving Practical Distributed Systems Correct
updated: 2021-08-03
---
IronFleet is a methodology for automated machine-checked verification in [Dafny](https://github.com/dafny-lang/dafny) of the safety and liveness of distributed system implementations.
It uses TLA-style state-machine refinement to reason about protocol-level concurrency, then use Floyd-Hoare style imperative verification to reason about implementation complexities.

IronFleet organizes a distributed system's implementation and proof into 3 layers:
the high-level spec layer, the distributed-protocol layer and the implementation layer.

To avoid complex reasoning about interleaved execution of low-level operations at multiple hosts, the proofs assume that every implementation step performs an atomic protocol step.
Since the real implementation's execution is not atomic, we use a _reduction_ argument to show that a proof assuming atomicity is equally valid as a proof for the real system.

All IronFleet code is publicly available at [GitHub](https://github.com/microsoft/Ironclad/blob/main/ironfleet/).

## The high-level spec layer

The developer writes the system's spec as a state machine.
The spec consists of three predicate: `SpecInit` describes acceptable starting states, `SpecNext` describes acceptable ways to move from an old to a new state, and `SpecRelation` describes the required conditions on the relation between an implementation state and its corresponding abstract state.

## The distributed-protocol layer

The developer specifies a distributed system state machine, which consists of N host state machines and a collection of network packets.
In each step of the execution, one host's state machine takes a step, atomically reading messages from the network, updating its state and sending messages to the network.
The atomicity greatly improves the proof that the protocol layer refines the spec layer.

### Connecting the protocol layer to the spec layer

The developer defines a refinement function `PRef` that takes a state of the protocol layer and returns the corresponding state of the spec layer.
She proves that if a step of the protocol layer takes the state from `old` to `new`, then there exists a sequence of spec steps that goes from `PRef(old)` to `Pref(new)`.

## The Implementation Layer

The developer writes single-threaded imperative code to run on each host.
She uses a ghost variable that records a "journal" of every network action, i.e., send and receive, that takes place.

### Connecting the implementation layer to the protocol layer

We prove the implementation layer correctly refines the protocol layer.
This is simplified by the fast that each step in the implementation layer corresponds to exactly one step of the protocol layer.
Since the implementation's event handler is not atomic (i.e., low-level operations by different machines may arbitrary interleave), we use a "reduction" argument to bridge the gap.

#### Reduction

Given a fine-grained behavior from a real concurrent system, we use reduction to convert it to an equivalent behavior of coarse-grained steps, simplifying verification.
Two steps can swap places in the behavior if swapping them has no effect on the execution's outcome.
Applying reduction requires identifying all the steps of the system, proving commutativity relationships among them, and applying these relationships to create an equivalent behavior.
We constrain the implementation to, in any given step, perform all of its receives before all its sends, to enable reduction.
We moreover require that a step may perform at most one time-dependent operation, such as clock read, blocking receive and non-blocking receive that returns no packets.
The step must perform all receives before this time-dependent operation, and all sends after it.
(I wonder about the consequences of this requirement.)

## Verifying Liveness

Liveness properties are much harder to verify than safety properties.
Safety proofs need only reason about two system states at a time: if each step between two states preserves the system's safety invariant.
Liveness requires reasoning about infinite series of system states.

IronFleet addresses the challenge via a custom TLA embedding in Dafny that focuses the prover's efforts in a fruitful directions.
We modified Dafny to better control how many times the SMT solver may expand a function's definition and to make Dafny use triggers more cautiously.

#### References

- [IronFleet: proving practical distributed systems correct (Symposium on Operating Systems Principles, 2015)](https://dl.acm.org/doi/10.1145/2815400.2815428)
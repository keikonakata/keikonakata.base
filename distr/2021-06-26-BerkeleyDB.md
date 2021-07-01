---
title: Berkeley DB
updated: 2021-07-01
---

The Berkeley Database is an embedded database engine, distributed as a library that can be linked directly into an application.
It stores and retrieves records, which consist of key/value pairs.

Berkeley DB supports three access methods: B+tree, Extended Linear Hashing and Fixed- or Variable-length Records.

Berkeley DB supports ACID transactions and provides a two-phase locking service.
During the operation of a transaction, locks are acquired but never released.
At the end of the transaction, locks are released but never acquired.
The Berkeley DB detects deadlocks and automatically rolls back one of the transactions.

Berkeley DB supports write-ahead logging with checkpointing.
Log writes are sequential, but data page writes are random.
This improves performance.

Berkeley DB is thread-safe.

#### References
[Berkeley DB (USENIX Annual Technical Conference, 1999)](https://dl.acm.org/doi/10.5555/1268708.1268751)
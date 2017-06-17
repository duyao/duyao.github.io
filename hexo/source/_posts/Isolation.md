title: Isolation
date: 2015-10-24 09:38:22
tags: database
categories: note

---

# Isolation

## Introduction 

Isolation is typically defined at database level as a property that defines how/when the changes made by one operation become visible to other.

Isolation is one of the ACID (Atomicity, Consistency, Isolation, Durability) properties.

- **Atomicity**: A transaction should be treated as a single unit of operation which means either the entire sequence of operations is successful or unsuccessful.

- **Consistency**: This represents the consistency of the referential integrity of the database, unique primary keys in tables etc.

- **Isolation**: There may be many transactions processing with the same data set at the same time, each transaction should be isolated from others to prevent data corruption.

- **Durability**: Once a transaction has completed, the results of this transaction have to be made permanent and cannot be erased from the database due to system failure.

Two-phase locking is the most common transaction concurrency control method in DBMSs, used to provide both serializability and recoverability for correctness.

<!--more-->

## Isolation level

The programmer must carefully analyze database access code to ensure that any relaxation of isolation does not cause software bugs that are difficult to find. Conversely, if higher isolation levels are used, the possibility of deadlock is increased, which also requires careful analysis and programming techniques to avoid.

- **Serializable**

  This is the highest isolation level.

  - [x] read locks
  - [x] write locks 
  - [x] range-locks 


- **Repeatable reads**

  - [x] read locks
  - [x] write locks
  - [ ] range-locks


- **Read committed**

  - [ ] read locks
  - [X] write locks
  - [ ] range-locks


  Putting it in simpler words, read committed is an isolation level that guarantees that any data read is committed at the moment it is read. It simply restricts the reader from seeing any intermediate, uncommitted, 'dirty' read. It makes no promise whatsoever that if the transaction reissues the read, it will find the same data; data is free to change after it is read.

- **Read uncommitted**

  This is the lowest isolation level.

  - [ ] read locks
  - [ ] write locks
  - [ ] range-locks


## Read phenomena
- **Dirty reads**
earlier updates will always appear in a result set before later updates.

- **Non-repeatable reads**

In this example, Transaction 2 commits successfully, which means that its changes to the row with id 1 should become visible. However, Transaction 1 has already seen a different value for age in that row. 

At the `SERIALIZABLE` and `REPEATABLE READ` isolation levels, the DBMS must return the **old** value for the second SELECT. 

At `READ COMMITTED` and `READ UNCOMMITTED`, the DBMS may return the **updated** value; this is a non-repeatable read.

- **Phantom reads**
This can occur when *range locks* are not acquired on performing a SELECT ... WHERE operation.

Note that Transaction 1 executed the same query twice. 

If the highest level of isolation were maintained, the same set of rows should be returned both times, and indeed that is what is mandated to occur in a database operating at the SQL `SERIALIZABLE` isolation level. 

However, at the lesser isolation levels, a different set of rows may be returned the second time.


## Summary

|Isolation level	|Dirty reads	|Non-repeatable reads	|Phantoms|
|:---:|:---:|:---:|:---:|
|Read Uncommitted	|may occur	|may occur	|may occur|
|Read committed 	|-			|may occur	|may occur|
|Repeatable Read	|-			|-			|may occur|
|Serializable		|-			|-			| - |


In lock-based concurrency control, isolation level determines the duration that locks are held.

- "C" - Denotes that locks are held until the transaction commits.

- "S" - Denotes that the locks are held only during the currently executing statement. Note that if locks are released after a statement, the underlying data could be changed by another transaction before the current transaction commits, thus creating a violation.

|Isolation level	|Write Operation	|Read Operation	|Range Operation (...where...)|
|:---:|:---:|:---:|:---:|
|Read Uncommitted	|S	|S	|S|
|Read Committed	 	|C	|S	|S|
|Repeatable Read	|C	|C	|S|
|Serializable		|C	|C	|C|
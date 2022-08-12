---
title: "What is ACID"
date: 2022-06-14T14:07:12+05:30
draft: false
tags: ["Database"]
---

ACID is the building blocks of a relational databases. Today we're going to understand what it means.

**tldr;** Just remember

- *Atomicity* relates to *Transactions* - All or Nothing.
- *Consistency* relates to *Constraints and Triggers* - All the data is consistent.
- *Isolation* relates to *Locks* - Concurrent operation doesn't intefere with each other.
- *Durability* relates to *Replication* - Committed Transactions to be persisted in case of system failure.


## Atomicity - *Transactions*

***
Atomicity guarantees **All or Nothing**.

A set of instructions are executed as an atomic unit, either all succeeds or none.
***

Say Alice is transferring 500$ to Bob's account:

> BEGIN TRANSACTION
> 1. Check 500$ Balance in Alice's Account - SUCCESS
> 2. Deduct 500$ from Alice's Account - SUCCESS
> 3. Deposit 500$ to Bob's Account - FAILURE :(
>
> END TRANSACTION

Due to network failure the Deposit 500$ step fails. Since all the steps are in a Transaction, all of them are reverted. 500$ is transferred back to Alice's account

## Consistency - *Constraints and Triggers*

***
Consistency ensures **All the data is consistent**.

When any change is carried out a set of rules are conformed with when the data changes.
***

Say we've added a constraint on *Accounts* table - account's amount cannot be < 0. This constraint is checked whenever data changes in *Accounts* table.

> 1. Check Balance in Alice's Account - 200$
> 2. Deduct 800$ from Alice's Account - FAILURE

Deducting 800$ from Alice's account failed because of the constraint mentioned before.

## Isolation - *Locks*

***
Isolation makes sure **Concurrent operation doesn't intefere with each other**.

If 2 transactions tries to write on the same row then only one must be allowed at a time.
***

Let's say there are two Transactions updating Alice's account

**T1**
> 1. Read Balance from Alice's Account - 1200$
> 2. Deduct 800$ from Alice's Balance - 400$
> 3. Update Alice's Account with 400$

**T2**
> 1. Read Balance from Alice's Account - 1200$
> 2. Deduct 200$ from Alice's Balance - 1000$
> 3. Update Alice's Account with 1000$

If **T1** and **T2** were allowed to run without locks then the updates can go wrong:

> - **T1** Reads Balance from Alice's Account - 1200$
> - **T2** Reads Balance from Alice’s Account - 1200$
> - **T1** Deducts 800$ from Alice's Balance - 400$
> - **T1** Updates Alice's Account with - 400$
> - **T2** Deducts 200$ from Alice's Balance - 1000$
> - **T2** Updates Alice's Account with - 1000$

Finally Alice's Account has 1000$ which is not right. Using locks with Transactions makes sure that only one transaction can perform operations on the same data.

## Durability - *Replication and Transaction Log Files*

***
Durability allows **Committed Transactions to be persisted in case of system failure**.
***

Say in our Alice and Bob example the system failed before transferring 500$ from Alice's account to Bob's account. Alice's account now has 500$ less while Bob didn't got the 500$ since it was committed.

After the system comes back online it must redo the deposit to Bob's account.

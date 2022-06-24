---
title: "What is ACID"
date: 2022-06-14T14:07:12+05:30
draft: false
---

ACID is the building blocks of a relational databases. Today we're going to understand what it means.

## Atomicity - *Transactions*

***
Atomicity guarantees **All or Nothing**.

A set of instructions are executed as an atomic unit, so either all succeeds or all are reverted.
***

For eg. Let's say Alice is transferring 500$ to Bob's account:

> BEGIN TRANSACTION
> 1. Check Balance in Alice's Account - SUCCESS
> 2. Deduct 500$ from Alice's Account - SUCCESS
> 3. Deposit 500$ to Bob's Account - FAILURE :(
> END TRANSACTION

Due to network failure the Deposit step fails. Since all the steps are in a Transaction, they will be reverted. Finally Alice will have her 500$ back in her account.

## Consistency - *Constraints and Triggers*

***
Consistency ensures **All the data is consistent**.

When any change is carried out a set of rules are conformed with when the data changes.
***

Let's say we've added a constraint on Accounts table where an account cannot have amount < 0. This constraint will be checked whenever data is changed in Accounts table.

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

If T1 and T2 were allowed to run without locks then the updates can go wrong:

> **T1** 1. Read Balance from Alice's Account - 1200$
> **T2** 1. Read Balance from Alice's Account - 1200$
> **T1** 2. Deduct 800$ from Alice's Balance - 400$
> **T1** 3. Update Alice's Account with 400$
> **T2** 2. Deduct 200$ from Alice's Balance - 1000$
> **T2** 3. Update Alice's Account with 1000$

Finally Alice's Account has 1000$ which is not right. Using locks with Transactions makes sure that only one transaction can perform operations on the same data.
## Durability - *Replication and Transaction Log Files*

***
Durability allows **Committed Transactions to be persisted in case of system failure**.
***

In case of our Alice and Bob bank transfer example if the failure happened to be a system failure then after the system comes online it must have 500$ deducted from Alice's account and redo the deposit to Bob's account.

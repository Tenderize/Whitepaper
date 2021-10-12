
# Tender Protocol

## Table of Contents

1. [**Introduction**](#1-Introduction)
2. [**Problem Statement**](#2-Problem-Statement)
3. [**Solution**](#3-Solution)
4. [**Technical Architecture**](#4-Technical-Architecture)
    1.   [TenderTokens](#41-TenderTokens)
    2.   [Tenderizer](#42-Tenderizer)
    3.   [Elastic Supply Pool](#43-Elastic-Supply-Pool)
    4.   [TenderFarm](#44-TenderFarm)
5. [**Protocol Economics**](#5-Protocol-Economics)
    1. [Governance Fee](#51-Governance-Fee)
    2. [Liquidity Fee](#52-Liquidity-Fee)
    3. [Swap Fee](#53-Swap-Fee)
    4. [Emergency Exit](#54-Emergency-Exit)
6. [**Governance**](#6-Governance)
    1.  [Overview](#61-Overview)
    2.  [$TENDER Token](#62-$TENDER-Token)

## 1. Introduction 

The Tender protocol enables liquid staking and yield aggregation for various web3 protocols. It provides end-users a way to earn automatically compounding staking rewards without locking up capital or having to keep close track of several stake delegation markets.

## 2. Problem Statement

Usually when depositing tokens in staking protocols it becomes 'frozen' and has to 'unthaw' before funds can be withdrawn again. This wait period often takes upward of 14 days, during which no rewards are earned and an added risk of negative market movements exists.  Since the capital is frozen there is also a lost opportunity cost as that capital can not be deployed elsewhere such as lending protocols nor collateralise the position.

On top of that staking is relatively complex. There can be mechanisms that require frequent action to optimise returns and compound staking rewards or one has to keep track of the performance of the node capital is allocated to and reconsider allocation. Because of these complexities capital is often used inefficiently and networks as a whole could benefit from more efficiency in their stake delegation markets.

## 3. Solution 

The Tender Protocol solves these problems and offers users an easy way to earn yield on their stake without added complexities. Users deposits are aggregated and collectively staked towards node operators based on a list curated by an incentivised governance mechanism.

When users deposit their tokens they receive a tenderToken derivative in return. TenderTokens are algoritmic elastic supply ERC-20 stablecoins that trade 1:1 with the underlying deposited tokens. As the protocol accrues staking rewards the balance of tenderTokens in user's wallets will increase relative to their stake weight. 

TenderTokens can be transferred freely, traded on decentralised exchanged or used in DeFi protocols which finally bridges web3 protocols with DeFi.  



## 4. Technical Architecture

- Tender governance
- TenderToken
- Tenderizer
- ElasticSupplyPool (liquidity pool)
- TenderFarm

![](https://user-images.githubusercontent.com/63185771/130828326-7c51078f-38d9-4aff-8c58-46d8f7160f6e.JPG)



### 4.1 TenderTokens

Each staking pool will have its own liquid ('tender') derivative called TenderTokens. They are minted when a user deposits funds and burned should a user manually unstake and sit through the unthawing period rather than sell their assets on the open market. 

#### ERC-20
 These tokens follow the ERC-20 token standard with a modified implementation. On top of the usual functionality to read balances or transfer tokens there will be added logic to algoritmically control the supply.
 
#### Elastic Supply
 
 The supply of TenderTokens will increase as the staking pool earns rewards or contract in case slashing occurs (nodes losing a portion of their deposit for deviant behaviour). This change in the supply is also reflected in the respective balances of each token holder. Due to this mechanism TenderTokens are stablecoins that are redeemable 1:1 for their underlying assets. 

**Elastic Supply Example** (STEAK is a fictive token)

1. Alice deposits 75 STEAK, 75 tenderSTEAK is minted for Alice
    - Total Value Locked: 75 STEAK
    - Tender Supply:      75 tenderSTEAK
    - Alice:              75 tenderSTEAK
2. Bob deposits 25 STEAK, 25 tenderSTEAK is minted for Bob
    - Total value Locked: 100 STEAK
    - Tender Supply: 100 tenderSTEAK
    - Alice: 75 tenderSTEAK
    - Bob: 25 tenderSTEAK
3. Protocol earns 40 STEAK in rewards
    - Total value Locked: 140 STEAK
    - Tender Supply: 100 tenderSTEAK
    - Alice: 75 tenderSTEAK
    - Bob: 25 tenderSTEAK
4. Atomically with step 3, the Protocol rebases the tenderSTEAK supply to match the STEAK TVL
    - Total value Locked: 140 STEAK
    - Tender Supply: 140 tenderSTEAK
    - Alice: 75 tenderSTEAK + 30 tenderSTEAK = 105 tenderSTEAK
    - Bob: 25 tenderSTEAK + 10 tenderSTEAK = 35 tenderSTEAK

### 4.2 Tenderizer

The Tenderizer contract is the bread and butter of the Tender Protocol, it's implementation is specific to each integration. It handles staking tokens and claiming rewards.

In case there are protocol specific rewards that are not its native staking token they will be swapped on decentralised exchanges (e.g. uniswap) whitelisted by the protocol. This takes place without interference of outside actors and is fully automated by the smart contract.

This contract is the only contract that is upgradeable in the first iteration of the protocol.

### 4.3 Liquidity Pool

Each supported integration will have its own liquidity pool through which tenderTokens can be swapped back and forth with their underlying tokens. 

To avoid impermanent loss when the tenderToken supply is rebased the concept of *Elastic Supply Pools* is introduced. Impermanent loss occurs when the values of the assets within the pool deviate from one another. It is *impermanent* because the loss is nullified if the assets in the pool return to its original ratios. 

A traditional liquidity pool wouldn't be aware of an event that changes the supply of the tenderTokens. In this case the tenderToken balance in the liquidity pool would change causing an impermanent loss as the tenderToken then trades at a discount or premium, depending on whether the supply contracts or expands. This change in balances within the pool causes impermanent loss and arbitrage occurs until the price reaches parity again. 

The Tender Protocol smart contracts as creator of the liquidity pool will be able to resyncronise the weights within the pool as the sole actor. This will prevent the price from changing when the tenderToken balance of the liquidity pool expands or contracts. 

e.g. 
1. tenderSTEAK/STEAK liquidity pool has a 50/50 weight distribution
    - 1000 STEAK
    - 1000 tenderSTEAK
2. rewards are earned, the tenderSTEAK supply is rebased, increasing the balance of the pool to 1500
    - 1000 STEAK
    - 1500 tenderSTEAK
3. the weights are resynced so that the exchange rate remains 1:1
    - STEAK weight = 1000 / 2500 = 40% 
    - tenderSTEAK weight = 1500 / 2500 = 60% 

### 4.4 TenderFarm

To incentivise liquidity provision each integration will have it's own yield farming contract called a 'TenderFarm'. When users add liquidity to the Elastic Supply Pool they will receive ERC20 tokens representing their liquidity share of the pool. These tokens can be staked in the TenderFarm to earn rewards. 

Because the Liquidity Pool exists partially of yield-generating 'TenderTokens', providing liquidity will already earn partial staking rewards relative to the token weight of the 'TenderToken' in the liquidity pool.

Currently the rewards will be a percentage cut on the staking rewards earned by the Tenderizer. These rewards will be automatically added to the TenderFarm as the tender protocol earns staking rewards.

In the future these yield farming contracts could be expanded to also include distribution of a tender protocol governance token. 

## 5. Protocol Economics

### 5.1 Governance Fee

An initial 2.5% of staking rewards will be deducted from the earned stkaing rewards by the 'Tenderizer' and assigned to the governance contract (a multisig initially) to fund protocol development, community grants, ... .

This fee is only on rewards and never on principal deposits by users. Governance is thus incentivised to effectively manage the 'Tenderizer' for good performance. 

### 5.2 Liquidity Fee

To bootstrap and retain liquidity an initial 7.5% of staking rewards will be assigned to users that provide liquidity and stake their liquidity pool shares in the 'TenderFarm'.

Users that provide liquidity forgo staking rewards relative to the weight of the underlying staked assets in the liquidity pool so this opportunity cost needs to be offset with an incentive. 

While the total yield might still be lower than staking your entire balance straight up, providing liquidity caters to a different risk profile. Since part of the provided assets will still be the staked asset token these will be subject to less protocol risk.

### 5.3 Swap Fee

A 0.5% trading fee will be charged on liquidity pools which will be paid to liquidity providers.

### 5.4 Emergency Exit

Whenever possible (depending on the implementation details of the underlying protocols) the 'Tenderizer' contracts provide an emergency exit whereby users can manually unlock their tokens, wait for the protocol unstaking period and then withdraw their tokens.

This will burn their 'TenderTokens' and unlock their respective share of staked assets from the underlying protocol. 


## 6. Governance

### 6.1 Overview

Administration and upgrades to the Tender protocol happen through the collective decision making of the Tenderize community. This can be adding new assets, changing global or tenderizer specific parameters, execute protocol upgrades, etc. While initially the protocol will be administered by a group of core team members and early contributers, we believe that ultimately Tenderize should be an unstoppable, community managed financial protocol.

In a first stage proposals will be voted on off-chain as a signal to the protocol administration committee of which updates to execute. The administration committee will exist out of core team members and early contributers/investors. In the second stage proposals will be executable code and not signals or suggestions for a small set of key actors to follow through on.

Participation in governance begins with the $TENDER token, which can be locked up to obtain voting rights. Voting rights can also be delegated to other actors that have voting rights. Through this community members can off-load some of their responsabilities to other community members that are able to make better informed decisions. Participants will receive a portion of the "Governance fee" from each of the Tenderizers equal to their share of $TENDER tokens that are eligible for voting rights.

We believe in rewarding active participation so incentives should be aligned accordingly, users that vote more often and with more conviction (measured by locking up voting power to proposals longer) should result in more rewards as they have more skin in the game.

### 6.2 $TENDER Token

The $TENDER token is an ERC-20 token that empowers the management of the Tenderize protocol by the community. By locking up $TENDER tokens to acquire voting rights, holders gain the right to perform management duties for the protocol by creating or voting on proposals based on their own analysis and theses.
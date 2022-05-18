---
title: "Redacted Cartel Custom Approval Logic Bugfix Review"
author: IceBear
categories:
  - Security 
  - Vulnerability
tags:
  - Redacted Cartel
  - Logic
  - Review
sitemap :
changefreq : daily
priority : 1.0

# table of contents
toc: true # 오른쪽 부분에 목차를 자동 생성해준다.
toc_label: "table of content" # toc 이름 설정
toc_icon: "bars" # 아이콘 설정
toc_sticky: true # 마우스 스크롤과 함께 내려갈 것인지 설정
---

본 글은 팀 스터디 목적으로 immunefi의 블로그 글을 읽고 요약한 내용이다.

1월 11일 [Redacted] Cartel에 대한 취약점을 화이트 해커인 Tommaso Pifferi가 immunefi에 제보한 내용이다.
해당 취약점은 악의적인 해커가 일반 사용자가 가지고 있는 balance를 해커에게로 보낼 수 있도록 만들어주는 것으로,
6백만달러 가량 위험에 노출되었으나 Redacted 측은 해당 취약점을 패치하고 화이트 해커에게 $560,000를 지급하였다.

## ERC-20의 approve/transferFrom 함수

ERC-20의 approve(spender, amount) 함수는 token의 owner가 spender에게 amount 만큼의 token을 
전송할 수 있는 권한을 주는 기능을 담당한다.
해당 권한을 얻은 spender는 transferFrom(sender, recipient, amount) 함수를 통해 token을 전송할 수 있다.

## 취약점 분석

취약점은 ERC-20 표준 함수를 잘못 구현한 것에서 발생하였다. 
[OpenZeppelin]은 ERC-20나 ERC-721에 맞게 구현해둔 프레임워크로, 개발자들이 해당 표준에 맞게 일일이 구현하지 않아도 되게 
해준다. 하지만 Redacted Cartel에서는 이를 사용하지 않고 자체 구현한 함수를 사용하였다.

~~~
function transferFrom(address sender, address recipient, uint256 amount) public virtual override onlyAuthorisedOperators returns (bool) {
   _transfer(sender, recipient, amount);
   _approve(sender, msg.sender, allowance(sender, recipient ).sub(amount, "ERC20: transfer amount exceeds allowance"));
   return true;
  }
~~~

transferFrom 함수를 살펴보면 먼저 sender로 부터 recipient에게 amount 만큼의 token을 전송한다.
그리고 난 뒤 _approve 함수에서 sender가 transferFrom 함수를 호출한 msg.sender에게 sender가 recipient에게 
권한을 준 만큼의 token 양에서 방금 보낸 amount 만큼을 뺀 값을 전송할 수 있도록 다시 허용한다.
따라서 공격자가 0개의 token을 sender가 approve를 많이 해둔 recipient에게 전송하도록 transferFrom 함수를 호출하면
해당 취약점으로 인해 공격자가 recipient에게 허용된 amount 만큼 token을 전송할 수 있게 된다.

## 취약점 패치

취약점 패치는 transferFrom 함수를 제거하고 OpenZeppelin 프레임워크를 이용하도록 코드가 재작성되었다. 

## 결론

approve/transferFrom 함수는 이에 대한 정확한 이해없이 구현하게 되면 이러한 취약점을 발생시킬 수 있다.
이 함수에 대한 취약점은 DamnVulnerableDefi의 [Truster] 문제에서도 확인할 수 있으며, BEARS 블로그에서도
해당 문제에 대한 풀이를 확인할 수 있다.
Defi 관련 코드 오디팅 시 OpenZeppelin과 같은 어느정도 검증된 프레임워크가 아닌 자체 구현 함수 존재시
특히 해당 부분을 상세하게 검토할 필요가 있다고 할 수 있겠다.

## 참고

* https://www.redactedcartel.xyz
* https://medium.com/immunefi/redacted-cartel-custom-approval-logic-bugfix-review-9b2d039ca2c5
* https://bears-team.github.io/wargame/DamnVulnerableDefi-Truster/

[Redacted]: https://www.redactedcartel.xyz
[OpenZeppelin]: https://openzeppelin.com
[Truster]: https://bears-team.github.io/wargame/DamnVulnerableDefi-Truster/
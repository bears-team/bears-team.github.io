---
title:  "APWine Incorrect Check of Delegations Bugfix 리뷰"
excerpt: "Immunefi APWine Incorrect Check of Delegations Bugfix 문서 학습 목적으로 살펴본 내용임"

author: Panda
categories:
  - Security
  - Smart Contract
tags:
  - ERC20
  - Logical Bug
  - Critical Severity
  - Review
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2022-02-16 19:00:00 +09:00
lastmod: 2022-02-17 19:00:00 +09:00
sitemap :
changefreq : daily
priority : 1.0

# table of contents
toc: true # 오른쪽 부분에 목차를 자동 생성해준다.
toc_label: "table of content" # toc 이름 설정
toc_icon: "bars" # 아이콘 설정
toc_sticky: true # 마우스 스크롤과 함께 내려갈 것인지 설정

#{% figure caption:"Le logo de **Jekyll** et son clin d'oeil à Robert Louis Stevenson"
#  %}
#  ![Le logo de Jekyll]({{"/assets/images_posts/2021-09-23-cve-2021-31956-part1/1.png"| #relative_url}})
#{% endfigure %}
---
# Executive Summary
이 문서는 Immunefi사의 기술 블로그중 [APWine Incorrect Check of Delegations Bugfix Review](https://medium.com/immunefi/apwine-incorrect-check-of-delegations-bugfix-review-7e401a49c04f) 문서를 읽고 BEARS 팀내 기술 세미나 자료로 활용하기 위해 작성되었으며, 해당 블로그 내용 공유를 통해 Smart Contract내 취약점 유형을 학습하는데 중점을 두고 작성된 문서입니다.

# Background
## APWine
* APWine 서비스의 핵심은 미래에 발생할 이자를 토큰화 한다는 것
* 일반적으로 이자 파밍 DEX 서비스의 경우 유동성을 공급하면 이자포함한 토큰(IBT: Interest Bearing Token)를 발급하는데, APWine의 경우 이 IBT토큰을 일정기간을 맡기면 미래의 받을 이자를 의미하는 토큰(FYT : Future Yield Tokens)으로 교환해줌. 이자에 대한 이자를 지급하는 파생상품처럼 보임
* 또한 다른 DEX 거래소의 IBT에 대한 서비스를 제공하는 지에 대해서는 추가적인 조사가 필요하나 만약에 그렇다면 벰파이어공격(Vampire Attack)에 활용할 수 있을지도 모름. 이런 가능성도 배제하지 못 하는 것이 자산 예치 의무기간이 최소 몇달로 상당함
* IBT의 일종인 Principle Tokens(PTs)와 Future Yield Tokens(FYT)는 APWine의 핵심기능이며 기본적으로 자산을 예치하면 PT를 제공하며, PT와 FYT의 교환비율을 초기에는 1:1인 것으로 파악됨
* FYT의 경우에는 대리 수령이 가능한데, PT를 맡기면 FYT가 민팅되는데, 이 민팅된 FYT 토큰을 받을 주소를 투자가가 원한다면 받을 주소를 임의로 지정할 수 있음. 상속이나, 여러 목적으로 다른 사람 주소로 돈을 보내고 싶은 고객을 유치하기 위해 이런 기능을 만들지 않았나 의심이 됨. 결국 FYT 토큰을 가지고 있으면 몇 개월뒤 예치 PT토큰에 이자를 더한 PT를 수령하고, PT 토큰을 가지고 예치 자산(예: BTC)를 가져갈 수 있음

# Introduction
* 2022.1.5.에 조자아텍(GaTech) SSLab(김태수 교수연구실)의 닉네임 setuid0라는 박사과정 대학원생이 Immunefi에 버그 바운티 프로그램을 통해서 제보
* 버그는 토큰을 소각할 때 발생하는 이자를 빼돌릴 수 있는 취약점
* APWine팀에서 100,000달러를 제보 4일후 보안전문가에게 지급하고 해당 취약점 패치

# Vulnerability
setuid0가 찾은 취약점의 관련 함수로 \_beforeTokenTransfer() 함수를 살펴보면 if 문내 require 조건을 살펴보면, 투자자가 원래 가지고 있던 PT보다 더 많은 토큰을 위임할 수 없다. if 조건문에서는 주소 검증을 실시하는데, 받는 주소, 보내는 주소가 FYT 금고(Vault)주소 이거나 NULL주소인지 여부를 확인한다. 일반적으로 금고(Vault)는 해당 토큰 금융서비스내에서 자산을 저장하는 역할을 담당한다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-02-17-immunefi-apwine-review/beforeTokenTransfer.png"| relative_url}})  |
|:--:| 
| 그림.1 _beforeTokenTransfer 함수 코드 |

ERC20 표준에 따르면 토큰에 소각은 반드시 이벤트를 발생시켜야 하며, 소각된 토큰은 NULL주소로 보내게 된다. 보통 이 기능은 transfer함수를 후킹해서 구현하게 되며, 최종적으로는 소각 토큰을 NULL주소로 보낸다고해서, NULL주소의 토큰이 계속 늘어나지는 않는다. 만약에 NULL주소에 Token이 계속 쌓이게되면 [Polygon 사례](https://bears-team.github.io/security/smart%20contract/immunefi-polygon-review/)같은 취약점 발생시 큰 피해를 입을 수 도 있다. 

\_beforeTokenTransfer의 설계는 지극히 정상적이라고 볼수있는 구조임, 그런데 재미있는 것은 \_burn 함수에서 \_beforeTokenTransfer함수를 호출하는데 ERC20에 따를려면 소각 대상 토큰을 NULL주소로 옮겨야 하기 때문에 \_beforeTokenTransfer(account, address(0), amount);같은 형식으로 호출하지만 해당 함수 내부에서는 if내 구문이 실행되지 않으며 FYT 금고내 유저 정보 또한 업데이트 되지 않고, 또한 적절한 밸런스에 대한 요청인지에 대한 require문도 실행되지 않는다.

개발자는 \_beforeTokenTransfer 함수를 코딩할 때 주소의 적절성 확인 및 밸런스 적절성 확인 이 두가지를 모두 의도하였지만, 실제로 동작하는 코드에서는 
이 점을 NULL 주소에 대한 확인을 하지만, 적절한 밸런스 여부에 대해서는 항상 통과가 된다.!!!

APWine 취약점은 이 점을 활용하여 예치금보다 많은 돈을 인출하게 된다.

## STEP1: 예치(Deposit)
익스(Exploit)의 첫번째 단계는 적절한 규모의 예치를 진행해야 한다. 이 때 신규 계정으로 예치를 진행해야 하며, "해당 이후는 createFYDelegationTo 함수 내 조건문 하나를 우회를 위해서다."라고 immuefi 블로그에서 설명하고 있지만 엄밀히 말하면 반드시 신규계정일 필요는 없다. 그러나 신규 계정으로 진행할 시 조건문의 한 변수를 0으로 만들 수 있어서 조건식 우회가 더 쉬워진다. 블록체인에서 지갑주소 새로 생성하는 것은 매우 쉬운 것이기 때문에, 하지 않을 이유는 없다라고 생각되며 우리가 윈도우나 리눅스내 취약점을 유발할 때 조건식을 간단히 만들어 보다 쉽게 우회하기 위해 조건식 변수들을 0으로 만들려고 하는 노력과 같다. _deposit 단계는 일반적인 이자파밍 서비스 프로토콜과 비슷한데 IBT를 예치하면, IBT만큼의 PT토큰을 찍어내고, 해당 PT 토큰을 FutureVault에 예치하게되면 예치 기간에 따른 FYT 토큰을 받게 된다. 

| ![Image Alt 텍스트]({{"/assets/images_post/2022-02-17-immunefi-apwine-review/deposit.png"| relative_url}})  |
|:--:| 
| 그림.2 \_deposit 함수 코드 |

## STEP2: 위임(Delegation)
앞의 단계에서 일정 금액을 예치한 이후에 FYT토큰을 위임하는 단계를 진행하면 된다. 위임행위와 관련된 함수는 createFYTDelegationTo이다. 해당 함수에서는 지금 지금 현재 위임요청할려는 토큰 양이 현재 보유한 토큰양에서 기존에 이미 위임을 한 토큰양을 뺀 것보다 작은지 확인하는 require문이 존재한다. 신규 계정으로 생성했기 때문에 현재 계정은 기존에 이미 위임한 토큰양이 0이다. 이 조건식을 간단히 하기 위해 신규 계정으로 예치하는 것이다. 보통 위임을 받는 주소는 공격자의 주소로 설정하게 된다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-02-17-immunefi-apwine-review/createFYTDelegationTo.png"| relative_url}})  |
|:--:| 
| 그림.3 createFYTDelegationTo 함수 코드 |

## STEP3: 인출(Withdraw)
마지막 단계는 인출단계이다. 인출과 관련된 함수는 \_withdraw함수이며 코드는 아래 그림과 같다. 아쉬운 것은 해당 함수내에 인출 토큰양에 대한 확인이 없다는 게 아쉽고, pt.burnfrom내에서 호출하는 \_beforeTokenTransfer()이 유효성 확인의 전부라는 사실이 아쉽다. \_beforeTokenTransfer함수의 경우 NULL주소로 인해 수량에 대한 확인을 수행하지 못 하고 넘어가게 되고, 위임한 토큰의 양에 상관없이 예치자는 원래 예치금 만큼 토큰을 받아갈 수 있게 된다. 일정 기간이 지난후 위임받은 FYT 토큰을 가지고 PT 토큰을 바꾼 다음 IBT 토큰을 가져가게 되면, 공격자가 에초에 위힘한 토큰양 만큼 금액을 추가적으로 찾아갈 수 있게 된다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-02-17-immunefi-apwine-review/withdraw.png"| relative_url}})  |
|:--:| 
| 그림.4 \_withdraw 함수 코드 |

최종적으로 STEP#1에서 STEP#3을 계속 반복함으로써 공격자는 일정 기간이 지난후 추가적인 토큰을 확보할 수 있다.

# Fix & Lesson
* \_withdraw 함수내에 인출량에 대한 조건문을 추가하였음
* 취약점 탐지 관점에서 조건문이 올바르게 작성되었는지 여부를 꼼꼼하게 확인해봐야 할 것 같다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-02-17-immunefi-apwine-review/bug_fix.png"| relative_url}})  |
|:--:| 
| 그림.5 수정된 함수 코드 |

# References
* [https://medium.com/immunefi/apwine-incorrect-check-of-delegations-bugfix-review-7e401a49c04f](https://medium.com/immunefi/apwine-incorrect-check-of-delegations-bugfix-review-7e401a49c04f)
* [https://www.apwine.fi/](https://www.apwine.fi/)
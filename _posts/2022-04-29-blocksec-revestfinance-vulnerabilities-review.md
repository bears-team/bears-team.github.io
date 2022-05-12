---
title:  "RevestFinance Vulnerabilities: More than Reentrancy 리뷰"
excerpt: "BlockSec팀 3월31일에 작성한 RevestFinance Vulnerabilities 글에 대한 리뷰입니다."

author: Panda
categories:
  - Security
  - Smart Contract
  - BlockSec
tags:
  - Private Smart Contract
  - Solidity
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2022-05-13 7:00:00 +09:00
lastmod: 2022-05-13 7:00:00 +09:00
sitemap :
changefreq : daily
priority : 1.0

# table of contents
toc: true # 오른쪽 부분에 목차를 자동 생성해준다.
toc_label: "table of content" # toc 이름 설정
toc_icon: "bars" # 아이콘 설정
toc_sticky: true # 마우스 스크롤과 함께 내려갈 것인지 설정
use_math: true #수식
#{% figure caption:"Le logo de **Jekyll** et son clin d'oeil à Robert Louis Stevenson"
#  %}
#  ![Le logo de Jekyll]({{"/assets/images_posts/2021-09-23-cve-2021-31956-part1/1.png"| #relative_url}})
#{% endfigure %}
---
# Executive Summary
BlockSec의 경우는 etherscan 서비스를 기반으로하는 공격 탐지 시스템을 개발하여 이를 활용해서 공격을 탐지하고, 공격 당하는 서비스업체에게 알려주는 서비스가 가능한 업체인데, 개인적으로 BlockSec의 글을 읽을 때마가, 공격인지여부를 판단을 어떻게 하는지 궁금합니다. BlockSec은 RevestFinance 서비스 사례에서 두 가지를 자랑하고 있습니다. 첫 번째는 BlockSec사의 공격 알림 시스템의 공격 포착 능력이고 두 번째는 자신들의 유효한 제로데이 탐지능력 이 두가지 입니다. 그럼 이 두가지에 대해서 이 번 블로그를 통해 살펴보로록 하겠습니다.

# Introudction
22년 3월27일에 이더리움기반 DeFi 서비스인 Revest Finance를 대상으로 ERC-1155 callback에 대한 공격이 있었습니다. 해당 공격으로 약 2백만달러 정도의 토큰(BLOCKS, ECO, LYXe, and RENA)가 탈취되었으며, 첫 번째 공격 분석이후 바로 BlockSec팀의 [트윗](https://twitter.com/BlockSecTeam/status/1508065573250678793)이 있었습니다. 이 문장으로 미뤄짐작해보면, 공격에 대한 알람은 자동화된 것으로 추정되고, 분석이후라는 구절을 생각해보면 실제 공격인지 여부는 전문가의 능력에 의존하는게 아닌지 생각해봅니다. 

실제 공격에 대한 트위터를 작성하는 동한 BlockSec팀은 Revest ToeknValut 컨트렉트 기능에 대한 의구심을 가지고 있었다라고 기술하고 있는데, 왜 그런 의구심을 가지게 되었는지는 지 이런 가치판단의 기준이 개인적으로 중요하다고 생각합니다. 혼자서 재미로 취약점을 찾고 분석하는 것과 다르게 회사차원에서 인력을 투입하는 것이기 때문에, 결과론 적으로 제로데이를 찾고, 해당 업체에 정보를 제공함으로써 행복한 결말을 맞이하였지만, 인력만 투입하고 아무런 결과도 못 얻는 경우도 생길 수 있기 때문에, 이 때의 BlockSec팀 리더의 판단의 흐름에 대해서 한 번 BEARS 팀에서 한 번 토의해보는 것도 좋을 것 같습니다. 이러한 의구심을 기반으로 Revest Finance의 컨트렉트의 기능을 분석했고, 앞에서 탐지한 취약점 보다 더 쉽게 돈을 탈취할 수 있는 취약점을 찾을 수 있었고, Revest Finance팀의 신속한 대응으로 큰 돈의 유출을 막을 수 있었다.라고 기술하고 있습니다. 

도입부 글에서 <span style="background-color:#fff5b1">왜 BlockSec팀을 추가적인 분석을 단행했는가?</span> 이 것에 대한 개인적인 의견은 <span style="background-color:#fff5b1">공격 탐지 시스템에 의해 확인된 공격이 Reentrancy 취약점이기 때문에 아닐까?</span>라는 생각을 해봤습니다. <span style="background-color:#fff5b1">그럼 왜 Reentrancy취약점이 문제인가?</span>라는 질문이 생길 수 있는데 이 것에 대한 제 생각은 Reentrancy취약점은 Solidity 취약점 타입중에 x86/x64에 대입하면 strcpy를 활용한 BoF처럼 <span style="background-color:#fff5b1">지금에 와서는 좀처럼 나오지 않는 아주 기초적인 취약점 유형이기 때문입니다.</span> 즉 너무 기초적인 취약점 조차 걸러낼 수 없는 개발자가 작성한 컨트렉트이기 때문에 충분히 다른 취약점이 존재할 수 있다라는 확신을 가지고 BlockSec팀은 분석을 착수한 것이 아닐까?라는 추정을 해봤습니다. 만약에 그렇다면 개인적으로는 합리적인 의심을 기반으로한 접근이라고 생각이 듭니다.

# Background: Revest Finance FNFT
FNTF에 대해서 아래와 같이 설명하고 있습니다.

The Financial Non-Fungible Token (FNFT) of Revest Finance makes the trustless transfer of the future rights to locked assets possible. 

간단히 설명해보면, FNFT는 기존에 immuefi 사례에서 볼 수 있듯이 예치된 자산에 대한 나의 지분률이라고 생각하면 됩니다. locked라는 표현은 일반적으로 stacking하거나 유동성을 공급하면 기본적인 예치기간이 있기 때문이고, [Immuefi의 APWine 사례](https://bears-team.github.io/security/smart%20contract/immunefi-apwine-review/)에서와 같이 예치한 사람이 아니라 다른 사람(대리수령)도 해당 토큰을 기반으로 돈을 찾아 갈 수 있기 때문에 trustless transfer라는 표현을 사용한 것으로 판단됩니다.

FNFT를 생성하는 방법으로 Revest Contract에서 제공하는 함수는 3가지로 소개하고 있습니다.

* mintTimeLock : 일정한 시간이 지나면 FNFT 토큰(지분) 만틈 Vault에서 예치토큰을 찾아감
* mintValueLock : 예치 자산이 사전에 약속된 가격 이상으로 상승하거나 하락할 때 토큰을 찾아 갈 수 있음
* mintAddressLock : 예치 자산을 지정된 주소만 해제할 수 있음

기초가산 해제를 위해 Revest 컨트렉트와 연결된 컨트렉트는 3개이며, 각각의 역활은 다음과 같습니다.

* FNFTHandler : ERC-1155(엔진코인에서 MultiToken관련 표준)를 상속받은 컨트렉트로 유니크 아이로 fnfid를 사용하는데, 토큰을 Lock을 걸어 FNFT를 민팅할 때마다 1씩 증가한다. 새로운 FNFT을 생성/소각하는 방법은 기준토큰 예치를 통한 생성과 기초자산 출금을 통한 소각말고는 방법이 없다.
* LockManager : 묶여있는 기초자산을 출금할 때 락(lock)을 해제할 조건을 기록하고 출금시 해제조건의 부합여부 확인과 같은 기능을 담당함
* TokenVault : 기초자산 토큰에 대한 송수신을 담당하고 각각의 FNFT에 대한 메타데이터 기록을 담당함

| ![Image Alt 텍스트]({{"/assets/images_post/2022-04-29-blocksec-revestfinance-vulnerabilities-review/figure01.png"| relative_url}})  |
|:--:| 
| 그림.1 기초자산에 대한 Lock과정 |

그림 1에서는 사용자 A가 100WETH를 최초 예치하는 과정을 그리고 있습니다. 맨처음 예치하는 것이기 때문에 fntfid는 1이되고, FNFT 토큰은 1:1 교환비율로 Unlock할 수 있는 대상 User A, User B, User C에 각각 50, 25, 25 생성됩니다. 즉 <span style="background-color:#fff5b1">1FNFT = 1WETH</span>가 됩니다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-04-29-blocksec-revestfinance-vulnerabilities-review/figure02.png"| relative_url}})  |
|:--:| 
| 그림.2 기초자산에 대한 Unlock및 출금과정 |

그림 2는 출금과정을 표현하고 있습니다. 사용자 X는 Unlock할 수 있는 대상이 아니기 때문에 거부되는 것을 보여주고 사용자 B의 경우 정당한 Unlock권한이 있으므로, 자기 지분 FNFT 25개 만큼 기초자산 25WETH를 찾아가게 됩니다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-04-29-blocksec-revestfinance-vulnerabilities-review/figure03.png"| relative_url}})  |
|:--:| 
| 그림.3 추가적인 기초자산 예치시 Revest Finance 동작(depositAdditionalToFNFT) 흐름도 |

그림1, 2를 통해 기초적인 동작흐름을 살펴보았습니다. 이제 부터 Revest Finance취약점과 관련 있는 부분을 살펴보도록 하겠습니다. 일반적으로 거의 모든 DeFi들이 동일한 기능을 지원하고 있습니다. 추가적으로 스와핑 풀에 유동성을 공급하거나, 추가적인 스테이킹을 하는 것이 지금 Revest Finance의 경우와 유사한 기능이라고 할 수 있습니다. 우리가 지금부터 살펴볼 그림 3.에서는 <span style="background-color:#fff5b1">함수는 depositAdditionalToFNFT</span> 동작 과정을 그리고 있습니다. 그림 2.에서 사용자 B는 25 WETH만큼의 기초자산을 가져갔습니다. 그래서 Vault에는 75WETH가 남아 있으며, 이 지분을 표현하기 위해 75FNFT가 발행되어 있습이다. 사용자 A가 이 자산에 추가적으로 기초자산을 더 할려고 depositAdditionalToFNFT 함수를 호출 합니다. depositAdditionalToFNFT의 함수 파라미터를 보면 <span style="background-color:#fff5b1">amount</span>이라는게 있습니다. 우리말로 번역하면 amount는 양이 되는데, 그 다음 변수 quantity도 번역하면 양입니다. 저는 개인적으로 이 부분이 조금 헷갈렸습니다. 그래서 이 부분을 실제 [Github에서 소스코드](https://github.com/Revest-Finance/RevestContracts/blob/59b533221f62a9a422a2443f2c34060b4c3fd3d1/hardhat/contracts/Revest.sol#L214)로 확인하고 명확하게 이해할 수 있었습니다. 그림 3.에서 quantity는 amount를 추가할 FNFT 토큰의 개수를 의미합니다. 그림 3.에서 75개면 지금 현재 Vault에 남아있는 전체 fnftid 1을 의미합니다. 그 다음 <span style="background-color:#fff5b1">amount는 한 개의 FNFT에 추가할 가치라고 생각하시면 됩니다.</span> depositAdditionalToFNFT가 호출되지간 1 FNFT는 1WETH였습니다. 이제 1FNFT를 1.5WETH로 만들고 싶습니다. 각 1FNFT의 가치가 0.5WETH씩 더해져야 하는데, 그 때의 양(amount)입니다. amount에 대해서 다들 이해가 되셨을거라 생각합니다. 그러면 사용자 A가 위와 같이 FNFT의 가치를 올릴려고 하면 Vault에 추가해야할 전체 금액은 $$0.5 WETH * 75 = 37.5 WETH$$ 입니다. 사용자 C가 A가 추가적으로 가치를 더 한 다음에 자기 지분만큼 찾아가게 되면 그림 2.의 사용자 B와 다르게 동일한 지분 25FNFT를 출금함에도 $$25FNFT * 1.5WETH/FNFT = 37.5 WETH$$ 을 출금할 수 있습니다. 25WETH를 찾아간 사용자 B가 왠지 손해본 느낌이네요... ㅎㅎ

| ![Image Alt 텍스트]({{"/assets/images_post/2022-04-29-blocksec-revestfinance-vulnerabilities-review/figure04.png"| relative_url}})  |
|:--:| 
| 그림.4 추가적인 기초자산 예치시 Revest Finance 동작(depositAdditionalToFNFT) 흐름도(depositAdditionalToFNFT의 quantity가 현재 발행양 보다 작을 경우)  |

그림.3의 경우는 민팅된 토큰 양만큼 추가하는 과정을 표현한 것이라면, 그림.4의 경우는 민팅된 코인 양보다 작은 양에 대해서 추가적인 예치를 할 때의 과정을 표현하고 있습니다. 이 경우는 먼저 <span style="background-color:#fff5b1">quantity</span>만큼 옛코인(fnftid가 1코인)을 소각처리를 하고 새로운 FNFT(fnftid가 2)를 <span style="background-color:#fff5b1">quantity</span>만큼 발금하고 이 새로운 FNFT의 각 토큰의 가치는 <span style="background-color:#fff5b1">1.5 + 0.5</span>해서 2.0WETH로 설정합니다. 

~~~
// Now, we transfer to the token vault
if(fnft.asset != address(0)){
    IERC20(fnft.asset).safeTransferFrom(_msgSender(), vault, quantity * amount);
}
ITokenVault(vault).handleMultipleDeposits(fnftId, newFNFTId, fnft.depositAmount + amount);
emit FNFTAddionalDeposited(_msgSender(), newFNFTId, quantity, amount);
~~~

# CASE#1: Re-entrancy Vulnerability
이 번 절에서는 Re-entracy취약점에 대해서 살펴보겠습니다. Re-entrancy 취약점은 그림.4에서 그림.7까지의 그림으로 설명되며, 기본적인 개념은 callback 함수 구조와 validation로직의 부재를 기반으로 특정 함수(대부분 출금함수)를 특정조건(Vault내 기초자산을 0)을 만족할 때까지 호출하는 취약점입니다. Solidity 초기에는 많이 유행했던 취약점입니다.


## STEP1

| ![Image Alt 텍스트]({{"/assets/images_post/2022-04-29-blocksec-revestfinance-vulnerabilities-review/figure05.png"| relative_url}})  |
|:--:| 
| 그림.5 공격자가 가치가 0인 FNFT를 민팅(가장 최근 fnftid가 1로 가정)  |

첫번째 단계에서는 <span style="background-color:#fff5b1">depositAmount값을 0으로 하여</span> 두 번 RENA토큰을 기반으로 FNFT를 민팅합니다. 첫 번째에는 2개, 두 번째에는 360,000개 depositAmount값이 0이기 때문에 실제 돈이 들어가야 하는 것은 아닙니다. 웹과 같은 일반 사용자 인터페이스가 아닌 Attacking contract를 통해 직접 mintAddressLock을 호출하는 것으로 판단됩니다. 일단 다음 단계로 넘어가기에 앞서 depositAmount값이 0인 것에 대한 검증을 하지 않는다는게 가장 큰 문제라고 생각이 됩니다. 과거 윈도우 R3와 달리 R0 드라이버 같은 경우 검증을 소흘히 해서 쓰기 버퍼의 주소가 유저영역인이 확인을 하지 않는다는지, 버퍼의 길이가 0인 것을 확인 하지 않는다는지 이런 경우가 있었는데, 요즘은 볼 수 없는 유형의 취약점이 되었는데, depositAmount가 0인지 확인하지 않는 것은 거의 앞에서 언급한 과거 윈도우 커널 취약점과 동일한 수준이라고 생각합니다. <span style="background-color:#fff5b1">뒤에 부분을 확인할 필요가 없이 아주 기본적인 함수 파라미터에 대한 검증이 없다는 것 자체가 가장 문제라고 봅니다.</span>

## STEP2

| ![Image Alt 텍스트]({{"/assets/images_post/2022-04-29-blocksec-revestfinance-vulnerabilities-review/figure06.png"| relative_url}})  |
|:--:| 
| 그림.6 공격자가 두번째 민팅할 때 콜백함수내에서 Revest contract의 depositAdditionalToFNFT함수를 재호출한다.  |

두 번째 단계에서는 _mint함수가 아직 완료되기 전에 depositAdditionalToFNFT함수를 <span style="background-color:#fff5b1">fnftid=1, amount=$$1*1e18$$, quantity=1</span>으로 호출합니다. 여기서 중요한 것은 아직 fnftid(2)로 민팅이 완료되지 않았기 fnftid의 1 증가가 발생하지 않은 상태이기 때문에 앞에서 quantity가 전체 민팅 개수 2보다 작기 조건을 활용하여 다시 fnftid가 2인 토큰을 민팅할 수 있습니다. 앞서 만든 fnfid(1)의 amount가 0이기 때문에 fnftid(2)인 토큰 1개는 <span style="background-color:#fff5b1"> $$0 + 1*1e18$$</span>의 amount로 업데이트 됩니다. 그리고 공격자는 $$1*1e18 RENA$$를 실제 입금하고, depositAdditionalToFNFT함수에서 fnftid가 하나 증가하고 최후의 _mint 함수에 의해 fnftid가 하나 증가하게 됩니다. 여기서 조금 아쉬운 것은 fnftid의 업데이트 위치가 아쉽습니다. 아마도 mint가 정상적으로 완료되는 것을 확인하고 fnftid를 1증가 시키는 것 이 논리적으로 깔끔하기 때문에 이렇게 구현한 것 같은데, _mint함수 호출전에 fnftid를 1증가 시키면 어떨까? 생각을 해봅니다.(실제 패치를 확인해본 결과 과거 Re-entracy취약점 패치와 유사하게 검증로직 추가, fnftid 업데이트 코드 위치변경을 주요 내용으로 패치를 하였습니다.)

그리고 마지막으로 mint함수가 완료되면서 quantity가 360,000으로 업데이트 됩니다. 다시 정리하면 depositAdditionalToFNFT로 amount를 업데이트 하고 mintAddressLock을 통해 quantity를 업데이트 한 것입니다. 이 모든게 mintAddressLock에서 amount가 0인 경우에 대해서 핸들리하지 않아서라고 판단합니다. 과연 amount가 0인 경우가 필요할 때가 언제인지 개인적으로 잘 생각이 나지 않습니다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-04-29-blocksec-revestfinance-vulnerabilities-review/mint.png"| relative_url}})  |
|:--:| 
| 그림.7 OpenZepplelin내 ERC-1150의 mint함수 설명.account가 컨트넥트면 callback함수 핸들러는 구현할 것을 설명하고 있다.| 


## STEP3

| ![Image Alt 텍스트]({{"/assets/images_post/2022-04-29-blocksec-revestfinance-vulnerabilities-review/figure07.png"| relative_url}})  |
|:--:| 
| 그림.8 공격자가 기초자산을 출금한다.  |

이제 모든것이 준비되었습니다. Lock자체도 공격자 주소기반으로 Lock을 걸었기 때문에, 공격자는 언제든지 편안하게 기초 자산을 출금할 수 있습니다. 

## Security Patch

패치를 보면 mint함수에서 _mint함수를 호출하기 전에 검증 및 fnftid 증가등을 하고 있는 것을 확인할 수 있습니다.
~~~
function mint(
    address account, 
    uint id, 
    uint amount, 
    bytes memory data
) external override onlyRevestController {
    require(amount > 0, "Invalid amount");
    require(supply[id] == 0, "Repeated mint for the same FNFT");
    supply[id] += amount;
    fnftsCreated += 1;
    _mint(account, id, amount, data);
}
~~~



# CASE#2: New Zeroday
두 번째 취약점은 BlockSec팀에서 자체적으로 찾은 Zeroday에 대한 설명입니다. 아마도 BlockSec팀에서 지금 Solidity 8.XX 시대에 Re-entrancy취약점이라고? 그럼 뭔가 더 첫번째 수준의 취약점이 더 존재하지 않을까? 이런 관점에서 Revest Finance의 코드를 분석하지 않았을까? 추가적인 코드 감사이유를 생각해봤습니다.  

BlockSec팀은 TokenVault 컨트넥트내의 <span style="background-color:#fff5b1">handleMultipleDeposits</span>함수내에 있는 depoitAmount를 업데이트 하는 부분을 언급하고 있습니다. 아마도 BlockSec팀은 amount를 업데이트하는 모든 함수를 다 뒤져본게 아닌가?라는 생각이 듭니다.

~~~
function handleMultipleDeposits(
    uint fnftId,
    uint newFNFTId,
    uint amount
) external override onlyRevestController {
    require(amount >= fnfts[fnftId].depositAmount, 'E003');
    IRevest.FNFTConfig storage config = fnfts[fnftId];
    config.depositAmount = amount;
    mapFNFTToToken(fnftId, config);
    if(newFNFTId != 0) {
        mapFNFTToToken(newFNFTId, config);
    }
}
~~~
depositAdditionalToFNFT함수에서 handleMultipleDeposits 함수가 호출되는데, 새로운 토큰의 amount업데이트 하는 부분입니다. BlockSec팀에서 전체 프로토콜을 분석했을 때 이 함수의 목적은 신규 NewFNFId의 토큰의 amount를 업데이트 하는게 이 함수의 목적인데, 위 코드를 보면 Old fnftid의 depoitAmount또한 업데이트하는 것을 확인할 수 있습니다.

## STEP1

| ![Image Alt 텍스트]({{"/assets/images_post/2022-04-29-blocksec-revestfinance-vulnerabilities-review/figure08.png"| relative_url}})  |
|:--:| 
| 그림.9 공격자는 amount 0로 360,000 RENA를 민팅한다. |

이 공격에서도 시작은 amount 0으로 민팅하는 것입니다. 토큰을 한 개도 예치하지 않고 FNFT를 민팅할 수 있습니다.

## STEP2

| ![Image Alt 텍스트]({{"/assets/images_post/2022-04-29-blocksec-revestfinance-vulnerabilities-review/figure09.png"| relative_url}})  |
|:--:| 
| 그림.10 공격자는 depositAdditionalToFNFT 함수를 호출한다. |

두 번째 단계에서는 TotalSuppliedFNFT 개수보다 작을 때 depositAdditionalToFNFT함수의 로직을 활용합니다. depositAdditionaToFNFT를 그림과 같이 파마리터 설정해서 호출하면 당연히 전체 발행량보다 적기 때문에 새로운 NewNFTId를 만들게 되고, 그림에서와 같이 fnftid(2)로 새로운 lock을 만들게 됩니다. 그런데 문제가 기존의 Old fnftid(1)의 amount을 0에서 $$1*1e18$$ RENA로 업데이트 하게 됩니다. amount가 $$1.0*1e18$$ 인 36만개의 RENA가 생겼습니다. 첫 번째 Re-entracy취약점 보다 더 쉽습니다. 

## STEP3

| ![Image Alt 텍스트]({{"/assets/images_post/2022-04-29-blocksec-revestfinance-vulnerabilities-review/figure10.png"| relative_url}})  |
|:--:| 
| 그림.11 공격자는 편안한 마음으로 토큰을 찾는다. |

마지막 단계는 편안하게 출금을 하면 됩니다. 

## Security Patch
Old FNFTId의 amount를 업데이트 하든지, New FNFTId의 amount를 업데이트 하든지 둘 중에 한 경우만 업데이트 하도록 변경되었습니다. 제가 볼때에는 amount 0으로 민팅하는 것도 막아야할 것 같은데, 그 내용이 없어서 조금 아쉽습니다.
~~~
function handleMultipleDeposits(
    uint fnftId,
    uint newFNFTId,
    uint amount
) external override onlyRevestController {
    require(amount >= fnfts[fnftId].depositAmount, 'E003');
    IRevest.FNFTConfig memory config = fnfts[fnftId];
    config.depositAmount = amount;
    if(newFNFTId != 0) {
        mapFNFTToToken(newFNFTId, config);
    } else {
        mapFNFTToToken(fnftId, config);
    }
}
~~~

# Conclusion
이 번 블로그에서는 BlockSec팀의 Revest Finance 취약점 분석 및 탐지 내용을 살펴봤습니다.
BlockSec 팀의 분석 글을 처음으로 리뷰를 해봤는데, Immuefi쪽 분석글 보다, 내용이 한결 이해하기 편했습니다. 분석도 그림을 직관적으로 잘 그려서 그런 것 같은데, 앞으로 BlockSec팀의 분석 글중 괜찮은 것을 가능한 많이 리뷰할 수 있도록 하겠습니다.

오랜만에 만난 Re-entrancy취약점 및 이해가 너무 쉬운 취약점을 살펴봤습니다. 블록체인 프로젝트는 수만개가 존재하고, 바이너리 리버싱 기반 취약점 탐지와 같이 코드 감사기반으로 취약점을 탐지할 여지는 아직 많다고 이번 사례를 통해 느꼈으며, 이 글을 읽는 모든 분들이 [immunefi bounty](https://immunefi.com/explore/)에 도전해 봄직 하지 않을까? 생각을 해봤습니다. 

# References
* [https://blocksecteam.medium.com/revest-finance-vulnerabilities-more-than-re-entrancy-1609957b742f](https://blocksecteam.medium.com/revest-finance-vulnerabilities-more-than-re-entrancy-1609957b742f)
* [https://twitter.com/BlockSecTeam/status/1508065573250678793](https://twitter.com/BlockSecTeam/status/1508065573250678793) 
* [https://docs.openzeppelin.com/contracts/3.x/erc1155](https://docs.openzeppelin.com/contracts/3.x/erc1155)
* [https://eips.ethereum.org/EIPS/eip-1155](https://eips.ethereum.org/EIPS/eip-1155)
* [https://docs.openzeppelin.com/contracts/3.x/api/token/erc1155#ERC1155-_mint-address-uint256-uint256-bytes-](https://docs.openzeppelin.com/contracts/3.x/api/token/erc1155#ERC1155-_mint-address-uint256-uint256-bytes-)
* [https://immunefi.com/explore/](https://immunefi.com/explore/)
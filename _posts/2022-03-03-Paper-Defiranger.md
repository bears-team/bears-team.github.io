---
title: "논문 리뷰: DEFIRANGER: Detecting Price Manipulation Attacks on Defi Applications"
author: Grizzly
categories:
  - Paper
tags:
  - Detection
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

## 들어가기에 앞서
개인적으로 스마트 컨트랙트에서 발생할 수 있는 취약점을 자동으로 발견하는데 관심이 많은 편이라, 논문과 깃허브 서칭을 많이 한다. 스마트 컨트랙트의 기능이 점점 더 복잡해지고, 시큐어 코딩 등이 보급되면서 Reentracy 같은 1세대(?) 취약점에 대한 연구가 사그라들고 있다. 이런 흐름에 편승해서 논문을 검색하다가 오늘 리뷰할 논문을 발견했다. 쉽고 내용도 괜찮아서 리뷰해보고자 한다. 논문도 상당히 짜임새있게 잘 썼다.

## Introduction
DeFi에 대해 공격은 direct와 indirect 공격으로 구분할 수 있다.
- direct attack: 공격자가 토큰의 시세를 직접적으로 변조.
- indirect attack: 공격자가 토큰의 시세를 간접적인 방법으로 변조. 쉽게 말해 defi에서 제공하는 금융 거래 기능을 이용해 시세를 변조하는 방법 (lending, AMM, etc...).

이 논문은 아래 3가지를 목표로 한다.
- DeFi에서 price manipulation attack을 탐지.
- 여러 contracts 사이의 수 많은 트랜잭션을 분석해야 함.
- DeFi app의 high-level semantics를 이해해야 함.

이 목표를 달성하기 위한 challenges로 다음 2가지를 제시하고 있다.
1. complicated interactions: DEFI app은 일반적으로 여러 스마트 컨트랙트로 구성되고, 이들 사이에는 많은 트랜잭션이 발생함. 이 복잡한 상호작용을 어떻게 분석할 것인가
2. semantic gap: Ethereum에서 관찰 가능한 raw transaction과 DEFI app이 제공하는 기능 사이에는 semantic gap이 있음. raw transaction에서는 함수의 구조는 알 수 없고, 오로지 매개변수만 알 수 있음.

이 challenges를 해결하기 위한 insights로 다음 2가지를 이야기 한다.
- 어떤 DEFI와 관련된 트랜잭션은 수 많은 내부(internal) 트랜잭션을 유발하지만, 모든 트랜잭션이 분석에 유용하지 않음.
- DEFI semantics를 복원하기 위해 platform-independent 방법을 제안 (semantic lifiting).

이를 바탕으로 다음을 제안한다.
- raw transactions로부터 cash flow tree(CFT)를 구성. CFT는 외부 트랜잭션에 의해 시작된 토큰 거래를 반영. → basic DEFI action
- CFT로 부터 토근 전송에 대한 high-level DEFI semantics를 복원.
    - liquidity mining(deposit tokens), liquidity cancle (withdraw tokens), trade(다른 토큰과 교환)
- 복원된 DEFI semantics로 표현된 탐지 패턴을 이용해 price manipulation attack을 탐지.

제안에 대한 평가(evlauation)를 요약하면 아래와 같다.
- DEFIRANGER 구현.
- DEFI semantics를 정확하게 복원할 수 있는지?
    - Etherscan과 비교했을 때, 더 많은 DEFI semantics를 식별했고, 더 낮은 false-negative rate를 보임.
- real-world의 price manipulation attack을 탐지할 수 있는지(0-day 포함)?
    - 350만개 이상의 트랜잭션을 분석해서 price manipulation attack을 탐지함.
    - 432개의 공격을 탐지함: 4개의 알려진 피해 사례가 포함되어 있었고, 5개의 0-day attack이 발견됨.

## Price manipulation attacks
이 논문에서는 Uniswap V1 protocol을 이용해 탐지하려는 공격을 설명한다.

### Price mechanism of the uniswap protocol
Uniswap V1에서 토큰의 시세는 아래와 같은 공식에 의해 결정된다.

![uniswap](/assets/images_post/2022-03-03-Paper-Defiranger/uniswap.png)

- $F(x)$: 사용자가 토큰 X를 x만큼 거래했을 때, 획득할 수 있는 토큰 Y의 양.
- $R_{X}$,$R_{Y}$: liquidity pool에서 토큰 X와 Y의 잔액.

토큰의 시세가 liquidity pool에 남아있는 토큰 X와 Y의 양으로 결정되기 때문에, pool을 drain할 수 있는 공격자가 토큰의 시세를 증가시키거나 감소시킬 수 있다. ← price manipulation

### Direct price manipulation attack
공격자가 DEFI app에서 적절하게 보호되지 않은 인터페이스를 어뷰징해서 시세를 조작한다.

![figure2](/assets/images_post/2022-03-03-Paper-Defiranger/figure2.png)

일반적으로 사용자가 토큰 X 10개로 토큰 Y 9.9개를 획득할 수 있다.
공격자는 다음과 같은 direct price manipulation attack을 할 수 있다.
  1. 공격자가 토큰 X 900개를 예치해서 pool 내에서 많은 지분을 차지함과 동시에 토큰 Y로 교환. 토큰 X와 Y의 비율이 무너지고, 토큰 Y의 시세가 상승함.
  2. 공격자가 DEFI app의 공개 인터페이스를 호출해서 토큰 X 10개를 팔고, 토큰 Y 2.75개를 획득함. (공격, 토큰 Y의 시세가 더욱 상승, DEFI app이 의도하지 않은 사용)
  3. 공격자가 토큰 Y 473개를 팔고 토큰 X 905새를 획득함. 2번째 단계에서 공격에 의해 토큰 Y의 시세가 더욱 상승했기 때문에 공격자가 차액인 토큰 X 5개를 획득할 수 있음.

### Indirect price manipulation attack
lending app의 경우, 담보물(collateral)의 시세를 계산하기 위해 토큰의 가격을 알 필요가 있다. 만약, 토큰의 시세가 조작 가능하다면, 더 많은 토큰을 대출할 수 있다 (undercollateralization).

![figure3](/assets/images_post/2022-03-03-Paper-Defiranger/figure3.png)

figure 3에서 lending app은 담보물의 시세를 계산하기 위해 AMM의 실시간 토큰 환율을 사용한다.
예를 들어, 초기 토큰 X와 Y의 환율이 1:1이고, 정상적인 상황에서 사용자가 토큰 X 1.5개를 담보하고, 토큰 Y 1개를 대출한다 (collateral rate 150%).
공격자는 다음과 같은 indirect price manipuation attack을 할 수 있다.
 1. price manipulation: 공격자가 많은 토큰 Y을 예치해서 pool에서 많은 양의 토큰 X를 drain함. 일시적으로 토큰 X의 시세사 상승함.
 2. profiting: 공격자가 토큰 X를 담보로 토큰 Y를 대출함. 토큰 X의 시세가 상승했기 때문에 평소보다 더 많은 토큰 Y를 대출할수 있음 (예를 들어, 토큰 X 1.5개를 담보하고, 원래라면 토큰 Y 1개를 대출할 수 있지만, 이경우, 토큰 Y 2개를 대출함).
 3. cost redemption: 토큰 X의 시세를 복원하기만 하면 됨.

## Challenges
### Challenge 1: Complicated interactions
DEFI app은 여러개의 스마트 컨트랙트로 구성되어 있고, 이들 사이에 복잡한 상호작용이 있다.
예를 들어, Harvest 해킹 사건의 경우, 1개의 공격 transaction에 대한 내부 transactiondl 1316개였다.

![figure8](/assets/images_post/2022-03-03-Paper-Defiranger/figure8.png)

### Challenge 2: Semantic gap
이더리움에서 관찰할 수 있는 raw transaction과 DEFI app이 정의한 기능 사이에 큰 semantic gap이 있다.

![figure4](/assets/images_post/2022-03-03-Paper-Defiranger/figure4.png)

DEFI app에서 토큰을 교환하기 위해 1개의 외부 트랜잭션을 발생키면 11개의 내부 트랜잭션이 발생한다.
Etherscan에서는 트랙잭션의 매개변수만 확인할 수 있다. 실제로 DEFI app에서 의미는 Uniswap V2에서 861.95 USDC를 0.5 ether와 교환을 의미한다.

## Solutions
### Method 1: Prunning unnecessary transactions
외부 트랜잭션에 의해 발생한 모든 내부 트랜잭션이 분석에 유용하지 않다. 이런 것들을 제거한다.
예를 들어, Figure 4는 토큰 거래에서 발생하는 트랜잭션인데, 3개의 내부 트랜잭션만 중요하고 나머지는 보조 기능이었다 (적절한 pool 찾기, 토큰 잔액 확인 등).

### Method 2: Recovering high-level semantics
Etherscan이 DEFI app의 semantics를 복원해주지만, false negative rate가 높다.
semantic lifting을 제안했다.

## Methodology
![figure5](/assets/images_post/2022-03-03-Paper-Defiranger/figure5.png)

### Data collection
Google의 BigQuery Ethereum과 같이 트랜잭션을 발생시킬 수 있는 서비스가 있지만, EVM 내부 상태를 알 수 없다.
Ethereum의 전체 노드를 수정된 Geth에 배포해서 재현하고, 필요한 데이터를 수집했다.
 - 트랜잭션의 메타 데이터 수집(매개변수)
 - 내부 트랜잭션이 실행될 때 EVM의 depth 수집. 여러 스마트 컨트래트에서 서로 다른 함수의 트랜잭션 발생 순서를 알 수 있음.
 - 내부 트랜잭션과 이벤트의 실행 순서를 수집. 토큰 전송 순서를 검색할 수 있음.

### CFT construction
토큰 전송에 대한 raw transaction을 DEFI semantics로 끌어올리기 위해 CFT를 생성한다.

![figure6](/assets/images_post/2022-03-03-Paper-Defiranger/figure6.png)

CFT에는 3가지 유형의 노드가  있다.
- Transaction node:
    - 외부, 내부 트랜잭션 모두 표현
    - edge는 부모 트랜잭션과 자식 트랜잭션
- Event node:
    - 스마트 컨트랙트를 실행하기 위한 내부 트랜잭션에 의해 초기화되기 때문에, 내부 트랜잭션의 자식 노드로 표현.
    - 뭔솔?
- Transfer node:
    - 외부 혹운 내부 트랜잭션에 의해 발생되는 Ether 혹은 ERC20 토큰 전송.

Building the tree structure
  - transaction과 event 노드에 대해 상하관계를 따져서 트리를 구성함.

Inserting and replcaing with transfer nodes
  - 구축된 트리에 transfer node를 연결함.
  - 먼저, ether transfer 노드를 연결함.
  - 그 다음, 표준 이벤트인 Transfer를 ERC20 토큰 transfer 노드로 치환.

Prunning tree branches
  - transfer node만 제외하고 나머지를 제거. 나머지 노드는 DEFI의 semantics를 복원하는데 유용하지 않음.
  - 이를 통해 challenge 1를 해소함. 실제로 real-world attack에 대해 94.23% 트랜잭션을 제거할 수 있음.

### Semantic lifting

![table1](/assets/images_post/2022-03-03-Paper-Defiranger/table1.png)

Defi action
  - transfer: spender에서 receipient로 토큰(asset)을 전송하는 것($T$). ERC20에서 spender가 0x0으로 설정되면 token mining을 의미하고($T_{m}$), reciepient가 0x0으로 설정되면 burning을 의미함($T_{b}$).
  - Liquidity mining and liquidity cancle: 사용자는 pool에 liquidity를 제공하는 대가로 LP token을 획득함 (liquidity mining, $LM$). 반대로, LP token을 소비해서 liquidity를 취소(liquidity cancle, $LC$)하고 예치했던 토큰을 회수할 수 있음. $LM$은 사용자의 liquidity($T$) 예치와 LP token minting($T_{m}$)으로 구성됨. $LC$는 LP 토큰 burning($T_{}b$)와 liquidity($T$) 인출로 구성됨.
  - Trade: 일반적인 상황에서, 2개의 transfer가 결합돼 trade가 만들어짐. 서로 다른 asset을 전송하고, pivot address($T1.receipient$ 혹은 $T2.spender$)를 가지고 있음.

Lifting algorithm

![algorithm1](/assets/images_post/2022-03-03-Paper-Defiranger/algorithm1.png)

Algorithm1은 CFT post-order로 순회($\mathtt{LiftLeaves}$)해서 DEFI의 semantice를 복원하는 알고리즘이다. 기본적으로 리프가 아닌 노드에 대해 모든 자식 노드들을 병합할 수 없을 때까지 병합한다.
노드를 병합할 때 2개의 가이드라인을 따른다 ($\mathtt{MergeLeaves}$).
  - 만약 2개의 인접한 리프가 Table 1에서 정의한 DEFI advanced action의 조건에 부합(matching)하면 병합. 즉, CFT에서 raw transaction이었던 노드(transaction, node, transfer)가 Table 1의 조건을 만족하면 DEFI action을 의미하는 추상적인 노드로 치환되는 것. Figure 6에서, 3과 4가 $Tr$로 병합됨.
  - 인접한 2개의 리프 사이에 transfer chain이 있다면, 하나가 다른 하나를 incoporate(일부로 포함?). 예를 들어, 1) A가 B에게 x Ether를 전송. 2) B가 C에게 x Ether를 전송. 이런 경우, 1이 2를 포함 (called incoporating). Figure 6에서 liquidity mining에서 transfer node 0과 1이 transfer chain을 형성하면, LM 노드가 0을 포함.

subtree에서 부모 노드가 1개의 자식 노드만을 갖는다면, 부모 노드를 제거하고 중복되는 이벤트를 제거한다 (lifting & removing).

### Attack detection
앞서 복원한 DEFI의 semantics와 attack pattern을 매칭해서 공격을 탐지한다. 그리고 매뉴얼하게 탐지된 트랜잭션을 확인한다.

![table2](/assets/images_post/2022-03-03-Paper-Defiranger/table2.png)

Detecting direct price manipulation attack
  - direct price manipulation attack에서 역방향 관계인 단계1과 3은 trade $Tr1$과 $Tr2$로 표현됨. 단계2는 $Tr2$나 transfer $Ba$($T,T_{m},T_{b}$를 포함)로 표현됨. trasfer가 liquidity pool에 영향을 미칠 수 있기 때문.

Detecting indirect price manipulation attack
  - indirect price manipulation attack에서 역방향 관계인 trade $Tr1$과 $Tr2$를 식별해서 단계 1과 2를 탐지.

문제는 역방향 관계인 trade가 위와 같은 공격 상황 외에도 자주 나타나고, 이것들이 탐지의 false negative rate를 증가시킨다. 따라서, Arbitrage를 정의해서 제거한다. 관찰 결과, arbitrage는 일반적으로 봇(entry contract, EC)이고, 1가지 유형의 asset만 가지고 있다.
  - arbitrage: 차익거래. 싼 값에 사서 비싼 값에 되파는 거래.
  - 4가지 유형의 트랜잭션을 arbitrage로 정의함.

![figure7](/assets/images_post/2022-03-03-Paper-Defiranger/figure7.png)

### A real example: Harvest hacking incident

위에서 기술한 방법으로 Figure 8에서 Figure 9의 간소화된 CFT를 구성한다.

![figure8](/assets/images_post/2022-03-03-Paper-Defiranger/figure8.png)

![figure9](/assets/images_post/2022-03-03-Paper-Defiranger/figure9.png)

Figure 9에서 역방향 trade인 $Tr1$과 $Tr2$, 공격자($Tr1.operator$)인 entry contract $EC$, 그리고 조작되는 pool($Tr1.pool$)를 식별한다.
liniquidity mining에서 공격자가 60.2m USDC를 third-party app에 예치했다.
Harvest는 69m fUSDC를 mint해서 공격자에게 제공함=한다 ($LM.pool != Tr1.pool$, $LM.reciepient ==Tr1.operator$).

## Evaluation
2가지 관점에서 DEFIRANGER를 평가했다: DEFI action을 정확하게 식별하는지, 실제 price manipulation attack을 탐지할 수 있는지

### DEFI action identification
현시점에서 비교대상으로는 Etherscan이 유일하다.
183,364개의 트랜잭션을 크롤링해서 Etherscan과 비교했다.

![figure10](/assets/images_post/2022-03-03-Paper-Defiranger/figure10.png)

![table3](/assets/images_post/2022-03-03-Paper-Defiranger/table3.png)

- advantages: platform-independent 시스템이기 때문에 새로운 DEFI에도 적용될 수 있음. Etherscan 보다 많은 DEFI action을 식별함.
- disadvantages: 2개 이상의 토큰을 교환하거나 liquidity 공급을 위해 여러 종류의 토큰을 추가하는 경우에 대해 잘못 식별하는 경우가 많았음 (시스템의 설계가 2개의 토큰 교환에 초점이 맞춰져서라고 보임). 실제 공격은 매뉴얼 분석에 의해 최종 확인되기 때문에 이런 false positive rate는 감내할 수 있음.

### Price manipulation attack detection
350,828,625개의 트랜잭션에서 대규모로 공격을 탐지했다.

![table4](/assets/images_post/2022-03-03-Paper-Defiranger/table4.png)

DEFIRANGER는 총 524개의 공격을 탐지했다.
매뉴얼 분석으로 423개의 공격을 최종 확인했다. 여기에는 9개의 공격이 포함되어 있고 이 중 4개는 알려진 공격, 5개는 알려지지 않은 공격이었다 (제보해서 2개의 CVE를 받음).
flase positive는 51개의 arbitrage 트랜잭션(앞선 제안방법이 실패한 경우), 31개의 잘못 식별된 트랜잭션으로 구성되어있었다.

## Attack analysis
확인된 공격에 대한 분석 결과를 제시한다.

### Vulnerability root cause analysis
분석 결과 확인된 공격은 2가지 카테고리로 분류할 수 있었다: insufficient access control, insecure price dependency between DEFI apps (그냥 direct와 indirect를 다른 말로 표현한 거 같은데).

- Insufficient access control
    - Loopring, Dracula, Seal Finance, Metronome은 잠재적으로 direct price manipulation 공격이가능함.
    - AMM의 시세를 바꿀 수 있는 함수에 대해 access control이 적절하게 적용되지 않음.
    - 이 app들에는 4개의 취약한 함수가 있음: sellTokenForLRC, drain, breed, closeAuction
    - 처음 3개는 공격자가 liquidity pool에 암호화폐를 팔 수 있게함.
    - 마지막 함수는 Metronome의 AMM pool에 직접적으로 ether를 예치할 수 있게 함.
- Insecure price dependency
    - 2가지 유형의 취약한 app들에 대해 잠재적으로 indirect price manipulation attack 가능했음: portfolio management apps, lending apps.
    - Depending on AMM’s real-time quotation
        - Harvest, Plouto, Value portfolio management
        - 요약하면, AMM의 실시간 시세는 공격자에 의해 조작될 수 있음.
        - 이 유형의 공격을 방지하기 위해 일정 기간 동안의 평균 시세를 사용하는 것이 바람직함.
    - Depending on AMM’s real-time reserves
        - Cheese Bank, WarpFinance, lending apps
        - real-time의 AMM reserve에 의존하는 것은 문제가 있음.
        - 공격자가 담보물의 가지를 증가시킬 수 있음. 따라서, 대출금을 상환하면서 더 많은 자산을 돌려 받음.

### Footprint analysis
공격자들은 대부분 비슷한 방법으로 공격을 진행했다.

![figure11](/assets/images_post/2022-03-03-Paper-Defiranger/figure11.png)

- prepare a malicious contract: 공격자가 EOA(External Owned Account, attack operator)를 통해 malicious contract를 배포
- attack the vulnerable apps: attack operator가 malicious contract를 호출해서 공격을 수행함. 이때 많은 경우 flashloan이 사용됨.
- launder profit: attack operator가 EOA로 획득한 자금을 전송함(token launderer). token launderer는 tornoda나 Ren(cross-chain application) 같은 mixing application을 이용해 자금을 세탁함.

이런 방법은 공격자를 추적하기 어렵게 한다.

### Impact Anlysis

![figure12](/assets/images_post/2022-03-03-Paper-Defiranger/figure12.png)

공격이 발생한 후 5개의 취약한 DEFI app의 시장 가치에 대한 영향을 분석했다 (USD 기준).
Harvest와 Value는 공격이 발생한 직후 시세가 떡락했고, Loopring, Dracula, 그리고 Seal Finance는 공격 의심 하루 뒤에 시세가 떡락했다.
시세 하락이 공격과 연관성이 커보이지만, 속단할 수는 없다. Loopring, Dracula, Seal Finance에 대한 공격은 0-day 공격이었다.

## 마치며
당장 작년까지만 해도 Reentrancy나 freezing ether와 같은 낮은 수준의 스마트 컨트랙트 취약점을 발견하기 위한 연구들이 꽤 많이 제시됐다. 물론 이런 취약점을 찾는 것도 중요하지만, 요즘에는 여러 스마트 컨트랙트가 연결되고 점점 더 복잡한 기능이 구현되면서 app이 가진 기능에 대한 고수준의 이해가 필요하게 됐다. 이 논문은 트랜잭션에서 app의 기능적 의미를 systematic하게 도출했다는 점에서 의의가 있다고 본다.

## References
- Wu, Siwei, et al. "Defiranger: Detecting price manipulation attacks on defi applications." arXiv preprint arXiv:2104.15068 (2021) [doi](https://doi.org/10.48550/arXiv.2104.15068)
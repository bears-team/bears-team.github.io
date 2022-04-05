---
title:  "Cyclic Arbitrage in DEX 리뷰"
excerpt: "Cyclic Arbitrage in DEX 논문을 DEX Bot 학습 목적으로 살펴본 내용임"

author: Panda
categories:
  - System Trading
  - Smart Contract
tags:
  - Ethereum
  - DEX
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2022-04-07 9:00:00 +09:00
lastmod: 2022-04-07 9:00:00 +09:00
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
이 문서는  [Cyclic Arbitrage in Decentralized Exchanges](https://arxiv.org/pdf/2105.02784.pdf) 논문을 읽고 BEARS 팀내 기술 세미나 자료로 활용하기 위해 작성되었으며, 이 논문을 분석한 목적은 DEX내에서 시세차익(Arbitrage)를 얻을 수 있는 방법에 대한 정보 수집을 목적으로 분석하였다.

# Introduction
논문에서는 먼저 블록체인 생태계에서 Decentralized Finance(DeFi)의 비중이 증가(2021년 10월까지 1200억달러)하고 있음을 언급하면서 이 DeFi 환경의 핵심 서비스로 DEX를 언급하고 있다. 논문에서 중심으로 삼고 있는 DEX거래소는 Uniswap 거래소이며, 해당 거래소는 Constant Product Market Maker(CPMM) 거래소의 대표격이다. 그 전에 DEX 거래소는 Automatic Market Maker(AMM) 알고리즘의 기반으로 토큰간의 상대적 가치를 결정, 이를 통한 거래를 하게 되는데 기본적인 DeFi의 개념이 없다면 BEARS팀의 Grizzly가 작성한 [내 마음대로 정리하는 DeFi 용어들](https://bears-team.github.io/commons/defi-term/)를 한 번 읽어 보는 것을 추천한다.

논문에서는 삼각 시세차익(Triangular Arbitrage)를 염두하고, 접근 한 것으로 보인다. 삼각 시세차익이라는 용어는 본 논문의 참고문헌을 기준으로 서로 다른 국가에 위치한 중앙화거래소(CEX)간 시간차를 활용한 시세차익을 의미하는 것으로 판단되며, 본 논문에서는 한 DEX 거래소내 3종류의 토큰 A, B. C가 존재할 때 $A \Leftrightarrow B$, $B \Leftrightarrow C$, $C \Leftrightarrow A$ 순서로 스왑핑(swapping) 거래가 이뤄 질 때 발생하는 시세차익을 삼각 시세차익으로 정의하고 있다고 봐도 된다. CEX를 활용한 시세차익에서는 거래소가 삼각형의 꼭지점이 되는 것이고, 본 논문 DEX에서는 토큰이 꼭지점에 해당한다.  

## Problem
이 논문에서 해결하고자 한 문제는 <span style='background-color: #fff5b1'>한 DEX 거래서내에서 시세차익을 유발하는 거래를 시스템적으로 발견하는 방법론 및 그 때의 최적화된 토큰 거래량 결정 방법 도출</span>이라고 할 수 있다. 이를 위해 Uniswap이라는 거래소를 목표로 설정하였고, 실제적으로 20년 5월4일부터 21년 4월15일까지의 Uniswap에서 발생하는 토큰 거래를 분석하였다. 아래 그림.1의 경우 논문에서 제시하고 있는 Uniswap에서 시세차익 거래를 표현한 것이며, 이는 꼭 Uniswap만 해당하는 것이 아닌, Constant Product Market Maker(CPMM) 알고리즘 방식의 모든 거래소에 해당한다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-03-21-ccs-cyclic-arbitrage-in-dex-review/figure01.png"| relative_url}})  |
|:--:| 
| 그림.1 Uniswap-V2에서의 시세차익의 예 |

## Contribution
저자들이 얘기하는 본 논문의 기여부분은 다음과 같다.
* 암호자산 시장에서의 시세차익 거래 동작에 대해서 이론과 경험적 분석을 바탕으로한 첫번째 연구
* DEX 거래소내 순환 시세차익 거래에 대한 최신 평가방법 제공
* 블록체인 기술내에서 사용자들이 다양한 매매 전략을 사용하고 있음을 밝혀내고, DEX 거래소를 활용한 사용자들이 금융 거래에 대해서 이해도를 높임

## Limitation
위에서 본 논문의 한계점은 범위라고 얘기할 수 있다. [SoK: Decentralized Exchanges (DEX) with Automated Market Maker (AMM) Protocols](https://arxiv.org/pdf/2103.12732.pdf) 논문을 보면, DEX 거래소에서 사용하고 있는 Automatic Market Maker(AMM) 알고리즘 종류가 다양한 것을 확인할 수 있다. 그러나 본 논문에서는 CPMM 알고리즘 경우에만 적용할 수 있는 알고리즘을 제안하고 있어 해당 알고리즘을 적용할 수 있는 대상이 제한적이다. 또한 분석 대상 거래소가 Uniswap뿐인 것도 조금 아쉽다. 현재 DEX 거래소는 1천여개가 넘어 간다고 알려져 있고, 이더리움 가스비 문제로 솔라나(Solana), 테라(Tera), 폴카닷(Polkadot)과 같은 체인에서도 DEX 거래소가 많이 생겨나고 있으며, 이 것을 [ImmuneFi Bug Bounty 사이트](https://immunefi.com/explore/)에서도 확인할 수 있다. 그리고 Uniswap의 경우 swapping때의 CPMM의 문제를 보완하기 위해 Uniswap-V3를 출시한 상태이며 본 논문은 Uniswap-V2를 대상으로 하고 있는 점도 아쉽다. 따라서 Uniswap-V3에서 토큰 스왑핑(Swapping)만으로 해당 논문에서와 같이 시세차익이 유발할 수 있는지에 대해서는 추가적인 조사가 필요해 보인다. 

## Future Research Topics
한계점을 바탕으로 DEX내 시스템 트레이딩을 위한 연구(조사/분석 포함) 주제를 생각해보면 다음과 같다.
* 솔라나, 테라, 폴카닷과 같은 네트워크에서의 시스템 트레이딩 봇들이 시세차익을 만들어 내는 거래 분석 즉 Cyclic 형태의 트랜젝션을 식별하는 방법 확보를 기반으로 각 체인별 특징을 식별하고, 시세차익 발생 조건식을 도출하는 것을 앞으로의 연구주제로 생각해볼 수 있다.
* 초기 CPMM 알고리즘이 아닌 실제 보완된 CPMM알고리즘을 비롯하여 다양한 거래소에서 사용되고 있는 AMM 알고리즘에 대한 시세차익 판단 알고리즘 도출이 필요하다. Uniswap-V3를 비롯하여 실제 각 DEX 거래소에서 사용하는 AMM 알고리즘에 대한 시세차익 유발 조건에 대해서 확인이 필요하다.
* 한 DEX 거래소내에서의 토큰 스와핑(Swapping)을 통한 시세차익 뿐만 아니라, 같은 메인넷상 동일한 범주의 AMM 알고리즘을 사용하는 DEX간의 시세차익을 활용해서 시세차익을 만들어 내는 방법, 다른 범주의 AMM 알고리즘을 활용하는 DEX간의 시세차익을 만들어내는 방법, 마지막으로 Flashloan과 같은 블록체인 네트워크에서만 존재하는 특별한 기능을 활용하여 시세차익을 만들어내는 방법

위와 같이 아직 DEX내 시세차익관련해서 연구해야할 주제는 많이 있는 것 같다.

# Background

## CPMM(Constant Product Market Maker) : Uniswap V2
[UniswapV2 swap function](https://github.com/Uniswap/v2-core/blob/4dd59067c76dea4a0e8e4bfdda41877a6b16dedc/contracts/UniswapV2Pair.sol#L159)

$$ 
\lim_{x\to 0}{\frac{e^x-1}{2x}}
\overset{\left[\frac{0}{0}\right]}{\underset{\mathrm{H}}{=}}
\lim_{x\to 0}{\frac{e^x}{2}}={\frac{1}{2}}
$$

## Arbitrage in Cryptocurrency Markets
논문의 해당 절에서는 가상화폐 시장에서 시세차익을 유발하는 거래에 대한 기존 연구를 정리하고 있는데, 이 부분에 대해서는 주제별 관련 논문을 나열함으로써 간략하게 정리하고자 한다.

# Cyclic Arbitrage Model

# Cyclic Arbitrage Opportunities
이 절에서는 순환 매매를 통한 시세차익 거래 기회가 존재하는지에 대한 실제 데이터 분석 결과를 설명한다.

### Data Collection
시세차인 거래 기회분석을 위해 이더리움 Uniswap-V2관련 데이터를 수집했으며, V2가 실제로 배포된 시점 2020년 5월 4일 10000835블록부터 해서 2021년 4월 15일 12244144블록까지 Uniswap 거래소에서 발생한 트랜젝션을 수집하였다.

### Profitable opportunities
그림 2.를 보면 수익(Revenue) 만들 수 있는 길이3(ETH부터 시작, ETH-A-B-ETH구조) 토큰 패스의 개수를 시간 축으로 표현한 그래프이다. 해당 그래프를 통해 확인할 수 있는 사실은 <span style='background-color: #fff5b1'>마이너에게 지급해야 하는 가스비 0.0001ETH 이상되는 길이3 토큰 패스는 존재한다.</span>는 것과, 논문에서 설명을 하고 있지는 않지만, 상승장으로 진입이 이윤을 만들어 낼 수 있는 경로의 개수가 증가한다는 것이다. 상승장으로 집입시 DeFi 시장 규모 자체가 커짐으로 좀 더 많은 기회가 생기는 것으로 추정할 수 있다. 

| ![Image Alt 텍스트]({{"/assets/images_post/2022-03-21-ccs-cyclic-arbitrage-in-dex-review/figure02.png"| relative_url}})  |
|:--:| 
| 그림.2 x축은 시간(날짜), y축은 해당 수익(녹색은 0.001ETH 이하, 파란색 0.0001ETH 이상)낼 수 있는 길이3짜리 토큰 패스 |

그림 3.은 시간별 순환 시세차익 거래가 만들어 낼 수 있는 수익의 변화와 단일 거래도 만들 수 있는 최대 수익에 대한 그래프이다. 규모적인 측면에서 당일 거래도 호황기일 때 100ETH 약 지금 시세로 4억 정도 만들어 낼 수 있음을 볼 수 있다. 그림 3에서 2020년 12월부터 본격적인 호황기 진입인데, 전체적인 순환 시세차익 수익이 떨어진 이유와 21년5월이후 수익이 증가한 이유에 대한 해석이 있었으면 했는데 이 두 시점에 대한 해석은 논문에 언급이 없으므로, 추가적인 조사가 필요해 보인다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-03-21-ccs-cyclic-arbitrage-in-dex-review/figure03.png"| relative_url}})  |
|:--:| 
| 그림.3 x축은 시간(날짜), y축은 해당 수익규모인데 오렌지색은 각각의 순환 시세차익 거래의 수익의 합, 파란색은 단일 순환 시세차익 거래로 만들수 있는 최대 수익 |

논문에서는 CEX 거래소를 활용한 시세차익 거래 대비 DEX 거래소를 활용한 시세차익 거래의 장점을 아래의 두가지로 설명하고 있다.
* Wider range of arbitage : 취급하는 토큰의 개수가 CEX 거래소 대비 매우 많다는 것이다. 보통 Uniswap과 같은 DEX 거래소 대비 CEX 거래소의 경우 토큰 상장 과정이 까다롭기 때문에 당연히 CEX 거래소에 상장된 토큰 개수가 적다. 따라서 시세차익을 만들어 낼 수 토큰 경로가 DEX가 CEX보다 더 많을 수 있고, 논문에 따르면 Uniswap에서 거래될 수 있는 토큰 조합이 30,000개 세계최대 거래소 Binance에서는 토큰 거래 종류가 400개로 거의 100배 정도 차이가 난다. 반대로 안정성에 대해서 의문을 제기할 수 도 있을 것 같은데 개인적인 견해로는 DEX 순환 시세차익 거래의 시작과 끝 토큰을 CEX 거래소에 상장된 토큰(주로 메인넷 토큰)으로 하면 안정성에 대한 문제도 해결 가능할 것으로 판단됨
* Larger market size : 논문에서는 Makarov와 Schoar의 논문 [Trading and arbitarge in cryptocurrency markets](https://www.sciencedirect.com/science/article/abs/pii/S0304405X19301746)을 근거로 이 부분은 설명하고 이는데, 34개의 CEX 거래소에서 발생하는 시세차익 수익이 4개월 동안 20억 USD인 것에 반해서, Uniswap한 곳에서만 하루에 2천4백만 USD의 시세차익이 존재한다고 한다. 하루에 2천4백만 USD이면 저 중에 정말 작은 부분만이라도 확실하게 먹을 수 있다면, 삶의 자유(Freedom)를 달성 가능한다.

# Cyclic Arbitrage Markets
논문에서는 Arbitrage Markets으로 제목을 정하고 있는데, 개인적으로는 Cyclic Arbitrage Markets으로 제목을 정하는게 맞을 것 같아서 Cyclic Arbitrage Market으로 변경했다. 그 이유는 실제 해당 절에서 저자가 언급하기를 본인들이 수집하고 분석한 것이 얼마나 많은 "cyclic arbitrages trading"이 실제 시장에서 발생하고 있는지 그리고 얼마나 많은 수익이 생겨나고 있는지를 조사한 내용에 대한 것을 기술하기 때문이다.

### Data Collection
Cyclic arbitrage 거래를 식별하기 위해 휴리스틱으로 접근했다라고 표현하고 있는데, 결국 이 부분은 사람이 많은 부분 판단했다라는 것이며, 기본적으로 100블록 이내(약 25분)안에 이뤄지는 모든 거래 정보를 수집하여 $A \Leftrightarrow B$, $B \Leftrightarrow C$, $C \Leftrightarrow A$ 순서로 스왑핑(swapping) 거래 패턴에 해당하는 거래들을 Cyclic arbitrage 거래로 정의한 것으로 판단된다. 여기서 100블록이내의 수행된 트렌젝션으로 제한한 이유는 실제 주관적인 판단이며, 논문 저자들이 DEX내에서 변동성이 큼으로 기회를 발견하고, 토큰 사이의 거래가 25분이 넘어가는 것들은 cyclic arbitrage 거래와 관련 없는 거래일 가능성이 높다고 판단했기 때문이다. 이 부분은 분석자의 관점에 따라 충분이 변경될 수 있는 부분이다.

### Overall statistics
* 292,606개의 Cyclic arbitrage거래 트랜젝션을 찾음, 그중 287,241개는 ETH로 시작함
* 17,189개의 서로 다른 순환 구조가 존재함, 그중에서 ETH-LCX-REVV 구조가 4710번으로 가장 인기가 많은 순환 구조임
* 265개의 순환 구조만 10번 이상 사용됨
* 전체 순환 시세차익 구조중에서 길이가 3인 경우가 약 85%(247,297)를 차지함

### Transacation Number
시장의 급하게 커지거나 비트코인의 가격 변동성이 커지면 트렌젝션 수도 급하게 증가하고, 가격이 어느 정도 안정권에 들어가면 일정 수준으로 계속 거래가 이뤄짐을 알 수 있음

| ![Image Alt 텍스트]({{"/assets/images_post/2022-03-21-ccs-cyclic-arbitrage-in-dex-review/figure04.png"| relative_url}})  |
|:--:| 
| 그림.4 날짜별 순환 시세차익 매매 트렌젝션 개수 |


### Transacation revenue
순환 시세차익 거래로 인한 수익 또한 시장에 규모에 비례하는데, 20년 2분기 하루에 약 10ETH정도인 순환 매매의 이익이 21년에는 1000ETH 이상 증가하는 것을 확인할 수 있다.
논문에서도 특별한 설명이 없다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-03-21-ccs-cyclic-arbitrage-in-dex-review/figure05.png"| relative_url}})  |
|:--:| 
| 그림.5 날짜별 순환 시세차익 수익(검정색:일일수익, 노란색: 누적수익) |

### Transaction cost
아래 막대 그래프를 보면 전체 트렌젝션의 약 48% 정도가 0.02ETH보다 작은 가스비를 지불하는 것으로 나타났음. 전체 수익(34,439.11 ETH)의 약 24.6%(8,458 ETH) 가스비로 지출됨. 요즘 솔라나 네트워크가 많이 느려졌는데, 아마도 솔라나는 트렌젝션 비용이 많이 저렴해서 봇을 마구 돌린다는 얘기가 있음. 특히 민팅있는 날의 더욱더 네트워크가 느려지는 현상이 있음

| ![Image Alt 텍스트]({{"/assets/images_post/2022-03-21-ccs-cyclic-arbitrage-in-dex-review/figure06.png"| relative_url}})  |
|:--:| 
| 그림.6 트렌젝션별 가스비의 분포 |

그림 7.에서는 가스비를 고려한 순 이윤에 대해서 설명을 하고 있는데, 지금도 그림이 직관적으로 이해가 되지 않는다. 논문에서는 해당 그래프를 설명하며, 약 7.1% 트렌젝션의 순수익이 마이너스라고 언급하고, 전체 트렌젝션의 78.4%가 순 이익이 0에서 0.1ETH, 가스비가 0.01에서 0.3ETH에 해당한다고 한다. 파란색 막대와 자주색(?) 빨간색 막대가 무엇을 의미하는지 명확하지 않다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-03-21-ccs-cyclic-arbitrage-in-dex-review/figure06.png"| relative_url}})  |
|:--:| 
| 그림.7 트렌젝션별 순이익 및 수익의 분포 분포 |

# Arbitrage Implementation
이 번 섹션에서는 앞에서 분석한 순환 시세차익에 대한 분석이후, 실제 어떤 식으로 구현이 되어있는지에 대한 분석에 대한 내용입니다. 논문에서는 실제 순환 시세차익 거래에 활용된 방법을 두가지로 구분하고 있습니다. 첫 번째는 <span style='background-color: #fff5b1'>Sequential implementation</span>방법이고 두 번째는 <span style='background-color: #fff5b1'>atomic implementation</span>방법입니다. 기존에 CEX기반의 시세차익 방법에서는 Sequential implmentation방법 밖에 존재할 수 없지만, 블록체인 스마트 컨트렉트 기술을 활용 덕분에 atomic implementation이라는 새로운 방법이 생겼습니다. Sequential implementation에서는 


# References
* [Formal Specification of Constant Product Market Maker Model and Implementation](https://github.com/runtimeverification/verified-smart-contracts/blob/uniswap/uniswap/x-y-k.pdf)
* [An analysis of Uniswap markets](https://web.stanford.edu/~guillean/papers/uniswap_analysis.pdf)
* [Cyclic Arbitrage in Decentralized Exchanges](https://arxiv.org/pdf/2105.02784.pdf)
* [SoK: Decentralized Exchanges (DEX) with Automated Market Maker (AMM) Protocols](https://arxiv.org/pdf/2103.12732.pdf)

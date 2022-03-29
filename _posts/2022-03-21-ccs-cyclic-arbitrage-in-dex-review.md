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

논문에서는 삼각 시세차익(Triangular Arbitrage)를 염두하고, 접근 한 것으로 보이며, DEX 거래소내 3종류의 토큰 A, B. C가 존재할 때 $A \Leftrightarrow B$, $B \Leftrightarrow C$, $C \Leftrightarrow A$ 순서로 스왑핑(swapping) 거래가 이뤄 질 때 
Test $A \Leftrightarrow B$

Test

$$ 
\lim_{x\to 0}{\frac{e^x-1}{2x}}
\overset{\left[\frac{0}{0}\right]}{\underset{\mathrm{H}}{=}}
\lim_{x\to 0}{\frac{e^x}{2}}={\frac{1}{2}}
$$

test

## Problem
## Limitation
## Contribution

# Background
## CPMM(Constant Product Market Maker) : Uniswap V2
[UniswapV2 swap function](https://github.com/Uniswap/v2-core/blob/4dd59067c76dea4a0e8e4bfdda41877a6b16dedc/contracts/UniswapV2Pair.sol#L159)

# Cyclic Arbitrage Model

# Arbitrage Markets

# Arbitrage Implementation

# References
* [Formal Specification of Constant Product Market Maker Model and Implementation](https://github.com/runtimeverification/verified-smart-contracts/blob/uniswap/uniswap/x-y-k.pdf)
* [An analysis of Uniswap markets](https://web.stanford.edu/~guillean/papers/uniswap_analysis.pdf)
* [Cyclic Arbitrage in Decentralized Exchanges](https://arxiv.org/pdf/2105.02784.pdf)
* [SoK: Decentralized Exchanges (DEX) with Automated Market Maker (AMM) Protocols](https://arxiv.org/pdf/2103.12732.pdf)

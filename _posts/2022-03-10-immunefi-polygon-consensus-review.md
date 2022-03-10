---
title:  "Polygon Consensus Bypass Bugfix 리뷰"
excerpt: "Polygon Consensus Bypass Bufix Review 문서 학습 목적으로 살펴본 내용임"

author: Panda
categories:
  - Security
  - Smart Contract
tags:
  - Polygon
  - Logical Bug
  - Critical Severity
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2022-03-16 19:00:00 +09:00
lastmod: 2022-03-17 19:00:00 +09:00
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
이 문서는 Immunefi사의 기술 블로그중 [Polygon Consensus Bypass Bugfix Review](https://medium.com/immunefi/polygon-consensus-bypass-bugfix-review-7076ce5047fe) 문서를 읽고 BEARS 팀내 기술 세미나 자료로 활용하기 위해 작성되었으며, 해당 블로그 내용 공유를 통해 Smart Contract내 취약점 유형을 학습하는데 중점을 두고 작성된 문서입니다.

# Introduction
1월15일 화이트해커 [Niv Yehezkel](https://www.linkedin.com/in/niv-y-b0618551)이 합의 알고리즘을 우회할 수 있는 최약점을 PoC와 함께 제보함. 해당 취약점은 공격자가 악성행위를 하기 위해 필요한 PoS 합의 알고리즘 내에서 확보해야할 Staking Power(지분)를 줄여주는 효과가 있다. 해당 취약점을 통해 무제한적인 인출이 가능하다고 immunefi에서는 설명하고 있으나, 해당 취약점을 활용하기 위한 조건이 좀 까다롭기 때문에 위험도 판정에서 High를 받았고, 보상 또한 $75,000 한화 약 8천만원 정도 된다.  
해당 취약점을 유발(Trigger)하기 위해 필요한 조건으로 아래와 같은 것들을 요구하며, 간단히 말하면 해당 조건을 유지하기 위해 상당한 돈이 필요하다.
* Validator 자리가 비어져 있어야 한다. 그리고 해당 자리를 가지고 유지하기 위해 
* flashbot을 사용해서 채굴자(miner)가 Validator 자리를 계속 차지하게 할려면 상당한 돈을 지불해야 한다.
* 또한 폴리곤의 체크포인트는 매 30-45분마다 발생하며, 공격에 필요한 시간에 비례하여, 비용이 발생하게 된다.

# Background
## Validator
Validator는 폴리곤 블록체인내에서 합의 그룹 업무에 참여하고 이더리움 메인넷에 체크포인트(checkpoints)를 커밋(commit)하는 노드(유닛)을 의미한다. Validator는 트랜젝션을 검증하고, 블록체인에 새로운 블록을 추가하고, 보상을 벌 수 있다.
## Delegator
검증 노트에 토큰을 스테이킹 함으로써 폴리곤 네트워크의 보안성 강화에 기여할 수 있으며, 스테이킹을 통한 보상을 얻을 수 있다.

## 검증자(Validator)는 어떻게 하면 될 수 있나?
위 질문에 대한 간략한 대답은 폴리곤 사이트에서 짧게 아래와 같이 기술하고 있다.  
만약에 Validator 빈 자리가 있다면, 누구나 상당한 양의 스테이킹을 가지고 검증자(validator)가 될 수 있다. 매일 정기적으로 검증자(validator) 선발 경매가 진행되는데, 거기서 경쟁자보다 많은 토큰 스테이깅을 제안함으로써 기존 검증자(validator)를 제치고 그 자리를 차지할 수 있다. 즉 검증자(validator) 자리를 예약하거나 하는 것은 안되며, 누구나 스테이킹양을 가지고 경쟁할 수 있는 열린 구조이다.

## Intro to the Consensus Mechansim
* 합의 알고리즘의 개념을 설명하고 있는, BEARS 블로그내 [합의 알고리즘 기초](https://bears-team.github.io/commons/consensus-algorithm/)을 참고하면 좀 더 쉽게 이해할 수 있다.

# Vulnerability Analysis
* Blob

# Vulnerability Fix & Lesson
* Blob

# References
* [https://medium.com/immunefi/polygon-consensus-bypass-bugfix-review-7076ce5047fe](https://medium.com/immunefi/polygon-consensus-bypass-bugfix-review-7076ce5047fe)
* [https://bears-team.github.io/security/smart%20contract/immunefi-polygon-review/](https://bears-team.github.io/security/smart%20contract/immunefi-polygon-review/)
* [https://polygon.technology/staking/](https://polygon.technology/staking/)
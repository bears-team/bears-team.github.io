---
title:  "Immunefi Review: Fei Protocol Flashloan Vulnerability Postmortem 리뷰"
excerpt: "2021년 5월 2일에 Immuefi에 제보된 Alexander Schlindwein의 Fei Protocol Flashloan 취약점에 대한 리뷰입니다."
author: Panda
categories:
  - Security
  - Smart Contract
  - Vulnerability
tags:
  - Immunefi
  - Flashloan
  - Review
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2022-08-02 7:00:00 +09:00
lastmod: 2022-08-02 7:00:00 +09:00
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
# Introudction
  * [Alexander Schlindwein](https://twitter.com/bobface16)이 2021년 5월 2일에 Immunefi에 제보한 취약점으로 Floshloan기반의 논리취약점. 
  * Alexander Schlindwein의 경우 다수의 DeFi관련 취약점을 제보한 실력이 높은 화이트해커로 DeFi업계에서 명성이 있는 것으로 주변분들의 얘기를 통해 알게됨.
  * 이 취약점 분석이 중요한 이유중 하나로 DeFi관련해서 취약점 분석시, 공격 코드가 공개된 블로그를 한 번도 확인하지 못 했는데, 해당 취약점의 경우 공격코드가 공개된 유일한 취약점 분석 블로그
  * Flashloan 취약점을 실제로 돌려볼 수 있는 소중한 기회라고 생각하고 자세히 분석할 필요성이 존재함

## Summary
  * flashloan 기능을 활용한 공격, 최대 60,000ETH 유출 가능
  * 21년 5월2일에 Alexander Schlindwein이 immuefi에 제보
  * Fei Protocol은 Alexander Schlindwein에게 $800,000 지급
  * 이후 Fei Protocol은 취약점에 대한 분석을 결과를 발표함

# Vulnerability
## Fei Protocol
  * 루나와 같은 알고리즘 스테이블 코인(Algorithmic Stablecoin)
  * Fei의 1달러 가치를 유지하기 위한 방법중 하나로 Protocol Controlled Value(PCV) 활용
  * PCV의 기본 방법은 유동성 공급을 통한 유니스왑(CPMM)의 ETH/FEI 쌍의 적절한 교환비를 유지하게 하여 안정적인 FEI 가치를 지속하게 함
  * ETH의 가치가 높으면 ETH를 ETH/FEI 풀에 공급, FEI가치가 상승하면 FEI를 공급, 어떻게 보면 ETH가 FEI의 담보 물건으로 볼 수 도 있음, 따라서 담보가 가치가 급하락해도 FEI의 1달러 유지를 위해 ETH를 공급해야함, 예 ETH가격이 400만원에서 200만원으로 줄어들었을 때 교환비는 그래로 이기 때문에, FEI의 가치가 0.5달러가 됨 이를 1달러로 끌어 올리기 위해 ETH를 2배 공급하면 ETH대비 FEI의 가치가 올라가게 되면 1달러에 가깝게 유지하게 됨
  * 유니스왑 풀에 적절한 ETH를 공급는 역활을 담당하는 것이 PCV임

## Whay Stablecoin?
  * 1달러 스테이블 코인을 찍어내는 것은 1달러를 찍어내는 것과 같음
  * 가상자산 시장에서 즉 FRB가 되는 것, FEI 프로토콜의 경우 ETH가 계속 상승한다면 그 상승분 만큼 달러를 계속 찍어낼 수 있게 되는 것임
  * 달러를 무한히 찍어낼 수 있는 것의 힘은 클린턴 정부에서 금본위제도를 포기하고 나서의 미국 경제 버블 증가속도를 보면 됨, 단 시장의 신뢰가 필요함 미국은 미국이라는 나라가 절대 망하지 않는다는 사람들의 통념을 기반으로 무한히 돈을 찍어내는 것임, [M1 차트](https://fred.stlouisfed.org/series/M1SL) 참고
  * 여기서 ETH역할을 하는게 미국 국채이고 금본위제도 폐기는 최악의 경우에서도 달러 가치 유지를 위해 달러 발행량 만큼 담보로 금을 보유해야한다는 족쇄를 풀어준 것. 미국 금융권의 요구사항을 클린턴정부가 들어준 것이고, 클린턴 정부가 들어준게 하나가 더 있는데, 일반 시중은행에서 금융상품을 다룰수 있게 해준것 원래는 우리나라 금산분리와 비슷한 취지로 미국도 투자은행과 시중은행의 역할를 엄하게 구분하고 있었음. 이 법안 폐지의 폐단이 들어난 사건이 미국의 서브프라임모기지 사태임. IMF로 우리나라의 중산층이 무너졌다면 서브프라임으로 미국의 중산층이 무너졌음



# Discussion


# Conclusion

# References
* [https://medium.com/immunefi/fei-protocol-flashloan-vulnerability-postmortem-7c5dc001affb](https://medium.com/immunefi/fei-protocol-flashloan-vulnerability-postmortem-7c5dc001affb)
* [https://medium.com/fei-protocol/fei-bonding-curve-bug-post-mortem-98d2c6f271e9](https://medium.com/fei-protocol/fei-bonding-curve-bug-post-mortem-98d2c6f271e9)
* [https://medium.com/immunefi/a-guide-to-reproducing-ethereum-exploits-fei-protocol-224b30b517d6](https://medium.com/immunefi/a-guide-to-reproducing-ethereum-exploits-fei-protocol-224b30b517d6)
* [https://github.com/fei-protocol/fei-protocol-core/pull/81/commits/f10a43d91fce77fe1d03f7914483120698aabc72](https://github.com/fei-protocol/fei-protocol-core/pull/81/commits/f10a43d91fce77fe1d03f7914483120698aabc72)

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
  * 이 취약점 분석이 중요한 이유중 하나로 DeFi관련해서 취약점 분석시, 공격 코드가 공개된 블로그를 한 번도 확인하지 못 했는데, 해당 취약점의 경우 공격코드가 공개된 유일한 취약점 분석 블로그로 생각해도 된다. Flashloan 취약점을 실제로 돌려볼 수 있는 소중한 기회라고 생각하고 자세히 분석할 필요성이 존재한다.

# Vulnerability


# Discussion


# Conclusion

# References
* [https://medium.com/immunefi/fei-protocol-flashloan-vulnerability-postmortem-7c5dc001affb](https://medium.com/immunefi/fei-protocol-flashloan-vulnerability-postmortem-7c5dc001affb)
* [https://medium.com/fei-protocol/fei-bonding-curve-bug-post-mortem-98d2c6f271e9](https://medium.com/fei-protocol/fei-bonding-curve-bug-post-mortem-98d2c6f271e9)
* [https://medium.com/immunefi/a-guide-to-reproducing-ethereum-exploits-fei-protocol-224b30b517d6](https://medium.com/immunefi/a-guide-to-reproducing-ethereum-exploits-fei-protocol-224b30b517d6)
* [https://github.com/fei-protocol/fei-protocol-core/pull/81/commits/f10a43d91fce77fe1d03f7914483120698aabc72](https://github.com/fei-protocol/fei-protocol-core/pull/81/commits/f10a43d91fce77fe1d03f7914483120698aabc72)

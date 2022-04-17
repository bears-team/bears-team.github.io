---
title:  "A survey on private smart contract construction method and business model(Part1)"
excerpt: "Private smart contract를 생성하는 방법에 대한 공개정보를 조사하고 이를 활용한 현행 사업모델에 대한 조사 1부 결과"

author: Panda
categories:
  - Security
  - Smart Contract
tags:
  - Private Smart Contract
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2023-04-21 7:00:00 +09:00
lastmod: 2023-04-21 7:00:00 +09:00
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
이 문서는  [Cyclic Arbitrage in Decentralized Exchanges](https://bears-team.github.io/system%20trading/smart%20contract/ccs-cyclic-arbitrage-in-dex-review/) 논문에서 언급이된 smart contract의 보안성을 높인 private smart contract와 관련된 현재 공개된 기술(기법)에 대해서 조사 및 분석한 결과를 공유하는 글입니다. 글의 전체적인 흐름은 공개 [Medium 블로그](https://medium.com/iovlabs-innovation-stories/private-smart-contract-execution-7df89e28eb30)을 중심으로 뼈대를 잡고, 추가적인 논문 내용이나 조사 내용을 덧붙여서 작성되었습니다. 해당 글은 BEARS 팀 세미나에서 발표된 내용이며 BEARS팀 블록체인 스터디에 참여하고 싶으신 분들은 bears.team.kr[AT]gmail.com으로 연락주시면 감사하겠습니다.

# Introudction
이 글을 읽고있는 분들이라면, 블록체인내 스카트컨트렉트의 속성으로 가장 중요한 것이 투명성이라는 것은 모두들 알고 있으리라 생각합니다. 이러한 투명성을 기반으로 DeFi와 같은 불특정 다수에 의해 금융거래 또한 가능한 것이고, 블록체인 사업이 처음에 기존 사업에 대한 차별성을 이러한 투명성을 바탕을 두고 강조하여 왔습니다. 그런데 이율배반적으로 최근에 블록체인 스마트컨트렉트 생태계에서 "Privacy" 문제가 대두되기 시작했습니다. 즉 블록체인 네트워크를 사용하는 사용자 가운데, 스마트컨트렉트의 내용을 보호하고 싶은 필요성이 생겼습니다. 간단하게 생각해볼 수 있는 예가 지난시간에 제가 정리한 [Cyclic Arbitrage in DEX](https://bears-team.github.io/system%20trading/smart%20contract/ccs-cyclic-arbitrage-in-dex-review/) 글에 나와있는 시세차익 봇들이 좋은 예가 될 수 있습니다. 해당 봇을 운영하는 업체측에서는 자신들이 어렵게 찾아내고 개발한 알고리즘이 대중에 의해 노출되는 것은 회사의 핵심 기술을 잃어 버리는 것임으로, 해당 업체 입장에서는 블록체인의 투명성은 해당 업체관점에서는 가장 큰 위협이라고 판단할 수 있습니다.

스마트 컨트렉트에서의 프라이버시의 구성요소를 3가지로 구분하고 있습니다.
* Privacy of the specification: Private Contract(적절한 번역을 찾지 못 했음)의 소스코드는 배포, 실행, 동기화 동안 은닉되어야 한다.
* Privacy of the execution: Private Contract의 활성화 이후, 실제 해당 컨트렉트에 대한 호출 변수 및 리턴 값들은 은닉되어야 한다.
* Privacy of the state: Private Contract의 내부변수의 상태 값은 특히 사용자 주요 정보는 소중히 다뤄야 한다.

이 후 섹션에서는 해당 블로그에서 소개하는 스마트컨트렉트 프라이버시 관련 서비스를 소개하는 내용을 정리하면서 1부 내용을 마치고, 2부에서는 BEARS 팀에서 관심을 많이 가지고 있는 기존 x86/64 시스템에서 코드 난독화(Obfuscation)와 유사한 성격을 가지고 있는 Ethereum Virtual Machine(EVM) 코드 난독화를 중심으로 관련 논문의 내용을 정리할 계획입니다. 이후 시간을 가지고 [Private Smart Contract Execution](https://medium.com/iovlabs-innovation-stories/private-smart-contract-execution-7df89e28eb30)에서 소개하고 있느 사용 서비스들에 관해 심도 있게 조사하는 글을 작성하도록 하겠습니다.

# Hawk and zkHawk
## Hawk
[Hawk](https://eprint.iacr.org/2015/675.pdf)
## zkHawk
[zkHawk](https://eprint.iacr.org/2021/501.pdf)

# Enigma

# Raziel

# Zether

# Zexe

# Zkay(v1 and v2)

# ShadowEth

# Ekiden

# Arbitrum

# Kachina

# Conclusions

# References
* [https://medium.com/iovlabs-innovation-stories/private-smart-contract-execution-7df89e28eb30](https://medium.com/iovlabs-innovation-stories/private-smart-contract-execution-7df89e28eb30)
* [https://arxiv.org/pdf/2104.09180.pdf](https://arxiv.org/pdf/2104.09180.pdf)
* [Hawk: The Blockchain Model of Cryptography and Privacy-Preserving Smart Contracts](https://eprint.iacr.org/2015/675.pdf)
* [zkHawk: Practical Private Smart Contracts from MPC-based Hawk](https://eprint.iacr.org/2021/501.pdf)
* Source Code Obfuscation for Smart Contracts
* EShield: Protect Smart Contracts against Reverse Engineering
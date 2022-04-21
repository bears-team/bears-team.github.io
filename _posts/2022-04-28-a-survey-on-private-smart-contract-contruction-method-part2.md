---
title:  "A survey on private smart contract construction method (Part2)"
excerpt: "Private smart contract를 생성하는 방법에 중하나인 Solidity 코드 난독화에 대한 논문 2개를 요약정리한 글입니다."

author: Panda
categories:
  - Security
  - Smart Contract
tags:
  - Private Smart Contract
  - Solidity
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2022-04-28 7:00:00 +09:00
lastmod: 2022-04-28 7:00:00 +09:00
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
이 문서는  [Cyclic Arbitrage in Decentralized Exchanges](https://bears-team.github.io/system%20trading/smart%20contract/ccs-cyclic-arbitrage-in-dex-review/) 논문에서 언급이된 smart contract의 보안성을 높인 private smart contract와 관련된 현재 공개된 기술(기법)에 대해서 조사 및 분석한 결과를 공유하는 글입니다. 지난번 [Medium 블로그](https://medium.com/iovlabs-innovation-stories/private-smart-contract-execution-7df89e28eb30) 글에 이어서 이번에는 Smart Contract의 코드 기밀성을 향상기키기 위한 방법으로 우리에게 익숙한 코드 난독화(Code Obfuscation)을 중심으로 관련 논문 2편을 정리할려고 합니다. 해당 글은 BEARS 팀 세미나에서 발표된 내용이며 BEARS팀 블록체인 스터디에 참여하고 싶으신 분들은 bears.team.kr[AT]gmail.com으로 연락주시면 감사하겠습니다.

# Introudction
Private Smart Contract관련 기술조사 1부에서 우리는 여러개의 Private Smart Contract를 지원하는 프레임워크를 아주 간단히 살펴보았다. Private Smart Contract를 지원하기 위한 프레임워크는 기본적인 이더리움과 같은 메인넷 위해서 Private Smart Contract를 지원하기 위한 Layer-2 기능을 수행하는 것으로 파악되었고, 대부분이 이더리움 위에서 동작하는 것으로 판단이 됩니다. 

Private Smart Contract를 지원하는 프레임들은 기본적인 큰 틀에서 접근방향이 유사하다고 볼 수 있습니다. 기존의 Solidity 기반 Smart Contract를 두 개로 구분, 사용자 입력, 주요 데이터 코드의 경우 Private한 장소(TEE, VM, 별도의 네트워크)에서 계산하고 이후 암호화된 계산된 결과를 Public 네트워크(예: 이더리움)에 검증을 받는 구조였습니다. 동작, 금액, 사용자에 대한 기밀성을 보장하기위한 서비스입니다. 이러한 동작방식은 Cyclic Arbitrage in DEX 조사시에는 예상하지 못 했던 것입니다. 이 논문을 분석할 때에는 x86/x64에서의 Themida, VMProtect와 같은 코드 난독화 정도의 기술이 적용된 Smart Contract를 Private Smart Contract라 생각했었습니다. 지금와서 생각해보면 난독화(Obfuscation) 기술의 취지 목적(분석자를 지치게 해서 역공학을 포기하게 만들자!)를 생각해보면 Private Smart Contract와는 약간의 거리가 있을 수도 있다라고 생각이 들었어야 했는데 말이죠...  

그러나 금액 및 송수신자에 대한 정보는 노출되어도 상관이 없고 보호해야할 대상이 Smart Contract내 존재하는 코드, 핵심 알고리즘으로 제한될 경우 Smart Contract 코드 난독화 또한 충분한 가치가 있다고 생각이 듭니다. 예를 들어, 여러분들이 윈도우에서 어떤 프로그램의 기능을 역공학을 통해 알아내야한다고 생각해봅다면, 비슷한 기능을 하는 윈도우 기반 프로그램이 다수 존재할 때 지금 현재 분석을 할려고 하는 프로그램이 Themida로 팩킹이 되어있다면, 팩된 프로그램을 분석하기 보다, 비슷한 기능을 하는 다른 프로그램을 살펴볼 것이며 가능한 팩킹되지 않는 대상을 리버싱을 통해 기능을 분석할 것입니다. 비슷한 예를 블록체인에 찾아보면 다음과 같습니다. 이더리움, 솔라나와 같은 핫한 메인넷에는 시세차익 거래를 하는 다양한 봇들이 있습니다. 이 봇들을 개발하는 회사들은 자기를 봇의 시세차익 거래 알고리즘을 지키고 싶을 것입니다. 만약 x86/x64에서의 UPX 정도로 팩킹된 봇이 있다면 해당 봇은 알고리즘을 다른 경쟁회사에서 헌납하는 경우와 동일한 것입니다. 만약에 Themida나 VMProtect 수준으로 난독화 되어 있다면, 역공학 분석자는 그 시간에 다른 회사 봇을 분석할 것입니다. 상대적인 난이도 차이로 봇의 핵심 기술을 지킨 것입니다. 

저는 기본적으로 1부에서 목록화한 Private Smart Contract 서비스대비 코드 난독화 기술이 실제 메인넷에서 스마트컨트렉트가 배포되기까지 과정이 간단하기 때문에 Private Smart Contract 서비스 사용하는 경우대비 속도적 측면에서 이점이 있을 것으로 판단됩니다. 다만 앞에서도 언급하였지만 코드 난독화가 가지고 있는 태생적인 한계점이 존재하기 때문에 이 점은 Solidity 난독화에서도 동일하게 적용됩니다.

본 블로그에서는 두 편의 Solidity기반 Smart Contract 코드 난독화 논문을 살펴볼 것이고, 한 논문의 경우는 github에 소스코드 또한 공개한 상태입니다. 두 편의 논문을 먼저 읽어본 저 개인적인 느낌은 두 편 모두 뭔가 좀 부족한데?라는 것이며, 그 부족한 부분을 우리 스터디 그룹에서 채워서 괜찮은 난독화 도구를 만들면 되지 않을까?라는 생각을 해봤습니다.  

# Source Code Obfuscation for Smart Contracts

# EShield

# Comparison

# Conclusions

# References
* Source Code Obfuscation for Smart Contracts
* EShield: Protect Smart Contracts against Reverse Engineering
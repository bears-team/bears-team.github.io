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
[Hawk](https://eprint.iacr.org/2015/675.pdf) 원래 목적은 [Zerocash 프로토콜](http://zerocash-project.org/)을 향상하기 위해 설계되었지만, 암호화폐내 프라이버시 트렌젝션을 위해서도 사용될 수 있습니다. Zerocash 찾아보면 [Zerocoin](https://zerocoin.org/media/pdf/ZerocoinOakland.pdf) 연구에서 시작했으며, 소개 글을 읽어보면 익명화 비트코인 개발을 목표로 하고 있음을 알 수 있습니다. 기존에 있던 모네로(Monero)와 목적은 동일한 프로젝트라 생각이 됩니다. 

Hawk 솔루션은 기본적인 접근은 기존의 스마트 컨트렉트를 일단 두 개로 나눕니다. 하나는 public smart contract(이하 공개 스마트컨트렉트)이고 이 스마트컨트렉트는 on-chain 즉 공개 블록체인 네트워크에서 동작하게 됩니다. 다른 하나는 private smart contract(이하 비공개 스마트컨트렉트)이며, 이 스마트컨트렉트는 독자적인 개별 네트워크에서 동작하게 됩니다. 이런 구조로 운영되는 서비스가 실제로 존재하는데 EOSIO기반으로 구현된 기관대상 거래소를 표방하고 있는 Bullish 거래소라고 할 수 있습니다. 기관들은 자기들 거래가 공개되는 것을 왜 꺼려하는지 모르겠으나, 실제 매매행위는 독립 네트워크에서 운영되며, 거래 트렌젝션의 해쉬값만 EOS 네트워크에서 검증하고 있습니다. 믈론 Hawk 프레임워크는 이 작업을 자동으로 지원해 주겠다가 주요 핵심입니다.

여기서 문제인 것은 비공개 네트워크에서의 검증자가 필요로 하게 됩니다. 그리고 이 검증자는 신뢰할 수 있는 대상이어야 하는 것은 당연하고, 해당 트렌젝션을 볼 수 도 있어야 합니다. 단 off-chain 네트워크의 관리자는 비공개 스마트 컨트렉트의 실행 결과에는 영향을 줘서는 안됩니다. 

중요한 것은 이 것들을 세부적으로 어떻게 구현했는가? 인데, 이 부분에 대해서는 해당 블로그에서는 소개하지 않고 있습니다. 즉 개별 논문을 다 분석해야할 것으로 생각이 됩니다. 개별 프로토콜 논문에 대한 분석을 코드에 대한 난독화 정리가 끝나는데로 할 계획입니다. 앞으로 계속 소개되는 솔루션(프레임워크)의 내용은 위에 소개한 Hawk 수준이며, 이 번 글에서는 이런 것들이 있구나 하는 수준으로만 확인하시면 되고, 기술적인 세부사항은 이러지는 글을 통해 확인하시면 될 것 같습니다.

## zkHawk
[zkHawk](https://eprint.iacr.org/2021/501.pdf)는 Hawk에서의 관리자 기능의 불편함을 [multi-party protocol(MPC)](https://en.wikipedia.org/wiki/Secure_multi-party_computation)를 활용해 해결한 프로토콜입니다. 이후 여러 솔루션에서 MPC가 계속 언급이 됩니다. 또한 밸랜스 합계와 같은 기능을 블록체인내에서 증명하기 위한 알고리즘으로 $$\sigma$$-protocol과 homomorphic commitments를 사용했다라고 하는데, 이 부분에 대해서는 논문을 확인해야 정확하게 알 수 있을 것 같습니다.

MPC(다자간 연산)는 1982년 앤드류 야오가 두 명의 백만장자의 재산 대결이라는 문제 기반으로 처음 제시했습니다. 간단히 그 내용을 살펴보면, 백만장자 A와 B 중 누구의 재산이 더 많은지 비교할 때 자신의 재산을 직접 노출시키고 싶지 않은 경우 신뢰가 있는 제3자를 두고 확인하는게 일반적인 방법입니다. 하지만 제3자도 신뢰할 수 없는 경우는 어떻게 해야 할까요? 이때 MPC가 솔루션이 될 수 있습니다. 백만장자 A와 B가 MPC 시스템에 각각 자신의 자산을 공유하지 않고 등록하면, 시스템이 자산의 총량을 연산하여 누구의 재산의 크기가 더 큰지 출력해줍니다. 이런 경우 서로의 재산을 공개하지 않은채 원하는 결과값인 재산의 크기만을 얻을 수 있습니다. 결국 MPC 문제는 위 양자간계산(2PC) 문제를 N명으로 확대한 경우이고, 이를 해결하기 위한 알고리즘이 여러개 존재하는 것으로 알고 있습니다. 이 부분에 대해서는 추후에 좀 더 조사하도록 하겠습니다. 구글의 [Private Join and Compute](https://github.com/Google/private-join-and-compute)도 MPC를 기반으로 한 데이터 공유 기술로 알려져 있습니다.

# Enigma(상용화 서비스)
[Enigma](https://web.media.mit.edu/~guyzys/data/enigma_full.pdf) 또한 위 zkHawk와 유사하게 MPC기술을 기반으로 기존의 블록체인을 보완하는 프레임워크로 알려져 있습니다. 그런데 zkHawk와 다르게 그냥 MPC가 아니라 distributed secure MPC로 소개하고 있습니다. distributed와 secure 단어가 Enigma의 특징을 잘 설명해주고 있는데, 암호화된 사용자 정보들이 작게 쪼개 비신뢰 노드에서 계산되어 그 암호화된 계산결과과 다음 단계로 진행되고 오직 사용자만이 암화화된 결과를 복호화 가능한 구조로 설계되어 있습니다. 

# Raziel
[Raziel](https://eprint.iacr.org/2017/878.pdf) 또한 위에서 소개한 방식과 유사한 시스템입니다. 해당 논문에서 주요 기여를 다음과 같이 소개하고 있습니다. 그냥 블로그 원분을 그대로 가져오겠습니다. 이 쪽으로 지식이 깊지 않기 때문에 세미나 참석자분들의 집단 지성을 빌리겠습니다.

* Practical formal verification of smart contracts: the proofs accompanying the smart contracts can be used to prove functional correctness of computations as well as other properties such as termination, invariants and any other requirements for well-behaved code.
* The code producer can convince the executing party of the smart contract of the existence and validity of proofs about the code without revealing any actual information about the proofs themselves or the code of the smart contract.
* A protocol for secure computation that allows offline parties and private parameter reuse.
* Using zero-knowledge proofs to prove the validity of smart contracts before execution.

그리고 Raziel역시 블록체인 네트워크를 공개/비공개 영역으로 구분하고 있고, 각 영역별 담당하는 역할 역시 위의 시스템과 유사합니다. 여기까지 보면 상용화된 서비스들이 논문으로 시작해서 사업화에 성공한 것 처럼 보이며, 개인적으로 차별화가 잘 보이지 않습니다. 다 비슷비슷해 보입니다.

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
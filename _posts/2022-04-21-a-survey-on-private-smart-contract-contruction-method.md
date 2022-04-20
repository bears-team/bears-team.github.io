---
title:  "A survey on private smart contract construction method (Part1)"
excerpt: "Private smart contract를 생성하는 방법에 대한 정리한 공개블로그 내용을 살펴봄으로써 Private smart contract관련 추가적인 조사 소요를 확인해보는 글"

author: Panda
categories:
  - Security
  - Smart Contract
tags:
  - Private Smart Contract
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2022-04-21 7:00:00 +09:00
lastmod: 2022-04-21 7:00:00 +09:00
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
이 문서는  [Cyclic Arbitrage in Decentralized Exchanges](https://bears-team.github.io/system%20trading/smart%20contract/ccs-cyclic-arbitrage-in-dex-review/) 논문에서 언급이된 smart contract의 보안성을 높인 private smart contract와 관련된 현재 공개된 기술(기법)에 대해서 조사 및 분석한 결과를 공유하는 글입니다. 글의 전체적인 흐름은 공개 [Medium 블로그](https://medium.com/iovlabs-innovation-stories/private-smart-contract-execution-7df89e28eb30)을 요약하는 정리한 것입니다. 대상 블로그에서는 정말 간단히 Private Smart Contract관련 솔류션들을 소개하고 있고, 앞으로 추가적으로 개별 시스템에 대한 세부적인 분석 조사가 필요해 보입니다. 해당 글은 BEARS 팀 세미나에서 발표된 내용이며 BEARS팀 블록체인 스터디에 참여하고 싶으신 분들은 bears.team.kr[AT]gmail.com으로 연락주시면 감사하겠습니다.

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
[Zether](https://crypto.stanford.edu/~buenz/papers/zether.pdf) 시스템은 3가지 정보를 은닉하는데 이 3가지 정보는 전송되는 자산의 크기, 송신자, 수신자입니다. 논문의 참여자의 이메일 주소내 도메인 정보 visa.com을 기반으로 생각해보면 기존 금융거래중 기관과 같이 노출을 꺼리는 대상을 염두하고 설계된 구조인 것으로 추정됩니다. Zether Smart Contract 및 Zether(ZTH)이라는 토큰을 기반으로 설계되어 있습니다. 해당 블로그에서는 기존의 Layer-1 블록체인 위에서 동작하는 Polygon과 같은 구조인지, 아니면 아예 독자적인 Layer-1 체인을 구성하는지에 대해서는 명확하게 설명하고 있지는 않습니다. 그러나 Zether 동작의 예시에서 이더리움 주소가 언급이 되는 것으로 봐서 Polygon과 같이 이더리움기반의 Layer-2가 아닐까?라는 생각이듭니다. 

이더리움 주소 A의 사용자 A는 키쌍을 만들고, ZSC내 공개키를 동봉하여 예치할 이더와 함께 트렌젝션을 생성합니다. 그러면 ZSC는 공개키를 활용하여 익명의 계정을 하나 만들고 예치할 이더와 동일한 가치의 ZTH를 해당 계정에 민팅합니다. 사용자 A는 ZTH를 Zether계정으로 보내게 됩니다. 

# Zexe
[Zxex](https://eprint.iacr.org/2018/962.pdf)는 앞에서 소개한 시스템들과 본질적으로 동일합니다. off-chain에서 계산, 해당 계산의 검증은 on-chain이라는 앞의 시스템과 유사한 구조를 가지고 있습니다. 단 이 논문의 저자는 decentralized private computation(DPC)라는 것을 소개하고 있는데, 핵심은 효율적인 구현을 위해 zk-SNARK라는 최적화 알고리즘을 사용하였고, 타원곡선과 recursive proof composition을 활용하였다라고 되어 있습니다. 이 부분은 논문을 자세히 확인해야할 것 같습니다. 

# Zkay(v1 and v2)
## Zkay v1
[zkay](https://files.sri.inf.ethz.ch/website/papers/ccs19-zkay.pdf) 논문에서 소개하는 핵심 아이디어는 사용자가 은닉변수를 정의할 수 있다, 그리고 이 변수는 public smart contract에서는 암호문 형태로 나타날 것이다.라는 겁니다. 해당 변수에 대한 업데이트는 non-interactive zero-knowledge proofs(NIZKs)를 기반으로 이뤄지게 됩니다. zkay는 자체 컴파일러는 가지고 있는데, 해당 컴파일러를 이용해서 스마트 컨트렉트를 두개로 나누게 되며 [zk-SNARK](https://eprint.iacr.org/2017/540.pdf)기반으로 실행이 됩니다. 

## Zkay v2
Zkay v1은 아래와 같은 한계점을 가지고 있다라고 [Zkay v2](https://arxiv.org/pdf/2009.01020.pdf)를 소개하고 있습니다. 참고로 Zkay v2논문은 v1논문보다 훨씬 내용이 많습니다.

Zkay v1 한계점
* Insecure encryption : NIZK기반 증명을 위해 ZoKrate 프레임워크를 사용했는데, 해당 프레임워크는 실전 암호화 기능을 지원하지 않기 때문에 암호화 관련해서 미약한 점이 있다라고 소개하고 있습니다.
* Missing integrity verification : 배포된 스마트 컨트렉트 바이트코드에 대한 증명을 지원하지 않는 한계점이 존재합니다.
* Undefined behavior : Zkay가 트렌젝션을 변환할 때 Integer underflow/overflow에 검증이 없기 때문에 사용자가 직접 이 부분을 검증해야 한다는 단점이 존재합니다.
* Initialization : Zkay v1은 변수들이 0으로 초기화 되어 있다라고 가정하는데, Solidity는 변수가 선언시 0으로 초기화 되어있는 것을 보장하지 않기 때문에, 초기화 되지 않은 변수는 Zkay v1내에서 예상하지 못 한 결과를 생성할 수 있습니다.

그래서 위 문제를 해결하기 위해서 저자들은 아래의 기능을 Zkay v2에 추가했다라고 합니다.
* 최신 비대칭 하이브리드(asymmetric hybrid) 암호화 기능 추가
* 함수호출과 같은 언어적인 기본적인 기능 추가(암호화폐관련 기능, 은닉 호출 흐름제어, 확장된 데이터 타입 등)
* zk-SNARKs 프레임워크 지원, 아마도 NIZK관련인 것 같습니다.
* 손쉬운 설치과 향상된 에러 메세지
* 컴파일 소요 시간 및 on-chain 비용 감소 

# ShadowEth
[ShadowEth](https://link.springer.com/article/10.1007/s11390-018-1839-y) 논문은 아쉽게도 Springer에 판권을 가지고 있는데 구글 검색을 하면 [이곳](https://www.trustkernel.com/uploads/pubs/shadoweth.pdf)에서 논문 PDF를 구할 수 있습니다. ShadowEth도 앞에 소개한 것들과 크게 다르지 않습니다. 단계를 2개로 구분하고 어떤 비밀정보가 밝혀지지 않도록 공개 블록체인에서 검증을 진행한다.

실제 스마트컨트렉트의 로직들은 off-chain의 trusted executed environments(TEE)에서 실행된다고 합니다. 여기서 얘기하는 TEE가 ARM Cortex의 TrustZone과 같은 것을 의미하는 지는 논문에서 직접확인해야 할 것 같습니다. 

ShadowEth는 이름에서 유추할 수 있듯이 이더리움(Ethereum) 스마트 컨트렉트를 사용합니다. 다만 스마트컨트렉트의 코드 및 데이터의 기밀성을 보장(확보)하기위해 사용자와 TEE사이에 안전한 채널을 구축해야합니다. 실제 스마트컨트렉트가 전송되기전에 암호화 되어야 하며, 정당한 권한을 지닌 수신자에 의해서만 복호화 되어야 합니다.

TEE는 키쌍을 생성하고, 컨트렉트네 중요 정보는 해당 공개키로 암호화되고, 정당한 사용자만 복호화 할 수 있습니다.

# Ekiden
[Ekiden](https://arxiv.org/pdf/1804.05141.pdf)는 위의 ShadowEth와 유사하게 블록체인에서 부족한 기밀성 확보를 위해 Trusted Execution Environments(TEE)를 활용하는 것을 제안하고 있습니다. 기본적인 구성은 2단계 구성으로 앞에서 설명한 모든 블록체인 해결책과 그 궤를 같이합니다. 스마트 컨트렉트의 실행은 TEE의 지원을 받아서 수행하게 되며, 이 실행이 올바르게 되었느지에 대한 검증은 on-chain에서 이뤄지게 됩니다.

Ekiden은 블록체인과 TEE의 하이브리드 시스템으로 소개하고 있으며, CIA를 보장한다고 주장하고 있습니다. Ekiden 시스템의 경우 3가지 구성요소를 가진다고 소개하고 있는데, 각 구성요소는 다음과 같습니다.

* Clients: 스마트컨트렉트의 최종 사용자를 의미합니다. 비밀 정보를 포함하고 있는 스마트컨트렉트를 생성하고 실행하는 주체이며, 스마트컨트렉트 실행에 필요한 계산을 Compute nodes에게 위임합니다.
* Compute nodes: Client로 부터의 요청을 처리하는 부분이며, contract TEE내에서 요청받은 스마트컨트렉트를 실행합니다. 
* Consensus nodes: 합의 알고리즘을 수행하는 블록체인 네트워크를 유지하는 기능을 지원합니다. 상태 업데이트 및 TEE 증명에 대한 유효함을 확인에 대한 가장 중요한 역할을 담당합니다.

블로그에서는 Ekiden의 특징으로 아래의 3가지를 언급하고 있습니다.
* Proof of publication: proof of publication은 검증자 E(TEE contract)와 비신뢰 증명자 P간의 상호작용적 증명을 뜻한다. 증명자 P는 제한된 횟수의 메세지를 publish(적절한 한글 단어가 생각나지 않음)할 수 있기 때문에, 공격자가 쉽게 위조하기가 어렵ㄴ다.
* Key management: 각각의 Ekiden 컨트렉트는 키쌍과 연결되어 있으며, 또한 내부상태 암호화 및 사용자 입력 암호화를 위한 대칭키 또한 가지고 있다.
* Atomic delivery: TEE가 실행되면 두 개의 메세지를 생성하게 됩니다. m1 메세지는 호출자에게 결과를 전달하는 것이고, m2 메세지는 블록체인의 상태를 업데이트하게 됩니다. 여기서 atomic하게 전달되는게 되는게 중요하다고 언급하면서 atomic delivery의 뜻은 m1, m2가 반드시 전달되든지, 전달되지 않았다면 전체 네트워크가 영원히 불가능하던지를 의미한다고 합니다. 즉 네트워크에 문제가 없으면 $$100%$$ 반드시 전달되어야 한다는 의미입니다.

# Arbitrum
[Arbitrum](https://www.usenix.org/system/files/conference/usenixsecurity18/sec18-kalodner.pdf)은 미리 선정된 관리자그룹의 통제내 존재하는 분리된 Virtual Machine(VM)에서 스마트컨트렉트를 구현하게 됩니다. 조사를 해보니깐 실제로 구현되어서 동작하고 있으며 CoinMarketCap에서 토큰이 구현되어 있고, 브릿지 서비스로도 활용되고 있는 것으로 나옮니다.  

검증과정을 위해서 bisection protocol이라는 것을 기반으로 하고 있습니다. bisection protocol에 대해서도 좀 더 자세한 분석이 필요해 보입니다. 일단 블로그 내용을 기반으로 이해를 해보면 bisection protocol은 피자를 공평하게 나누는 문제와 동일한 것 같습니다. 두 사람이 불만이 없이 피자를 나누는 방법은 피자를 자르는 사람과 피자를 나눴을 때 조작을 선택하는 사람을 구분하는 것인데, Arbitrum의 관리자 두명이 있습니다. 그리고 관리자가 증명해야할 대상이 있습니다. 참과 거짓을 두고 두 관리자가 논쟁을 시작하게 되는데, 논쟁 시작전 두 관리자가 약간의 돈을 예치합니다. 증명해야할 대상을 제안하는 쪽인 주장자(asserter)가 되고 다른 한 쪽이 도전자(challenger)가 됩니다. 주장자는 논쟁을 2개로 나눌수 있고, 도전자는 2개로 나눠진 논쟁중에서 하나를 선택할 수 있습니다. 이런 식으로 더 이상 논쟁을 나눌수 없을 때까지 진행하고, 마지막으로 도전자가 선택한 논쟁을 검증자가 검증을 하게됩니다. 만약에 검증결과가 참이되면 주장자가 이기게 되는 것이고 반대의 경우에는 도전자가 이기게 됩니다. 이긴사람은 자기가 예치한 돈을 가져올 수 있고, 또한 진 쪽의 예치금도 절반 가져올 수 있습니다. 진 사람의 나머지 반 예치금은 검증가가 가져가게 됩니다.

위 알고리즘의 경우 참여자가 제한되기 때문에 관리자중 한 사람은 무조건 올바르지 않으면 전체 프로토콜이 무너질 수 있습니다.

여기서 쟁점, 논쟁이 추상적인 개념인데, 이게 코드로는 무엇인지를 확인하는 과정이 필요할 것 같습니다. Arbitrum 프로토콜 시스템에서 필요한 4가지 역할(구성요소)를 살펴보고 다음과 같습니다.

* Verifier: 증명기능을 담당하는 전역 객체(Global Entity)이라고 설명하고 있는데, Global Entity 어떤 의미인지 정확하게 확인이 필요해 보입니다. 검증자는 중앙화된 객체가 될 수 도 있고 다중그룹 합의 시스템(비트코인, 이더리움의 마이너를 의미하는 것 같음)이 될 수 도 있습니다.
* Key: 프로토콜에서 유동성(화폐, currency)를 보유하고 있으면서 트렌젝션을 제안하는 참가자로 공개키의 해쉬값을 이용한 ID를 사용합니다. 공개키와 대응된 비밀키로 서명한 트렌젝션을 제안합니다.
* Virtual Machine(VM): 프로토콜내의 가상 참여자이다.라고 기술되어 있는데 너무 추상적으로 설명하고 있는 것 같습니다. 모든 VM은 자신만의 코드와 데이터를 가지고 있으면, 해당 코드에 기술된 기능을 수행한다고 합니다. 일종의 특별한 트렌젝션으로 생성하고, 자신만의 화폐(currency)를 가지고 있으며 메세지와 이 화폐를 주고 받는다고 합니다.
* Manager: 관리자 그룹은 VM의 진행상황을 모니터링하는 집단을 의미합니다. VM이 만들어지면 해당 VM의 트렌젝션은 관리자집단을 지정하게 됩니다. 각 매니저들은 자신들의 공개키를 기반으로 식별됩니다.

# Kachina
[Kachina](https://eprint.iacr.org/2020/543.pdf)은 Universial Composability(UC)를 기반으로한 이론적인 시스템입니다. 기본적인 개념을 제안하는 논문이라고 보면되고, 이를 실제로 확장해서 구현한 것이 Zkay라고 보면 됩니다. Zkay와 내용이 대동소이함으로 내용 요약은 스킵하겠습니다.

# Conclusions
이 블로그에서 소개하는 시스템들은 우리 학습팀의 관심범위를 벗어나는 것이 아닌가 생각이 들며, 거의 구성자체가 별도의 네트워크 환경을 구축하는 수준이라서 조금 당황스럽습니다.

[Cyclic Arbitrage in Decentralized Exchanges](https://bears-team.github.io/system%20trading/smart%20contract/ccs-cyclic-arbitrage-in-dex-review/)에서 Private Smart Contract를 보고 단순 Solidity 코드의 난독화 정도 생각한 것이 오판이였습니다. Arbitrages 봇을 제작해서 저런 서비스를 사용한다면, 해당 서비스 사용비용까지 포함하여 수익을 창출해야 할 것 같습니다. 생각해보면 코드 난독화라는 것이 시간을 지연할 뿐, 실제적으로 가독화를 애초에 못 하게하는 개념이 아니기 때문에 위 시스템처럼 암화화를 통해 기밀성을 확보하면 확실하게 시세차익 봇의 핵심 알고리즘을 지킬 수 있지 않을까? 싶습니다.

그리고 위 시스템들의 제출 학회를 보면 CCS, USENIX같이 보안쪽으로 Top수준에 제출된 논문인 것을 확인할 수 있으며, 블록체인내에서의 기밀성 확보가 핵심 시버시가 될 것 같은 개인적인 느낌을 받았습니다. 블록체인에서의 검증 기능의 힘만 빌리고 내부 내용은 보호하고 싶은 사람의 욕구는 분명히 존재하기 때문에, 소스코드가 공개되어 있는 모델을 기반으로 심도 있는 분석을 하는 것도 저희 팀에 도움이 될 것 같습니다.

이 후 파트에서는 Solidity 난독화관련 논문에 대한 리뷰를 진행할 계획이며, 난독화도 어떻게 얼마나 복잡하게 하느냐에 따라 분석자를 포기하게 만들어 어느 정도 수준의 기밀성을 확보할 수 있다고 생각합니다. x86/x64에서의 Themida같은 패커를 생각해보면 충분히 납득하실 수 있을 겁니다.

# References
* [https://medium.com/iovlabs-innovation-stories/private-smart-contract-execution-7df89e28eb30](https://medium.com/iovlabs-innovation-stories/private-smart-contract-execution-7df89e28eb30)
* [https://arxiv.org/pdf/2104.09180.pdf](https://arxiv.org/pdf/2104.09180.pdf)
* [Hawk: The Blockchain Model of Cryptography and Privacy-Preserving Smart Contracts](https://eprint.iacr.org/2015/675.pdf)
* [zkHawk: Practical Private Smart Contracts from MPC-based Hawk](https://eprint.iacr.org/2021/501.pdf)
* Source Code Obfuscation for Smart Contracts
* EShield: Protect Smart Contracts against Reverse Engineering
* [https://github.com/eth-sri/zkay](https://github.com/eth-sri/zkay) 
* [Arbitrum 브릿지 사용법](https://www.bao-bab.co.kr/column_board/154887)
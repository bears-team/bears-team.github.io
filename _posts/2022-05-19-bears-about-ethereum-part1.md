---
title:  "About Ethereum Part1"
excerpt: "이더리움과 관련된 기본 지식, 개념을 세미나용으로 정리한 글입니다."

author: Panda
categories:
  - Background
  - Ethereum
tags:
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2022-05-26 7:00:00 +09:00
lastmod: 2022-05-26 20:00:00 +09:00
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
이 포스트에서는 블록체인과 관련하여 기본적인 개념들을 소개할 계획입니다. 다만, 비트코인(Bitcoin)과 같은 전송코인이 중심이 아니라 이더리움(Ethereum), 스마트컨트렉트(Smart Contract)를 중심으로 내용을 살펴볼 예정이며, [ethereum.org](https://ethereum.org/en/developers/docs/intro-to-ethereum/) 사이트에서 기술하고있는 내용을 중심으로, 기타 구글 검색을 통해서 나오는 내용을 추가하는 형식으로 주요 개념들을 정리하였습니다.

# What is a Blockchain?
이더리움 공식 홈페이지에서는 블록체인(BlockChain)에 대해서 간단하게 "네트워크상 존재하고 관리되는 공개 데이터베이스(Database)"라고 기술하고 있습니다. 데이터베이스(이하 DB)라면 학부 수업시간에 배운 트렌젝션(Transcation), 동시화(Sychronization), 무결성(Integrity)과 같은 속성이 이 글을 읽으시는 분들도 머리속에 떠오를 겁니다. 기본적으로 블록체인은 기존 DB와 다르게 동시성 문제는 무조건 직렬화(Serialization)으로 해결하고 있습니다. 무결성지원은 블록 체인에서의 체인(Chain)부분이 지원하고 있다고 봐도 될 것 같습니다. 블록체인에서 블록과 블록이 연결되어 있는데 이 연결성은 암호학적 일반향함수(Crytographic oneway function), 쉽게 말하면 해쉬함수를 기반으로 동작하고 있습니다. 변조된 데이터가 동일한 해쉬결과 값을 가지는 것을 충돌현상(Hash Collision)이라고 하는데, 단 하나의 블록에서 이를 발생 시키는 것은 정말 어렵습니다. 물론 구글에서 SHA2 충돌에 성공하였지만, 블록체인은 SHA2보다 매우 안전한 SHA3를 기본 해쉬함수로 사용하고 있습니다. 하나를 블록은 변조하는 것도 어려운데, 자식 블록의 악의적인 데이터 변조를 위해 모든 조상 블록의 데이터를 변조해야 한다면 블록변조는 실용적 관점에서는 불가능합니다. 이 부분에 대해서는 [Youtube영상](https://youtu.be/SSo_EIwHSd4)을 확인하시면 쉽게 이해하실 수 있습니다.

블록체인을 DB관점에서 바라본다면, 블록(Block)은 트렌젝션 데이터와 현재 블록의 상태를 저장하는 곳이라고 보면될 것 같습니다. 하나의 블록안에는 여러개의 트렌젝션을 담을 수 있고, 이더리움의 높은 가스비를 해결하는 방법으로 한 블록에 다수의 트렌젝션을 담아서 검증하는 방법을 많이 사용됩니다.

## Ethereum
이더리움은 Layer1 불록체인의 일종으로 비트코인과 같이 가치저장용(?), 전송용 블록체인과 구분이 된다. 현재 Layer1 블록체인은 여러 종류가 생겼으며, 최근에 가장 문제가 되었던 LUNA(루나) 또한 Layer1 블록체인이다. 이더리움이 비트코인과 달리, 그 블록체인 위해서 다양한 기능을 가능하게 하는 그 기반은 Smart Contract(스마트 컨트렉트)와 이 스마트 컨트렉트를 실행하는 Ethereum Virtual Machine(EVM)이다. 여기에 추가하자면 스마트 컨트렉트를 기술하기 위한 Solidity(솔리더티)라는 JavaScript비슷한 언어도 존재한다. 스마트 컨트렉트의 실행은 EVM에서 수행되며, 이 스마트 컨트렉트의 수행에 실행, 증명, 검증에 대한 보상으로 ETHER(이더)를 EVM을 구동하는 Miner(마이너)들이 받게 된다. 재미있는 것은 대부분의 Layer1 블록체인들이 EVM을 다 지원한다는 것이다. 따라서 기존에 이더리움 네트워크에서 구동되던 서비스가 이더리움이 자신들의 서비스와 맞지 않다고 판단할 때, 예를 들면 rust(러스트)기반의 솔라나(Solana)와 같은 블록체인으로 그대로 옮겨갈 수 도 있다. 또한 블록체인 앱 개발 회사의 경우 Solidity로 구현 후 동일한 앱을 다양한 체인위에서 서비스할 수 도 있다.

## Smart Contract
그럼 Smart Contract는 무엇인가? 개인적으로는 Smart Contract(스마트 컨트렉트)는 블록체인(이더리움)위에서 돌아가는 하나의 기능(Function) 또는 코드 조각(Code Snippet)이라고 생각한다. 응용프로그램(Application)이라고 설명하고 있는데도 있지만, 개인적으로는 여러개의 스마트 컨트렉트를 구현해서 하나로 통합한 것을 DApp(Decentralized Application)이라고 개인적으로 이해하고 있다. [Immunefi](https://immunefi.com/)에 등록된 다양한 프로젝트를 살펴보면 거의 대부분 프로젝트들이 다수의 스마트 컨트렉트를 활용해서 구현하고 있는 것을 알 수 있다. 앞에서 언급했던 것과 같이 Smart Contract는 EVM위에서 동작하게 되며, 스마트 컨트렉트는 Non Von Neumann Virtual Machine이다. 스마트 컨트렉트가 Non Von Neumann 것은 블록체인에서의 핵심 속성인 실행중에 코드를 변경할 수 없기 때문이다. 즉 코드 및 관련 변수 정보들이 없고 오직 ROM만 존재한다. 따라서 이와 관련된 문제가 있을 수 있는데, 예를 들면 스마트 컨트렉트 업데이트가 필요할 때, 그리고 스마트 컨트렉트의 보안성을 강화하기 위한 난독화를 실시할 때 어려움이 있다. 

아래 EVM의 그림을 보면, 스마트 컨트렉트의 코드(바이트코드) 및 상태정보들이 모두 ROM에 위치하여 실행되는 것을 확인할 수 있습니다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-19-bears-about-ethereum-part1/evm.png"| relative_url}})  |
|:--:| 
| 그림.1 EVM의 간략한 구조도 |

스마트 컨트렉트가 실행되기 위해서는 블록체인 네트워크내 배포(Deploy)가 되어야 하는데 일반적으로 배포는 특수한 주소(0x0 번지)로 스마트 컨트렉트를 전송함으로써 이뤄지게 됩니다. 배포가 완료되면 스마트 컨트렉트도 고유한 주소를 가지게 되며 이 주소가 계정(Account)가 됩니다. 바로 이어서 다시 설명이 됩니다.

## Accounts
이더리움 블록체인에서 계정(Accounts)는 이더(ETHER)를 저장할 수 있는 공간을 의미합니다. 이더리움 블록체인을 이용하는 사용자는 계정(Accounts)을 생성할 수 있고, 이더를 보내거나 받거나 할 수 있습니다. 이렇게 적고보니 계정이라고 번역하는 것 보다 계좌라고 번역하는게 더 가까워 보입니다. 글을 읽으시는 분들이 본인이 더 적절하다고 생각하는 표현으로 사용하셔도 괜찮을 것 같습니다. 이더리움에서 Accounts는 두가지 종류가 있습니다. EOA(Externally Owned Account)라고 해서 일반적으로 사용자가 계정을 만들게 되면 EOA입니다. 그리고 당연하게 스마트 컨트넥트도 Account가 될 수 있습니다. 스마트 컨트렉트로 고유 이더리움 주소를 가질 수 있으며, 이 것은 다양한 DeFi 코드에서 확인이 가능합니다. 따라서 EOA와 EOA, EOA와 스마트 컨트렉트, 스마트 컨트렉트와 스마트 컨트렉드간에 이더의 송수신이 가능합니다.

그럼 이 두 계정의 차이점은 무엇일까요? 이더리움 공식 문서에서는 다음과 같이 이 두 계정의 차이점을 설명하고 있습니다.

### EOA(Externally Owned Account)

* Creating an account costs nothing
* Can initiate transactions
* Transactions between EOAs can only be ETH/token transfer

### CA(Contract Account) : Smart Contract

* Creating a contract has a cost because you're using network storage
* Can only send transactions in reponse to receiving a transaction
* Transactions from an EOA to a contract account can trigger code which can execute many differenct actions, such as transfering tokens or even creating a new contract

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-19-bears-about-ethereum-part1/ca.png"| relative_url}})  |
|:--:| 
| 그림.2 모든 CA는 EOA에 의해 동작하게 된다. |

## Blocks
이더리움 블록체인은 제네시스(최초의 블록)라는 상태에서 시작하며, 제네시스 상태란 어떠한 트렌젝션도 발생하지 않은 최소의 상태입니다. 이 최초의 상태에서 이더리움 블록체인의 상태를 변하게 하는 트렌젝션이 발생하면 t+1 스테이트로 전이(transition)이 발생하게 되며 이런 블록체인 세계의 상태를 변이하는 트렌젝션을 묶어서 검증하고, 기록하게 되는데 이것이 블록이다. 이러한 블록들은 이전 블록의 해쉬값을 기반으로 연속적으로 연결되어 있습니다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-19-bears-about-ethereum-part1/blocks01.png"| relative_url}})  |
|:--:| 
| 그림.3 모든 블록들은 해쉬값을 기반으로 체인구조로 연결되어 있다. |

블록에 대한 검증(Validate)를 이더리움과 같은 PoW에서는 마이너라고 불리는 검증자 노드에서 수행하며, PoS에서는 검증자(Validator)라고 불리는 노드에서 수행하게 됩니다. 이더리움에서 마이너는 트렌젝션이 가지는 가스비와 마이닝의 보상인 이더를 가질수 있고, 이 보상을 획득할려는 마이너의 경쟁을 구도 속에서 분산화라는 목적을 달성할 수 있습니다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-19-bears-about-ethereum-part1/blocks02.png"| relative_url}})  |
|:--:| 
| 그림.4 블록 헤더내 속성(Properties). |

* parentHash : 부모 블록의 해쉬값
* nonce : mixHash와 결합하여 현재 블록이 충분한 연산을 행했음을 입증하는 해쉬, PoW에서 nonce값을 찾는 해쉬연산을 마이너들이 수행하게되며, 이는 무작위공격(brute-forcing)을 통해서 획득합니다.
* timestamp : 현재 블록 시작시 unix 타임 스탬프
* ommersHash : 현재 블록의 ommers 해쉬값의 리스트
* beneficiary : 이 블록 채구에 대한 수수료를 받을 account 주소
* logsBloom : log 정보를 구성하는 bloom filter라는 자료 구조의 형태
* difficulty : 블록 생성 난이도, 난이도는 고정이 아니라 블록체인의 혼잡도와 같은 블록체인 상태에 따라 난이도의 변화합니다.
* extraData : 블록과 관련된 정보를 저장하는 임의 바이트 배열
* number : 현재 블록의 count(제네시스 블록이 0일 때, 이후 블록들에 대해서 이 값이 하나씩 증가)
* gasLimit : 블록 당 현재 GAS 제한량
* gasUsed : 현재 블록에서 사용된 가스의 총량
* mixHash : nonce와 더블어 현재 블록이 충분한 연산이 실행되었음을 입증하는 해쉬값, Miner들은 mixHash값을 결과로 하는 입력 nonce값을 찾아야 하는 문제를 풀어야하고, 이 문제는 일방향 함수(oneway function) 해쉬 알고리즘을 기반으로 하기 있기 때문에 이런 계산에 최적화된 고성능 GPU가 CPU보다 유리하기 때문에, GPU 품귀 현상을 유발하였던 것 입니다. 좀 더 추가적으로 설명하면, NVidia, Intel, AMD에서 채굴용 AP 제품을 출시 했으나, 인기가 없었는데 그 이유는 채굴 사업이 망했을 때 채굴용 AP는 중고로 판매할 수가 없으나 고성능 GPU의 경우 중고로 판매가 수월하기 때문에 채굴 사업자 관점에서 GPU가 리스크 관리 차원에서 더 좋은 선택이 됩니다.
* stateRoot : 상태 트리의 루트 노드 해쉬값
* transactionRoot : 이 블록에 포함된 모든 트랜젝션을 포함하는 트리 자료구조의 루트 노드의 해쉬값
* receiptsRoot : 이 블록에서의 모든 트랜젝션의 Receipt을 포함한 트리의 루트노드의 해쉬값

## Transcations
이더리움 공식 홈페이지에서는 트렌젝션을 EOA가 유발한 어떠한 행위를 의미한다라고 기술하고 있습니다. [Matering Ethereum](https://github.com/ethereumbook/ethereumbook)도서에서는 트렌젝션을 EOA에서 생성한 서명된 메세지로서 이 메세지는 이더리움 네트워크를 통해 전달되며, 이더리움 블록체인에 기록이 된다라고 설명하고 있습니다. 결국 이 트렌젝션에 의해서 블록체인 네트워크의 상태가 변화하게 되는 것이고, 이 블록체인 세계의 상태값들은 EVM에서 스마트 컨트렉트 바이트코드를 실행할 때 영향을 주게 됩니다. 

블록체인 네트워크에서의 트렌젝션은 직렬화되어서 처리됩니다. 즉 트렌젝션을 동시에 처리하는 것은 없습니다. 기본적으로 Step By Step으로 처리하게 됩니다. 이더리움내 트렌젝션은 다음과 같은 속성들을 가지고 있습니다.

* Nonce : x86/x64 시스템에서도 네트워크 프로토콜 관련된 공격중 Replay Acttack을 방지하기 위해 Nonce를 설정하는데, 이더리움 트렌젝션에서도 동일한 목적으로 Nonce를 활용하며, 여기에 추가적으로 순차적인 트렌젝션 처리를 위해서 Nonce를 사용한다. 암호학에서 처럼 의사 랜덤을 활용해서 Nonce값을 설정하기 보다, 이더리움에서는 생성된 트렉젝션 순서에따라 Nonce 값을 1씩 증가시킨다. 
* Gas price : 발신자가 지급하는 가스의 가격
* Gas limit : 이 트랜젝션을 처리하기 위해 소비될 최대 가스양 
* Recipient : 트렌젝션의 목적이 이더리움 주소
* Value : 이 트랜젝션을 통해 전달할 이더(ETHER)
* Data : 가변 길이 바이너리 데이터 페이로드
* v,r,s : EOA의 ECDSA 딪털 서명의 세가지 구성요소

# Conclusion
이 번 포스트에서는 스마트 컨트렉트 기반 블록체인의 대표인 이더리움에 대해서 알아보고, 관련된 주요 키워드들의 개념을 전체적으로 간단히 알아보았습니다. 다음 포스트에서는 각 개념들이 실제로 어떻게 블록체인 내에서 존재하는지 geth환경내에서 살펴보고 분석해보는 시간을 가지도록 하겠습니다. 읽어주셔서 감사합니다. 

# References
* [https://ethereum.org/en/developers/docs/intro-to-ethereum/](https://ethereum.org/en/developers/docs/intro-to-ethereum/)
* [https://ethereum.org/en/developers/docs/blocks/#:~:text=Blocks%20are%20batches%20of%20transactions,derived%20from%20the%20block%20data.](https://ethereum.org/en/developers/docs/blocks/#:~:text=Blocks%20are%20batches%20of%20transactions,derived%20from%20the%20block%20data.) 
* [https://ethereum.org/en/developers/docs/dapps/](https://ethereum.org/en/developers/docs/dapps/)
* [https://ethereum.org/en/developers/docs/transactions/](https://ethereum.org/en/developers/docs/transactions/)
* [https://andersbrownworth.com/blockchain/](https://andersbrownworth.com/blockchain/)
* [https://medium.com/@eiki1212/ethereum-block-structure-explained-1893bb226bd6](https://medium.com/@eiki1212/ethereum-block-structure-explained-1893bb226bd6)
* [https://medium.com/remix-ide/the-anatomy-of-a-transaction-receipt-d935aacc9fcd](https://medium.com/remix-ide/the-anatomy-of-a-transaction-receipt-d935aacc9fcd)
* [https://minstar0410.tistory.com/28](https://minstar0410.tistory.com/28)
* [https://immunefi.com/](https://immunefi.com/)

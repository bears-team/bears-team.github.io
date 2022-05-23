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
그럼 Smart Contract는 무엇인가? 개인적으로는 Smart Contract(스마트 컨트렉트)는 블록체인(이더리움)위에서 돌아가는 하나의 기능(Function) 또는 코드 조각(Code Snippet)이라고 생각한다. 응용프로그램(Application)이라고 설명하고 있는데도 있지만, 개인적으로는 여러개의 스마트 컨트렉트를 구현해서 하나로 통합한 것을 DApp(Decentralized Application)이라고 개인적으로 이해하고 있다. [Immunefi](https://immunefi.com/)에 등록된 다양한 프로젝트를 살펴보면 거의 대부분 프로젝트들이 다수의 스마트 컨트렉트를 활용해서 구현하고 있는 것을 알 수 있다. 앞에서 언급했던 것과 같이 Smart Contract는 EVM위에서 동작하게 되며, EVM은 Non Von Neumann Virtual Machine이다. EVM이 Non Von Neumann 머신인 것은 블록체인에서의 핵심 속성인 실행중에 코드를 변경할 수 없기 때문이다. 즉 RAM이 없고 오직 ROM만 존재한다. 따라서 이와 관련된 문제가 있을 수 있는데, 예를 들면 스마트 컨트렉트 업데이트가 필요할 때, 그리고 스마트 컨트렉트의 보안성을 강화하기 위한 난독화를 실시할 때 어려움이 있다. 

## Accounts
이더리움 블록체인에서 계정(Accounts)는 이더(ETHER)를 저장할 수 있는 공간을 의미합니다. 이더리움 블록체인을 이용하는 사용자는 계정(Accounts)을 생성할 수 있고, 이더를 보내거나 받거나 할 수 있습니다. 이렇게 적고보니 계정이라고 번역하는 것 보다 계좌라고 번역하는게 더 가까워 보입니다. 글을 읽으시는 분들이 본인이 더 적절하다고 생각하는 표현으로 사용하셔도 괜찮을 것 같습니다. 이더리움에서 Accounts는 두가지 종류가 있습니다. EOA(Externally Owned Account)라고 해서 일반적으로 사용자가 계정을 만들게 되면 EOA입니다. 그리고 당연하게 스마트 컨트넥트도 Account가 될 수 있습니다. 스마트 컨트렉트로 고유 이더리움 주소를 가질 수 있으며, 이 것은 다양한 DeFi 코드에서 확인이 가능합니다. 따라서 EOA와 EOA, EOA와 스마트 컨트렉트, 스마트 컨트렉트와 스마트 컨트렉드간에 이더의 송수신이 가능합니다.

그럼 이 두 계정의 차이점은 무엇일까요? 이더리움 공식 문서에서는 다음과 같이 이 두 계정의 차이점을 설명하고 있습니다.

### EOA(Externally Owned Account)

* Creating an account costs nothing
* Can initiate transactions
* Transactions between EOAs can only be ETH/token transfer

### Smart Contract

* Creating a contract has a cost because you're using network storage
* Can only send transactions in reponse to receiving a transaction
* Transactions from an EOA to a contract account can trigger code which can execute many differenct actions, such as transfering tokens or even creating a new contract



## Blocks
## Transcations

# Conclusion

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

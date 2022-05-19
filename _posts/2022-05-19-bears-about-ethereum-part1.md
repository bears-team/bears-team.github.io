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

블록체인을 DB관점에서 바라본다면, 블록(Block)은 트렌젝션 데이터와 현재 블록의 상태를 저장하는 곳이라고 보면될 것 같습니다. 하나의 블록안에는 여러개의 트렌젝션을 담을 수 있고, 이더리움의 높은 가스비를 해결하는 방법으로 접근하고 있습니다.

## Ethereum
## Account
## Blocks
## Transcations

# Conclusion

# References
* [https://andersbrownworth.com/blockchain/](https://andersbrownworth.com/blockchain/)
* [https://ethereum.org/en/developers/docs/blocks/#:~:text=Blocks%20are%20batches%20of%20transactions,derived%20from%20the%20block%20data.](https://ethereum.org/en/developers/docs/blocks/#:~:text=Blocks%20are%20batches%20of%20transactions,derived%20from%20the%20block%20data.)
* [https://ethereum.org/en/developers/docs/transactions/](https://ethereum.org/en/developers/docs/transactions/)
* [https://medium.com/@eiki1212/ethereum-block-structure-explained-1893bb226bd6](https://medium.com/@eiki1212/ethereum-block-structure-explained-1893bb226bd6)
* [https://medium.com/remix-ide/the-anatomy-of-a-transaction-receipt-d935aacc9fcd](https://medium.com/remix-ide/the-anatomy-of-a-transaction-receipt-d935aacc9fcd)
* [https://minstar0410.tistory.com/28](https://minstar0410.tistory.com/28)

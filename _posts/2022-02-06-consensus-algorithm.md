---
title: "합의 알고리즘(Consensus algorithm)"
author: Grizzly
categories:
  - Commons
tags:
  - ConsensusAlgorithm

sitemap :
changefreq : daily
priority : 1.0
# table of contents
toc: true # 오른쪽 부분에 목차를 자동 생성해준다.
toc_label: "table of content" # toc 이름 설정
toc_icon: "bars" # 아이콘 설정
toc_sticky: true # 마우스 스크롤과 함께 내려갈 것인지 설정
---

## Overview

이번 포스팅은 블록체인을 처음 공부하기 시작할 때, 주요 관심사는 스마트 컨트랙트 보안이지만, 그래도 합의 알고리즘은 알아야겠지?하는 생각에서 정리해놓았던 것이다. 다소 두서없이 정리되어있는 점 무한한 양해바란다.
(예전에 공부할 때 정리해놨던 것들은 대부분 이런식이네 ;P)

합의 알고리즘은 블록체인 기반 네트워크가 안전하고 신뢰할 수 있게 동작하도록하는 핵심이다. 블록체인은 누구나 아는 것처럼 P2P 분산네트워크이며, 누구나 네트워크에 참여할 수 있는 만큼, 내가 어떤 `사실`을 주장하면, 이에 대한 신뢰성을 부여할 수 있어야 한다. 그렇지 않으면, 네트워크에는 거짓된 정보로 넘쳐날 것이고, 결과적으로 안전하게 운영될 수 없을 것이다. 이런 관점에서 먼저, 분산 네트워크에서 발생할 수 있는 문제부터 살펴보자.
(참고로, 합의 알고리즘은 내 주 관심사와 다소 동떨어져있기 때문에 이 포스팅에서 현존하는 모든 합의 알고리즘들을 다루지도 않을 거다.)

## Byzantine Generals Problem (비잔틴 장군 문제)

- 분산 (컴퓨팅) 환경에서 발생할 수 있는 시스템 장애를 표현한 문제.
- 중앙집중 시스템에서는 single point failure가 발생하지 않는한, 시스템 혹은 조직의 구성원이 같은 정보를 신속하고 효율적으로 전달 받을 수 있음.
- 분산 환경에서는 구성원들이 모두 동등한 권한을 소유하기 때문에 시스템에 협조적이지 않은 참여자로 인해 시스템이 장애를 일으킬 수 있음. (이 상황에는 참여자들이 서로 떨어져있다는 것이 가정되어 있음)
- 이로 인해 시스템이 동작을 멈추거나, 다른 시스템에 잘 못된 정보를 전달하게 될 수 있음.
- 이런 상황에서 분산 시스템에 장애에 견디기 위한 threshold와 합의를 하기 위한 문제를 비유적으로 표현한 것이 Byzantine Generals Problem.

![bgp](/assets/images_post/2022-02-06-consensus-algorithm/bgp.png)

### Description

- 5명의 비잔틴 제국 장군이 도시(성)을 공격하여 함락시키려 함.
- 각 장군은 100명의 병사를 소유하고, 도시는 300명의 병사가 수비함 (병사들은 장군의 지시에 복종).
- 즉, 최소 4명의 장군이 동시에 공격해야 도시를 함락시킬 수 있음 (배신자가 전체의 1/3 이상이면 공격 실패).
- 배신자의 존재를 감안해서 어떻게 동시에 도시를 공격하기로 합의 할 수 있을까? (다수결에 의한 합의)

## 암호화폐 별 합의 알고리즘 요약

|Consensus algorithms|Cryptocurrencies|Algorithm|
|--------------------|----------------|---------|
|PoW|Bitcoin|SHA256|
|PoW|Ethereum|KECCAK256|
|PoW|Litecoin|Scrypt|
|PoW|Monero|Cryptonight|
|PoW|Zcash|Equihash|
|PoS|Waves(LPoS)|LPoS|
|PoS|Qtum|POS 3.0|
|PoS|Nxt|SHA256|
|PoS|Blackcoin|Scrypt|
|PoS|Nano|Blake2b|
|DPoS|Solana||
|DPoS|EOS|DPoS|
|DPoS|Cardano|Ouroboros|
|DPoS|TRON|DPoS|
|DPoS|Lisk|DPoS|
|DPoS|BitShares|DPoS|
|PBFT|Ripple||
|PBFT|Stellar||
|PBFT|Zilliqa|KECCAK|
|PoC|Burst|Shabal256|
|DAG|IOTA|Curl-P|
|DAG|Byteball|DAG|
|DAG|Travelflex|DAG|
|PoA(Hybrid PoW/PoS)|Dash|X11|
|PoA(Hybrid PoW/PoS)|Decred|BLAKE256|
|PoA(Hybrid PoW/PoS)|Komodo|Equihash|
|PoA(Hybrid PoW/PoS)|Peercoin|SHA256|
|PoA(Hybrid PoW/PoS)|Espers|HMQ1725|
|dBFT|NEO|RIPEMD160|
|PoI|NEM(XEM)|Ed25519|
|PoB|Slimcoin|Dcrypt|

## Consensus algorithms

- 블록체인은 분산 시스템이기 때문에 비잔틴 장군 문제와 같이 분산 시스템이 갖고 있는 문제가 발생할 수 있음.
- 합의 알고리즘은 이러한 문제를 해결하기 위한 수단일 뿐, 단순히 블록체인만을 위해 사용되는 것은 아님.

### 작업 증명 (Proof of Work, PoW)

- 많은 연산(작업)을 수행해서 조건(블록 난이도)를 만족하는 해시 값을 생성 (느림).
- 생성된 해시 값은 많은 피어 혹은 노드에 해의 검증(증명)됨 (빠름).
- 작업에 많은 전력이 소모됨.
  - 블록 생성을 위해 조직화된 채굴장이 운영되며, 전세계적으로 채굴을 위해 사용되는 전력은 소규모 국가의 연간 전력 사용량을 초과함.
  - 연산 능력이 열등한 개인은 블록 생성의 기회가 거의 없음.
- 확률적 안전성 또한 문제가 될 수 있음.
  - 비트코인의 경우, 6개 이상의 블록이 연결된 brach가 global main branch가 되고, 이 brach에 연결된 블록에 기록된 트랜잭션들만 인정됨.
  - 따라서, 어떤 트랜잭션이 발생했다는 사실이 인정되려면, 6개의 블록이 연결되길 기다려야함.
  - 비트코인 블록체인에서 블록은 평균적으로 10분마다 생성되기 때문에 6개의 블록이 생성되려면 1시간이 소요됨.
  - 즉, 어떤 트랜잭션이 인정되려면, 1시간을 대기해야 함.

![pow1](/assets/images_post/2022-02-06-consensus-algorithm/pow1.png)

- 공격자가 전체 노드 대비 51% 이상 해시 생성 능력을 갖게된다면, 블록을 조작할 수 있게 됨.
- 비트코인의 블록 해시 생성 과정.

![pow2](/assets/images_post/2022-02-06-consensus-algorithm/pow2.png)

### 지분 증명 (Proof of Stake, PoS)

- 지분(코인)을 더 많이 소유한 노드가 더 많은 트랜잭션을 기록(검증)하거나 블록을 생성할 권한을 갖음.
- 어떤 노드가 검증자(validator)가 되길 원한다면, 블록체인 네트워크 상에 staking 프로세스를 통해 자신의 지분(코인)을 커밋 (validator라는 용어도 블록체인마다 다름).
- 지분을 더 많이 staking한 노드가 PoS 프로토콜에 의해 validator로 선출되어 트랜잭션을 검증함(이에 대해 보상을 받음).
- 이때, 네트워크에 참여한 어떠한 노드라도 validator가 될 수 있음.

![pos1](/assets/images_post/2022-02-06-consensus-algorithm/pos1.png)

- 가장 지분을 많이 가진 특정 노드만 validator로 선출되는 것을 방지하기 위해 다양한 방식이 제시됨.
  - 이에 대한 안정장치가 없으면, 가장 많은 지분을 소유한 노드만 블록 생성 수수료를 독식하게 됨.
  - randomized selection
  - coin age selection
- PoS 블록체인을 공격하기 위해 PoW에서와 유사하게 51% attack을 해야하는데, PoW와 달리 전체 연산량의 51%가 아닌, 전체 지분의 51% 를 소유해야 함.
- 연산이 아닌 voting에 의해 블록이 생성되기 때문에 PoW에 비해 TPS(Transactions per Second)이 높음.
- Nothing at Stake 문제
  - fork가 발생했을 때, 어떤 노드(validator)가 양쪽 brach의 block에 모두 투표(검증)하는 상황.
  - validator 입장에서 양쪽에 모두 서명해도 손해가 없음.
  - 모든 validator가 이런 식으로 투표를 한다면, fork가 관리되지 않음.
  - 공격자가 이런 행위를 하면, 예치금(stake)를 많이 사용해서 손해를 입힘.

![pos2](/assets/images_post/2022-02-06-consensus-algorithm/pos2.png)

### 위임지분증명 (Delegated Proof of Stake, DPoS)

- PoS와 유사하게 지분에 비례해 노드가 블록을 생성하거나 트랜잭션을 검증.
- 지분을 일정량 이상 소유하면 모두가 validator가 될 수 있는 PoS와 달리, DPoS에서는 노드들 사이의 투표로 권한을 위임받은 상위노드를 선택함.
- 일부 상위 노드만 블록을 생성하거나 트랜잭션을 검증할 권한을 받음. (Steem은 20개, EOS는 21개의 상위 노드를 선출)
- PoS는 PoW에 비해 월등히 높은 TPS를 보이지만, DPoS는 소수 노드의 합의에 의해서만 블록이 생성되기 때문에 PoS보다 성능이 높음.
- 신뢰성 높은 상위노드를 선출하는 것이 중요함.

![dpos](/assets/images_post/2022-02-06-consensus-algorithm/dpos.png)

### 그밖의 합의 알고리즘
- 트레이딩 증명 (Proof of Trading, PoT)
- 소각 증명 (Proof of Burn, ProB)
- 두뇌 증명 (Proof of Brain, PoB)
- 중요도 증명 (Proof of Importance, PoI)
- 신뢰성 증명 (Proof of Believability, PoB)
- 흐름 증명 (Proof of Flow, PoF)
- 권위 증명 (Proof of Authority, PoA)
- 저장 증명 (Proof of Storage, PoS)
- 용량 증명 (Proof of Capacity, PoC)
- 경과시간 증명 (Proof of Elapsed Time, PoET)
- 비잔틴 장애 허용(Byzantine Fault Tolerance, BFT)

## 마치며

합의 알고리즘은 블록체인이 동작할 수 있도록 해주는 가장 중요한 개념이라고 할 수 있겠다. 혹시 블록체인 공부를 처음 시작하는 사람들에게 도움이 되었길 바란다. 역시 예전에 공부할 때 정리한 것을 그대로 복사/붙여넣기하다보니 글이 너무 정신 없다. 기회가 되면, 정리하지 못한 합의 알고리즘도 추가하고, 조금 더 정리해보겠다. 

## References
- Leslie Lamport, Robert Shstak, Marshall Pease, "The Byzantine Generals Problem," ACM Transaction on Programming Language and Systems, vol. 4, iss. 3, 1982.
- Seyed et al., "A Survey of blockchain consensus algorithms performance evaluation criteria," 2020.
- [https://brunch.co.kr/@banksalad/313](https://brunch.co.kr/@banksalad/313)
- [https://itwiki.kr/w/Nothing_at_Stake](https://itwiki.kr/w/Nothing_at_Stake)
- [https://www.steemcoinpan.com/hive-101145/@ayogom/dpos-dpos](https://www.steemcoinpan.com/hive-101145/@ayogom/dpos-dpos)
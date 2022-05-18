---
title:  "Polygon Consensus Bypass Bugfix 리뷰"
excerpt: "Polygon Consensus Bypass Bufix Review 문서 학습 목적으로 살펴본 내용임"

author: Panda
categories:
  - Security
  - Smart Contract
  - Polygon
tags:
  - Review
  - Logical Bug
  - Critical Severity
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2022-03-16 9:00:00 +09:00
lastmod: 2022-03-16 9:00:00 +09:00
sitemap :
changefreq : daily
priority : 1.0

# table of contents
toc: true # 오른쪽 부분에 목차를 자동 생성해준다.
toc_label: "table of content" # toc 이름 설정
toc_icon: "bars" # 아이콘 설정
toc_sticky: true # 마우스 스크롤과 함께 내려갈 것인지 설정

#{% figure caption:"Le logo de **Jekyll** et son clin d'oeil à Robert Louis Stevenson"
#  %}
#  ![Le logo de Jekyll]({{"/assets/images_posts/2021-09-23-cve-2021-31956-part1/1.png"| #relative_url}})
#{% endfigure %}
---
# Executive Summary
이 문서는 Immunefi사의 기술 블로그중 [Polygon Consensus Bypass Bugfix Review](https://medium.com/immunefi/polygon-consensus-bypass-bugfix-review-7076ce5047fe) 문서를 읽고 BEARS 팀내 기술 세미나 자료로 활용하기 위해 작성되었으며, 해당 블로그 내용 공유를 통해 Smart Contract내 취약점 유형을 학습하는데 중점을 두고 작성된 문서입니다.

# Introduction
1월15일 화이트해커 [Niv Yehezkel](https://www.linkedin.com/in/niv-y-b0618551)이 합의 알고리즘을 우회할 수 있는 최약점을 PoC와 함께 제보함. 해당 취약점은 공격자가 악성행위를 하기 위해 필요한 PoS 합의 알고리즘 내에서 확보해야할 Staking Power(지분)를 줄여주는 효과가 있다. 해당 취약점을 통해 무제한적인 인출이 가능하다고 immunefi에서는 설명하고 있으나, 해당 취약점을 활용하기 위한 조건이 좀 까다롭기 때문에 위험도 판정에서 High를 받았고, 보상 또한 $75,000 한화 약 8천만원 정도 된다.  
해당 취약점을 유발(Trigger)하기 위해 필요한 조건으로 아래와 같은 것들을 요구하며, 간단히 말하면 해당 조건을 유지하기 위해 상당한 돈이 필요하다.
* Validator 자리가 비어져 있어야 한다. 그리고 해당 자리를 가지고 유지하기 위해 
* flashbot을 사용해서 채굴자(miner)가 Validator 자리를 계속 차지하게 할려면 상당한 돈을 지불해야 한다.
* 또한 폴리곤의 체크포인트는 매 30-45분마다 발생하며, 공격에 필요한 시간에 비례하여, 비용이 발생하게 된다.

위와 같은 이유로 해당 취약점은 활용 가능성이 떨어지나, 실제 공격 가능성 보다 취약점 유형관점에 해당 리뷰를 보면 의미가 있다고 생각합니다. 취약점을 유발하기 위해, deposit, withdraw 함수만 주로 분석을 하게 되는데, 위 함수와 다른 기능으로 구현된 함수라도 주요 변수(예: amount)에 영향을 줄 수 있다면, 그 것을 활용하여 취약점을 유발할 수 있다는 것을 본 취약점 사례로 확인할 수 있습니다.
취약점을 찾는 과정에서 deposit, withdraw 기능을 구현한 함수 뿐만 아니라, 핵심 변수를 접근하는 다양한 기능 함수 또한 함께 분석함으로써 취약점을 찾을 가능성을 높일 수 있다라고 생각됩니다. 

# Background
## Validator
Validator는 폴리곤 블록체인내에서 합의 그룹 업무에 참여하고 이더리움 메인넷에 체크포인트(checkpoints)를 커밋(commit)하는 노드(유닛)을 의미한다. Validator는 트랜젝션을 검증하고, 블록체인에 새로운 블록을 추가하고, 보상을 벌 수 있다.
## Delegator
검증 노트에 토큰을 스테이킹 함으로써 폴리곤 네트워크의 보안성 강화에 기여할 수 있으며, 스테이킹을 통한 보상을 얻을 수 있다.

## 검증자(Validator)는 어떻게 하면 될 수 있나?
위 질문에 대한 간략한 대답은 폴리곤 사이트에서 짧게 아래와 같이 기술하고 있다.  
만약에 Validator 빈 자리가 있다면, 누구나 상당한 양의 스테이킹을 가지고 검증자(validator)가 될 수 있다. 매일 정기적으로 검증자(validator) 선발 경매가 진행되는데, 거기서 경쟁자보다 많은 토큰 스테이깅을 제안함으로써 기존 검증자(validator)를 제치고 그 자리를 차지할 수 있다. 즉 검증자(validator) 자리를 예약하거나 하는 것은 안되며, 누구나 스테이킹양을 가지고 경쟁할 수 있는 열린 구조이다.

## Intro to the Consensus Mechansim
합의 알고리즘의 개념을 설명하고 있는, BEARS 블로그내 [합의 알고리즘 간략 정리](https://bears-team.github.io/commons/consensus-algorithm/)를 참고하면 도움이 될 것 같습니다.
합의 알고리즘(메커니즘)이란 어떤 진실(사실)에 대하여 해당 네트워크에서 동의하는 방법을 의미하는데, 중앙화된 시스템의 경우 해당 네트워크를 지배하는 주체가 진실여부를 판단하면 되지만, 분산화된 네트워크 구조에서 익명성을 지닌 각각의 자동화된 권위 주체에 의해 진위여부가 결정된다. 예를 들어 Proof of Work(PoW)의 경우 해당 네트워크 노드의 적어도 51%의 합의가 필요하다. 이러한 합의 메커니즘은 토큰 탈취와 같은 경제범죄의 예방효과도 가지고 있는데, 악성행위자가 합의구조를 악용하기 위해서는 네트워크의 51%를 제어가능해야 하기 때문이다. 따라서 합의구조를 기반으로 하는 공격의 불가능한데, PoW의 경우는 51%의 노드를 장악해야하지만, PoS 합의 알고리즘에서는 네트워크 메인넷 토큰 수량의 3분의2에 1을 더한 수량이 필요하다.  
다시 폴리곤으로 돌아와서, 폴리곤의 경우 이더리움위에서 돌아가는 네트워크인데, 레이어1인 이더리움의 경우 지금 현재는 PoW 구조이며, 폴리곤의 경우 이더리움의 확장성 높은 가스비를 해결하기위한 레이어2 확장 솔류션으로 합의 메커니즘은 PoS이다. 폴리곤에는 네트워크 안전성 확보를 위해 토큰 스테이킹을 기반으로하는 검증자(validator) 그룹이 존재한다. 폴리곤 네트워크에서 검증자 그룹의 역할은 다음과 같다.
* Run a full node : 블록체인 네트워크 운영
* Produce blocks : 볼록생성
* Validate and participate in concensus : 검증 및 합의알고리즘 참여
* Commit checkpoints on the Ethereum : 이더리움 네트워크에 체크포인트 남기기(폴리곤이 이더리움기반 레이어2 솔류션인 관계로...)
검증자로서 참여하면 스테이킹 토큰 양에따라 보상을 얻을 수 있다.  
검증자외 폴리곤 네트워크의 안정성에 참여할 수 있는 방법으로는 위임자(Delegator)가 있다. 위임자는 어려운 개념이 아니라 자기가 가지고 있는 토큰을 검증자(Validator)에 위임한 것을 뜻하며, 자기가 원하는 양만큼 맡길 수 있다. 위임자 또한 보상을 획득할 수 있는데, 보상의 크기는 위임한 토큰 수량에 비례한다. 첨언하여 보다 정확하게 설명하면 검증자 지분을 사는 것이며, DeFi(디파이)에서 유동성을 공급함으로써 ibToken을 지급받는 것과 동일한 개념이다.([buyshare](https://github.com/maticnetwork/contracts/blob/e7505c926d871f7ff2191691808f2c4661516366/contracts/staking/validatorShare/ValidatorShare.sol#L370)함수 참조)

# Vulnerability Analysis
폴리곤의 검증자 관리와 관련해서 [스테이킹 관리 컨트렉트](https://github.com/maticnetwork/contracts/blob/e7505c926d871f7ff2191691808f2c4661516366/contracts/staking/stakeManager/StakeManager.sol)을 살펴보면 되고, 해당 컨트렉트의 경우 아래와 같은 기능을 수행한다.
* checkpoint signature verification
* reward distribution
* slashing validators
* stake management

검증자(validator)로서 네트워크에 참여하는 것은 [stakeFor](https://github.com/maticnetwork/contracts/blob/e7505c926d871f7ff2191691808f2c4661516366/contracts/staking/stakeManager/StakeManager.sol#L446)함수에 의해서 이뤄지며, 내부적으로는 [\_stakeFor](https://github.com/maticnetwork/contracts/blob/e7505c926d871f7ff2191691808f2c4661516366/contracts/staking/stakeManager/StakeManager.sol#L1056) 함수가 호출된다. 이 컨트렉트에 토큰을 스테이킹함으로써 누구나 폴리곤 네트워크에 참여할 수 있으며, 네트워크 참여의 증거로 아래 그림.1에서와 같이 ValidatorId에 해당하는 NFT를 발급받게 된다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-03-10-immunefi-polygon-consensus-review/stakeFor.png"| relative_url}})  |
|:--:| 
| 그림.1 \_stakeFor 함수내 ValidatorId 생성부분 |

그럼 원하는 사람 모두가 검증자로 참여 가능한가?라는 질문에 대답은 검증자 참여는 앞에서도 설명한 것 과같이 제한이 있으며 현재 폴리곤의 경우 100개로 제한하고 있다. 하지만 자리가 날 경우 토큰양을 기반으로 많이 가진 참여자가 검증자 자리를 차지할 수 있다. 참여 슬롯을 의미하는 변수는 validatorThreshold가 존재한다.

해당 검증자가 위임자를 받고 싶으면 acceptDelegation bool 파라미터를 셋팅하면 되는데, 이 때 [validatorShare](https://github.com/maticnetwork/contracts/blob/e7505c926d/contracts/staking/validatorShare/ValidatorShare.sol) 컨트렉트를 생성하는 것을 그림.1에서 확인할 수 있다.  

[validatorShare](https://github.com/maticnetwork/contracts/blob/e7505c926d/contracts/staking/validatorShare/ValidatorShare.sol) 컨트렉트는 위임자 참여와 관련된 보유하고 있으며, 그 기능은 아래와 같다.
* buying and selling shares of the validators
* caculating the rewards earned

위임자는 폴리곤 네트워크에 [stakeManager](https://github.com/maticnetwork/contracts/blob/e7505c926d871f7ff2191691808f2c4661516366/contracts/staking/stakeManager/StakeManager.sol) 컨트렉트에 토큰(폴리곤에서는 MATIC)을 스테이킹 함으로써 참여할 수 있다. 이때 스테이킹 양에 따른 검증자의 지분을 받게 되며, 일정 시간이 지난 후 지분을 매각함으로써 원래 맡긴 토큰과 보상을 받게 된다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-03-10-immunefi-polygon-consensus-review/buyVoucher.png"| relative_url}})  |
|:--:| 
| 그림.2 buyVoucher 함수 |

그림.2 buyVoucher함수에서 내부적으로 [\_buyShares](https://github.com/maticnetwork/contracts/blob/e7505c926d871f7ff2191691808f2c4661516366/contracts/staking/validatorShare/ValidatorShare.sol#L370)함수를 호출하며, 그림.3에서와 같이 결국 스테이킹 토큰양에 비례하여 지분을 받게 된다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-03-10-immunefi-polygon-consensus-review/buyShare.png"| relative_url}})  |
|:--:| 
| 그림.3 \_buyShare 함수 |

스테이킹되는 토큰 및 스테이커의 추가되거나 제거될 때 매번 stateManager 컨트렉트에 있는 [updateTimeline(amount, stakerCount, epoch)](https://github.com/maticnetwork/contracts/blob/e7505c926d871f7ff2191691808f2c4661516366/contracts/staking/stakeManager/StakeManager.sol#L740)가 호출되며, 이 함수의 주요 기능한 전체 스테이킹된 토큰의 양과 스테이커의 개수를 계산하는 것이다.  

StakingManager 컨트렉트의 경우 검증자(validator) 상태에 대한 정보를 유지하기 위해서 "validateState"라는 구조체를 이용한다. 해당 구조체는 멤버로 아래와 같은 변수를 가지고, 이 변수는 본 취약점에서 중요한 요소이다.
* validatorState.amount : 해당 검증자(validator)에 의해 스테이킹된 토큰의 전체 양을 의미하며, 이는 곧 해당 검증자의 파워를 의미한다.
* validatorState.stakerCount : 해당 컨트렉트에 스테이킹한 스테이커의 전체 개수를 의미한다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-03-10-immunefi-polygon-consensus-review/updateTimeline.png"| relative_url}})  |
|:--:| 
| 그림.4 updateTimeline 함수 |

만약 검증자(validator)가 언스테이킹을 하게 되면, 전체 스테이킹 파워는 위임된 토큰의 양과 검증자 토큰 양이 함께 업데이트 된다. 

```javascript
updateTimeline(-(int256(amount) + delegationAmount), -1, targetEpoch);
```

이를 \_unstake 함수에서 확인할 수 있다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-03-10-immunefi-polygon-consensus-review/unstake.png"| relative_url}})  |
|:--:| 
| 그림.5 \_unstake 함수 |

StakeManager 컨트렉트의 경우 [migrateDelegation](https://github.com/maticnetwork/contracts/blob/e7505c926d871f7ff2191691808f2c4661516366/contracts/staking/stakeManager/StakeManager.sol#L521) 기능을 가지고 있는데, 해당 함수를 통해 위임자가 위임한 토큰을 다른 검증자에게 옮길 수 있다. 아래 그림을 보면 migrateDelegation 함수코드가 보이는데, validator\[fromValidatorId\].contractAddress의 [migrateOut](https://github.com/maticnetwork/contracts/blob/e7505c926d871f7ff2191691808f2c4661516366/contracts/staking/validatorShare/ValidatorShare.sol#L166) 함수를 호출하여 토큰을 인출하는데, 이때 검증자로 부터 받을 리워드를 포함한 스테이킹 토큰이 인출되며, 위임자가 보유하고 있는 지분은 소각처리 된다. 이후 검증자의 상태는 업데이트 된다. [migrateIn](https://github.com/maticnetwork/contracts/blob/e7505c926d871f7ff2191691808f2c4661516366/contracts/staking/validatorShare/ValidatorShare.sol#L183)함수의 경우 buyVoucher 함수의 동작(\_buyShares 호출)과 같다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-03-10-immunefi-polygon-consensus-review/migrateOut.png"| relative_url}})  |
|:--:| 
| 그림.6 migrateOut 함수와 migrateIn 함수 |

이 번에 제보된 폴리곤 취약점의 경우 위임자가 위임된 토큰을 다른 검증자로 옮기는 과정에서 취약점이 발생하게 된다. migrateOut 함수를 보면 updateValidatorState함수 호출을 통해 updateTimeline(-amount)가 호출이 될 것이며, 이를 통해 stakeManger 컨트렉트에서 전체 검증자 파워이 줄어들 것이다. 이 때 어떤 검증자가 언스테이킹을 한다면, 전체 스테이킹 파워는 검증자의 토큰양 및 위임자의 토큰 양만큼 줄어들게 된다. 
* 검증자가 언스테이킹 한다면, 위임자는 자신의 토큰을 이동시킬 수 있다.
* 위임자가 자신의 토큰을 이전할 때, 전체 스테이킹 파워가 줄어 들게되며
* 검증가가 언스테이킹 될 때, 전체 스테이킹 파워에서 검증자 토큰 및 위임자 토큰이 전체 스트테이킹 파워가 빠지게 된다.

이 것을 코드 수준에서 다시 한 번 더 살펴보면 다음과 같다.
위임자가 migrateOut함수를 호출하면 그림.6에서와 같이 내부적으로 stakeManager 컨트렉트의 updateValidateState(validatorId, -int256(amount)) 함수가 호출되며, 이는 updateTimeline(amount, 0, 0) 함수를 통해 전체 스테이킹 파워를 해당 amount만큼 줄이게 된다. 이 때 좀더 해당 검증자의 상태 스테이킹 중인지 여부만 확인하는 코드가 있다면 위 버그를 막을 수 있다. 검증자가 언스테이킹 상태일 경우는 위 동작을 하지 않도록 하면 될 것이다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-03-10-immunefi-polygon-consensus-review/updateValidatorState.png"| relative_url}})  |
|:--:| 
| 그림.7 updateValidatorState 함수 |

# Exploitation
* stakeFor 함수을 이용하여 새로운 검증자(validator)를 생성한다.
* buyVoucher 함수를 호출하여 엄청난 양의 지분을 확보한다.
* 위 과정을 전체 지분의 2/3 합의를 통과할 정도로 반복한다.
* 검증자 자리가 비면 해당 검증자 자리를 확보한다.
* migrateDelegation 함수를 사용해 토큰을 해당 신규 검증자로 옮긴다.
* 토큰을 옮긴 검증자를 언스테이킹 한다. 전체 스테이킹 파워가 줄어듦
* 체크 포인트를 기다린다. 빈 검증자 자리를 차지한다.
* 위 작업을 2/3 합의 알고리즘을 충분히 무력화 시킬때까지 지속적으로 반복한다. 단 하나의 검증자로도 합의 알고리즘을 통과시킬 정도로까지 반복 가능

| ![Image Alt 텍스트]({{"/assets/images_post/2022-03-10-immunefi-polygon-consensus-review/MajorityCheck.png"| relative_url}})  |
|:--:| 
| 그림.8 다수성을 확인 |

합의 알고리즘을 무력화 했기 때문에, 예치된 토큰을 전체 인출한다라는 것과 같은 거짓된 트렌젝션을 합의함으로써 경제적 이득을 가져갈 수 있다.

# Vulnerability Fix & Lesson

버그의 수정은 updateValidatorState함수내에 해당 검증자가 살아있는 검증자인지를 확인하는 코드가 추가되었다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-03-10-immunefi-polygon-consensus-review/updateValidatorStateFixed.png"| relative_url}})  |
|:--:| 
| 그림.9 수정된 updateValidatorState함수 |


# References
* [https://medium.com/immunefi/polygon-consensus-bypass-bugfix-review-7076ce5047fe](https://medium.com/immunefi/polygon-consensus-bypass-bugfix-review-7076ce5047fe)
* [https://bears-team.github.io/security/smart%20contract/immunefi-polygon-review/](https://bears-team.github.io/security/smart%20contract/immunefi-polygon-review/)
* [https://polygon.technology/staking/](https://polygon.technology/staking/)
* [https://github.com/maticnetwork/contracts/blob/e7505c926d871f7ff2191691808f2c4661516366/contracts/staking/stakeManager/StakeManager.sol](https://github.com/maticnetwork/contracts/blob/e7505c926d871f7ff2191691808f2c4661516366/contracts/staking/stakeManager/StakeManager.sol)
---
title:  "Polygon Lack Of Balance Check Bug fix Postmortem $2.2m Bounty 리뷰"
excerpt: "Immunefi Polygon Lack Of Balance Check Bugfix Postmortem-$2.2m Bounty 문서 학습 목적으로 살펴본 내용임"

author: Panda
categories:
  - Security
  - Smart Contract
tags:
  - Ploygon
  - Logical Bug
  - Critical Severity
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2022-1-26 07:00:00 +09:00
lastmod: 2022-1-29 11:00:00 +09:00
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
이 문서는 Immunefi사의 기술 블로그중 [Ploygon Lack Of Balance Check Bug fix Postmortem-$2.2m Bounty](https://medium.com/immunefi/polygon-lack-of-balance-check-bugfix-postmortem-2-2m-bounty-64ec66c24c7d) 문서를 읽고 BEARS 팀내 기술 세미나 자료로 활용하기 위해 작성되었으며, 해당 블로그 내용 공유를 통해 Smart Contract내 취약점 유형을 학습하는데 중점을 두고 작성된 문서입니다.

# Background
## Polygon project
* 2017년도에 Matic 프로젝트라는 이름으로 프로젝트 시작
* 2021년 2월에 Ploygon으로 리브랜딩
* Proof of Stake (PoS) 
* 인도 사람들이 주축(인도에 본사 위치)
* 이더리운 호환 블록체인 네트워크 구축을 목표
* 블록체인 네트워크의 거래 처리 속도를 개선하고 비용을 줄이는 것으로 목표로함(이더리움과의 연결성을 중시)

| ![Image Alt 텍스트]({{"/assets/images_post/2022-01-26-immunefi-polygon/polygon_project.png"| relative_url}})  |
|:--:| 
| 그림.1 Polygon 프로젝트의 목표 |

## Ploygon Address(0x0)
* 특별한 주소이며, 폴리곤 프로젝트에서 토큰을 소각하거나 생성할 때 사용하는 주소
* 아래 그림을 보면 지금 현재에도 관점에 따라 상당하다고 생각할 수 있는 약 3.5만개의 MATIC(폴리콘 토큰 단위)이 존재하며, 취약점 제보시 9,276,584,322 MATIC이 존재했던 것으로 Immunefi 원문 포스트에 기술되어 있습니다.


| ![Image Alt 텍스트]({{"/assets/images_post/2022-01-26-immunefi-polygon/polyscan_address_0.png"| relative_url}}) | 
|:--:| 
| 그림.2 Ployscan에서 확인한 0x0 주소 |

# Introduction
Immunefi에서 공개한 글을 분석해보면, 취약점은 매우 단순하고 Exploit 난이도가 낮은 취약점이라 그 효과 측면에서 매우 활용성이 높은 취약점으로 판단할 수 있습니다. 그래서 취약점을 이해하는데 큰 어려움이 없을 것으로 생각이 됩니다. 해당 취약점은 개발자가 폴리곤(Polygon) 프로젝트에 대한 이해가 부족한 상태에서 다른 프로젝트의 코드를 재활용하는 과정에서 발생한 것으로 개인적으로 추정하고 있습니다. 해당 취약점과 관련된 주요 이벤트를 시간적으로 정리하면 아래와 같이 됩니다.

* 2022.12.3 : Leon Spacewalker 취약점 제보 \rightarrow 하드포크할 정도로 심각한 취약점
* 2022.12.4 : 익명의 화이트해커가 동일한 취약점 제보 \rightarrow 500,000 MATIC 지급
* 2022.12.4~5 : 해당 취약점을 활용한 801,601MATIC 탈취함
* 2022.12.5 : 취약점 패치되었으나, 최악의 경우 폴리곤(Polygon) 프로젝트가 종료될 수 도 있는 9,276,584,332 MATIC의 손실을 가져올 수 도 있는 취약점이였음
* 취약점 원인을 보면 단순 코딩 실수라기 보다, 해당 프로젝트의 개발자들이 자기 자신이 속한 프로젝트에 대한 이해가 떨어진 상태에서 외부 코드를 참조해서 구현했기 때문에 발생한 것으로 의심되는 좋은 사례라고 생각이 들고, 이와 같은 경우가 찾아보면 꽤 있을 것으로 판단됨 

# Vulnerability
폴리곤(Polygon) 네트워크에서 이더리운 ERC20와 같이 MRC20 표준이 존재하며, 해당 표준에 가스비 없이 전송할 수 있는 것을 규정하고 있다. 상식선에서 폴리곤 네트워크에서 가스비가 존재한다면 이더리움을 사용하지 폴리곤을 사용할 일이 없고, 이더리움의 높은 가스비 문제를 해결하겠다고 하며, 이더리움 네트워크 활용 프로젝트의 경우 비슷한 표준을 가지고 있을 가능성이 높다고 생각합니다. MRC20표준내 가스비 없는 전송을 구현하기 위해 transferWithSig() 함수가 존재하며, 해당 함수의 코드는 아래 그림과 같습니다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-01-26-immunefi-polygon/transferWithSig.png"| relative_url}}) | 
|:--:| 
| 그림.3 transferWithSig 함수 코드 |

위 코드를 보면 ecrecovery 함수가 보이는데 해당 함수의 기능은 재활용 공격(Replay attack)을 방지하기 위해 서명된 해쉬값을 입력으로 받아 전송자의 주소를 리턴하는 함수이다. ecrecovery 함수의 리턴값 주소를 \_transferFrom 함수의 인자로 전달하게 된다. 취약점의 근본 원인은 ecrecovery 함수에서 발생하게 된다. 

| ![Image Alt 텍스트]({{"/assets/images_post/2022-01-26-immunefi-polygon/ecrecovery.png"| relative_url}}) | 
|:--:| 
| 그림.4 ecrecovery 함수 코드 |

위 ecrecovery 함수를 보면 서명의 길이가 65가 아니면 무조건 0x0 번지 주소를 리턴하게 되어있다. 문제는 함수 하단에서 결과값이 0x0번지일 경우 에러를 출력하게 해놨지만, 에러를 의미하는 주소를 해당 프로젝트에서 의미있는 값을 활용하는 것 차제가 조금 위험한 함수 설계가 아닌가?라는 생각이 드는건 어쩔 수 없다. 윈도우 커널 코드만 봐도 에러의 경우 리턴 값은 0xC000XXXX나 0xFF00XXXX와 같은 잘 활용되지 않는 값을 기반으로 에러코드를 설정하고 있는 것을 쉽게 확인할 수 있다.

이후 ecrecovery함수의 결과를 아래 그림에서와 같이 \_transferFrom함수에서는 아쉽게도 검증없이 그래도 사용하게 된다. ecrecovery 함수에서 0x0 주소가 리턴이 되었더라고, \_transferFrom함수에서 한 번만 더 검증이 있었더라면, 아니면 \_transferFrom함수에서 호출하는 \_transfer함수에서 라도 검증하는 루틴이 있었어야 했으면 해당 취약점을 방지할 수 있었을 텐데, 폴리곤(Poloygon) 프로젝트의 경우 그러한 운이 없었던것 같다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-01-26-immunefi-polygon/transferFrom.png"| relative_url}}) | 
|:--:| 
| 그림.5 _transferFrom 함수 코드 |

결국 일단 급한데로 폴리곤(Ploygon) 프로젝트에서는 아래 그림과 같이, 해당 함수를 제거하는 것으로 급하게 취약점을 제거한 것으로 보입니다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-01-26-immunefi-polygon/patch.png"| relative_url}}) | 
|:--:| 
| 그림.6 폴리곤 프로젝트팀에서 실시한 패치 정보 |


# Lesson
* 함수 설계시, 에러 값(상수)를 잘 정해야 한다.
* 함수 구현시, 입력값에 대한 검증 루틴 삽입은 기본중에 기본이다. 윈도우 커널의 경우 대부분의 함수에서 이제는 기본적인 취약점 Integer Over/Uderflow을 방지하기 위해 범위값 체크는 거의 다 한다.
* 모든 블록체인 프로젝트들이 구현에 쫒기다 보니, 의외로 치명적인 결과를 가지올 수 있는 Exploit이 쉬운 논리 취약점이 많을 것으로 생각되며, 비슷한 유형을 찾을 수 있는 패턴검색기반 코드 검증이 필수적이라 판단된다.

# References
* [https://medium.com/immunefi/polygon-lack-of-balance-check-bugfix-postmortem-2-2m-bounty-64ec66c24c7d](https://medium.com/immunefi/polygon-lack-of-balance-check-bugfix-postmortem-2-2m-bounty-64ec66c24c7d)
* [https://polygonscan.com/address/0x0000000000000000000000000000000000000000](https://polygonscan.com/address/0x0000000000000000000000000000000000000000)
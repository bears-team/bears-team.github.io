---
title:  "Ploygon Lack Of Balance Check Bug fix Postmortem $2.2m Bounty"
excerpt: "Immunefi Ploygon Lack Of Balance Check Bugfix Postmortem-$2.2m Bounty 문서 학습 목적으로 조사한 내용임"

author: Panda
categories:
  - Security
  - Smart Contract
tags:
  - Ploygon
  - Vulnerability
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2022-1-26 07:00:00 +09:00
lastmod: 2022-1-26 7:10:00 +09:00

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
이 문서는 Immunefi사의 기술 블로그중 Ploygon Lack Of Balance Check Bug fix Postmortem-$2.2m Bounty 문서를 읽고 BEARS 팀내 기술 세미나 자료로 활용하기 위해 작성되었으며, 해당 블로그 내용 공유를 통해 Smart Contract내 취약점 유형을 학습하는데 중점을 두고 작성된 문서임.

# Background
## Ploygon project
* 2017년도에 Matic 프로젝트라는 이름으로 프로젝트 시작
* 2021년 2월에 Ploygon으로 리브랜딩
* Proof of Stake (PoS) 
* 인도 사람들이 주축(인도에 본사 위치)
* 이더리운 호환 블록체인 네트워크 구축을 목표
* 블록체인 네트워크의 거래 처리 속도를 개선하고 비용을 줄이는 것으로 목표로함(이더리움과의 연결성을 중시)

| ![Image Alt 텍스트]({{"/assets/images_post/2022-01-26-immunefi-polygon/ploygon_project.png"| relative_url}}) | 
|:--:| 
| 그림.1 Ploygon 프로젝트의 목표 |

## Ploygon Address(0x0)
* 특별한 주소이며, 폴리곤 프로젝트에서 토큰을 소각하거나 생성할 때 사용하는 주소
* 아래 그림을 보면 관점에 따라 상당하다고 생각할 수 있는 약 3.5만개의 MATIC(폴리콘 토큰 단위)이 존재한다. 취약점 제보시 9,276,584,322 MATIC이 존재했던 것으로 Immunefi 블로그에 기술되어 있음


| ![Image Alt 텍스트]({{"/assets/images_post/2022-01-26-immunefi-polygon/ployscan_address_0.png"| relative_url}}) | 
|:--:| 
| 그림.2 Ployscan에서 확인한 0x0 주소 |

# Introduction

# Vulnerability

# References
* https://medium.com/immunefi/polygon-lack-of-balance-check-bugfix-postmortem-2-2m-bounty-64ec66c24c7d
* https://polygonscan.com/address/0x0000000000000000000000000000000000000000
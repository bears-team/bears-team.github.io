---
title:  "Bitswift Unlimited Mint Bugfix Postmortem 리뷰"
excerpt: "Immunefi Bitswift Unlimited Mint Bugfix Postmortem 문서 학습 목적으로 살펴본 내용임"

author: Panda
categories:
  - Security
  - Smart Contract
tags:
  - Web
  - Logical Bug
  - Critical Severity
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2022-02-17 19:00:00 +09:00
lastmod: 2022-02-17 19:00:00 +09:00
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
이 문서는 Immunefi사의 기술 블로그중 [Bitswift Unlimited Mint Bugfix Postmortem](https://medium.com/immunefi/bitswift-unlimited-mint-bugfix-postmortem-147a1e57dca9) 문서를 읽고 BEARS 팀내 기술 세미나 자료로 활용하기 위해 작성되었으며, 해당 블로그 내용 공유를 통해 Smart Contract내 취약점 유형을 학습하는데 중점을 두고 작성된 문서입니다.

# Background
## Bitswift(BITS)
* 캐나다 소재 블록체인 기술 기업 주관의 프로젝트(블록체인 네트워크)
* 기존 사업영역에 블록체인 기술을 제공하는 것이 주요 사업임
* 블록체인 네트워크는 2014년에 런칭됨
* BITS(Bitswift 토큰)은 Bitswift 블록체인과 Bitswift사에서 직접 제공하는 서비스 Bitswift 캐시(크립토 케이트웨이 서비스) 및 관련 제품에 활용됨
  * Bitswift Forging Pool(LZLZ)
  * Bitswift Cash : 다른 다양한 블록체인 네트워크에 접속할 수 있는 게이트웨이 서비스
  * NFTMagic ART : BITS를 지원하는 NFT 시장
  * Sigbro : Ignis, ARDR에 서명된 트랜젝션을 전송할 수 있는 모바일앱
# Introduction
* 2021.12.8.에 보안전문가 Immunefi에 버그 제보
* 제보한 버그는 BCAD 토큰을 무한히 민팅할 수 있고, 해당 토큰을 이용해 지금 liquid 풀에서 다른 토큰을 빼올 수 있음
* Bitswift사는 4,515달러를 보안전문가에게 지급하고 해당 취약점 패치
# Vulnerability

# Lesson

# References
* Bitswift 관련 사이트
  *  [https://bitswift.io](https://bitswift.io)
  *  [https://bitswift.tech](https://bitswift.tech/)
  *  [https://bitswift.cash/](https://bitswift.cash/)
  *  [https://nftmagic.art/](https://nftmagic.art/)
  *  [https://bitswift.cash/explorer/blocks](https://bitswift.cash/explorer/blocks)
  *  [https://myaccount.bitswift.network/index.html](https://myaccount.bitswift.network/index.html)
* [https://medium.com/immunefi/bitswift-unlimited-mint-bugfix-postmortem-147a1e57dca9](https://medium.com/immunefi/bitswift-unlimited-mint-bugfix-postmortem-147a1e57dca9)
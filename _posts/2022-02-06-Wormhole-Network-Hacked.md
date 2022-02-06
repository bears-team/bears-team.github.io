---
title: "Wormhole Network 해킹"
author: IceBear
categories:
  - Security 
  - Vulnerability
tags:
  - Wormhole
  - Solana 
  - Ethereum
sitemap :
changefreq : daily
priority : 1.0

# table of contents
toc: true # 오른쪽 부분에 목차를 자동 생성해준다.
toc_label: "table of content" # toc 이름 설정
toc_icon: "bars" # 아이콘 설정
toc_sticky: true # 마우스 스크롤과 함께 내려갈 것인지 설정
---

지난 2월 2일 Wormhole 네트워크에서 약 12만개의 wETH(ETH와 1:1 교환 가능)가 해킹으로 유출되는 사건이 발생했다.
![Wormhole](/assets/images_post/2022-02-06-Wormhole-hack/wormhole-twitter.png)
이는 원화로 약 3000억이 넘는 금액에 달하는 규모이다.
이 글에서는 이번 공격에 대한 여러 분석 글을 토대로 공격이 가능했던 이유를 살펴보고자 한다.

## Wormhole Network 이란

[Wormhole]이란 여러 블록체인들을 연결해주는 Bridge 서비스이다. 
현재 지원하는 체인은 Ethereum을 비롯하여 Solana, Terra, BSC, Polygon 등이 있다.
![WormholeNetwork](/assets/images_post/2022-02-06-Wormhole-hack/wormhole-network.png)

Wormhole 네트워크는 19개의 Guardian에 의해 운영되는데, Guardian은 체인간 전달되는 메시지들을 인증해주는 역할을 한다.
2/3 이상의 Guardian들이 인증을 해주면 해당 메시지는 신뢰할 수 있다고 간주한다.

## 취약점 분석



[Wormhole]: https://docs.wormholenetwork.com/wormhole/
[WormholeNetwork]: https://wormholecrypto.medium.com/introducing-wormhole-32b16d795c01
[Wormhole 트위터]: https://twitter.com/wormholecrypto/status/1489001949881978883
[Wormhole 해킹 분석]: https://wormholecrypto.medium.com/wormhole-incident-report-02-02-22-ad9b8f21eec6
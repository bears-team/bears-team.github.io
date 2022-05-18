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
  - Review
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
Guardian들에는 [Chorus One], [Staked.us], [P2P Validator] 등이 있다.

만약 Ethereum 체인에서 Solana 체인으로 토큰을 이동시키고자 한다면, Ethereum 체인에서 Wormhole 서비스 위에 있는 Portal에 
ETH를 예치시키면 이를 인증된 메시지를 통해 Solana 체인에 Wormhole wrapped 버전의 토큰(weETH)이 발급(mint)되는 방식이다. 

## 취약점 분석

공격자는 Ethereum 체인상에서 존재하지 않는 ETH 120,000개를 Wormhole을 통해 Solana 체인 상 Wormhole wrapped ETH(weETH)를 발급받았다. 그리고 이 중에 93,750개를 다시 Ethereum 체인상으로 돌려보내 ETH를 확보하게 된 것이다.

그럼 어떻게 공격자는 존재하지 않는 ETH를 Wormhole을 통해 Solana 체인 상의 weETH를 발급받을 수 있었을까?

취약점은 Solana 체인에 연결된 Wormhole contract 상에서 signature verification 코드에 존재했다.
Ethereum 체인에서 오는 메시지를 verify_signatures 함수를 통해 검증하고, 문제가 없으면 Solana 체인상의 Wormhole wrapped ETH(weETH)를 발급한다. 
![VerifySignaturesFunction](/assets/images_post/2022-02-06-Wormhole-hack/verify_signatures_function.png)

Solana는 사용자가 함수를 호출할 때 input account를 지정할 수 있는데, 그 중에 하나는 Program(Solana에서는 Contract를 Program으로 부른다)에서 사용하는 system program인 sysvar::instruction이다. 
Solana에는 특별한 sysvar::instruction이 존재하는데, 이는 이전에 실행된 instruction에 대한 정보를 제공한다. 
![VerifySignaturesStruct](/assets/images_post/2022-02-06-Wormhole-hack/verify_signatures_struct.png)

verify_signatures 함수는 이 sysvar instruction account가 진짜 sysvar instruction account 인지 검증하지 않았다. 
따라서 공격자가 자신이 만든 account를 sysvar instruction account로 설정하여 메시지를 보내면 verify_signatures 함수에서 이를 검증없이 sysvar instruction으로 간주하여 사용하게 된다. 
verify_signatures 함수에서는 함수 이름에서 알 수 있듯이 받은 메시지의 signature를 검증하게 되는데, 이 때 사용하는 함수를 공격자가 입력한 account의 함수로 호출하게 됨에 따라 검증을 통과하게 만들 수 있는 것이다.

공격자가 공격하기 전에 0.1 ETH를 예치했었는데 이 때의 input account를 보면 실제 sysvar instruction account였다.
![LegitEthDeposit](/assets/images_post/2022-02-06-Wormhole-hack/legit_eth_deposit.png)

하지만 120,000 ETH를 예치한 공격 transaction을 보면 공격자가 설정한 다른 account임을 확인할 수 있다.
![FakeEthDeposit](/assets/images_post/2022-02-06-Wormhole-hack/fake_eth_deposit.png)

이를 이용하여 공격자는 손쉽게 Solana 상에서 Wormhole wrapped ETH를 발급받게 되었던 것이다.

## 취약점 패치

취약점 패치는 verify_signatures 함수에서 input account가 진짜 sysvar instruction account인지 체크하는 코드를 추가하는 것으로 이루어졌다.
![VerifySignaturesPatch](/assets/images_post/2022-02-06-Wormhole-hack/verify_signatures_patch.png)

## 취약한 내용이 공격 이전에 공개되었었다?

공격이 있기 전인 1월 13일날 작성된 commit에는 해당 취약점을 패치하는 코드가 이미 들어가 있었다.
![JanuaryPatch](/assets/images_post/2022-02-06-Wormhole-hack/january_patch.png)

해당 commit은 공격이 발생하기 9시간 전에 github에 올라왔으며, 해당 패치가 배포되기 전에 공격이 이루어졌다고 한다.

정확한 것은 알 수 없지만 아마도 공격자는 github commit을 모니터링하고 있다가 해당 취약점 패치를 확인하고 그 즉시 공격을 실행한 것이 아닌가 추측해볼 수 있다.

## 참고

* https://wormholecrypto.medium.com/wormhole-incident-report-02-02-22-ad9b8f21eec6
* https://twitter.com/wormholecrypto/status/1489001949881978883
* https://extropy-io.medium.com/solanas-wormhole-hack-post-mortem-analysis-3b68b9e88e13
* https://twitter.com/kelvinfichter/status/1489041221947375616
* https://twitter.com/AlexSmirnov__/status/1489044639768268803
* https://twitter.com/samczsun/status/1489044999211823107

[Wormhole]: https://docs.wormholenetwork.com/wormhole/
[WormholeNetwork]: https://wormholecrypto.medium.com/introducing-wormhole-32b16d795c01
[Chorus One]: https://chorus.one
[Staked.us]: https://staked.us
[P2P Validator]: https://p2p.org
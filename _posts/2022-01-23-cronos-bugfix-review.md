---
title: "Cronos Theft of Transaction Fees Bugfix 리뷰"
author: IceBear
categories:
  - Vulnerability
tags:
  - Cronos
  - Ethermint
  - Theft of Transaction Fees
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

ImmuneFi 블로그 글인 "Cronos Theft of Transaction Fees Bugfix Postmortem"을 정리해보고자 한다.
원문은 [ImmuneFi 블로그]를 참조하면 된다.

블록체인에서는 Transaction을 통해 모든 것이 이루어진다고 할 수 있다. 
Transaction이 처리되도록 하기 위해서 수수료(Fee)를 지불하게 되는데, Fee를 많이 지불하면 할수록 해당 Transaction이 빨리 처리될 수 있다.
Cronos 블록체인의 Ethermint 클라이언트에서 Fee를 훔치는 방법을 [zb3]라는 해커가 발견하였다.

## Cronos 블록체인과 Ethermint

Cronos는 EVM(Ethereum Virtual Machine)과 호환가능한(compatible) Cosmos SDK로 구현된 블록체인으로, Crypto.com의 확장 프로젝트이다.
![Cronos](/assets/images_post/2022-01-23-immunefi-cronos/cronos.png)
그리고 이를 기반으로 만들어진 것이 Ethermint이다.
![Ethermint](/assets/images_post/2022-01-23-immunefi-cronos/ethermint.png)
따라서 Ethermint를 통해 EVM 기반 블록체인과 Cosmos 기반 블록체인을 동시에 지원하므로, 두 체인간 Transaction을 연결(Bridge)하거나 전파(Propagate)할 수 있다.

Ethermint는 MsgEthereumTx를 이용해 Ethereum Transaction을 SDK Message로 감싼다(Wrap).
그리고 Transaction은 Ethermint의 Handler들을 통해 처리되는데, AnteHandler는 그 중에 하나이다.
AnteHandler는 수수료 처리나 Signature 검증과 같은 사전작업을 수행한다. 그리고 Cosmos SDK로 된 Transaction에만 적용된다.

## 취약점 분석

Ethermint는 Ethereum transaction을 보내고자할 때 JSON-RPC Endpoint를 사용한다.
이를 통해 transaction을 수행하면 MsgEthereumTx로 포맷이 변경되며, ExtensionOptionsEthereumTx 구조체가 추가된다.
노드에 의해 transaction이 처리될 때 ExtensionOptionsEthereumTx는 EtherGasConsume Handler를 포함한 AnteHandler로 처리되도록 한다.
EtherGasConsume Handler는 Gas 가격 x Gas Limit 만큼 계정에서 차감하고, transaction이 수행되고나면 수수료는 반환된다.

취약점은 MsgEthereumTx에 ExtensionOptionsEthereumTx가 존재하는지 확인하는 코드가 없다는데 발생한다.
ExtensionOptionsEthereumTx가 없는 MsgEthereumTx를 공격자가 생성하게되면, 노드에서 EtherGasConsumeDecorator Handler가 호출되지 않아 결과적으로 Gas가 차감되지 않는다.
그리고 Transaction이 처리된 이후 사전에 차감되었어야할 수수료를 돌려받게 된다.

[ImmuneFi 블로그]: https://medium.com/immunefi/cronos-theft-of-transactions-fees-bugfix-postmortem-b33f941b9570 
[zb3]: https://github.com/zb3

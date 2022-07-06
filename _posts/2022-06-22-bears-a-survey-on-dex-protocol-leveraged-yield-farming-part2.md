---
title:  "A Survey On DEX Protocol Part1: Leveraged Yield Farming-Part2(Tokenflow Analysis)"
excerpt: "DeFi 취약점 연구를 위한 DeFi 프로토콜 분석, 그 첫번째 Alpaca Finance에 대한 Overview에 이어 Tokenflow 분석 결과에 대한 글입니다."

author: Panda
categories:
  - Background
  - Smart Contract
  - Bears Team
  - Tokenflow Analysis
tags:
  - DeFi
  - Survey
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2022-07-5 7:00:00 +09:00
lastmod: 2022-07-5 20:00:00 +09:00
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

# Introudction

# Project Configuration Troubleshooting
## CASE#1 : Compile Failed
## CASE#2 : Metamask(BalanaceOf doen't work)
## CASE#3 : import failed(Hardhat)

# Service components(Source Code Level)


# Tokenflow Analysis
Tokenflow라는 단어는 구글에 검색해도 잘 안 나올겁니다. 왜냐하면 제가 만든 용어라서 그렇습니다. 일반적으로 우리가 기존 시스템(x86, Android, iOS, ...)에서 취약점을 탐지하기 위해서 분석을 할 때, 크게는 두 가지 관점에서 분석을 합니다. 바로 Controlflow, Dataflow입니다. Controlflow는 함수 호출을 중심으로 기능을 분석하면서, 취약한 로직, 코드가 있는 부분을 발견 버그를 취약점을 발전시키는 것이라면 Dataflow 분석은 입력데이터로 부터 이 입력 데이터가 영향을 주는 코드를 중심으로 분석하면서 취약점을 탐지하는 분석입니다. Dataflow기반 취약점 탐지 방법으로는 가장 유명한 기법이 Taint analysis라고 볼 수 있습니다. 이러한 관점을 Blockchain에게 적용했을 때 결국 Smart Contract에서의 취약점이 의미가 있을려면 Token의 무단 인출이 가능해야 함으로 Token을 중심으로 DApp을 분석하자는 관점으로 Tokenflow라는 용어를 만들어 봤습니다. 스마트 컨트렉트 생태계애서의 Dataflow라고 생각할 수 있습니다. 앞으로 제 포스팅에서는 Tokenflow, Tokenflow analysis라는 용어를 계속 사용할 계획입니다.


## The Overview Tokenflow

| ![Image Alt 텍스트]({{"/assets/images_post/2022-06-22-bears-a-survey-on-dex-protocol-leveraged-yield-farming-part2/alpaca_farm02.png"| relative_url}})  |
| 그림.1 Alpaca Finance Leveraged Yield Farming 중심의 서비스 관계도 |

* Alice : 대여자(Lender), BNB토큰을 Lending Pool, 코드상 Vault에 예치하고 대응하는 ibToken인 ibBNB를 받게 된다. 관련코드는 [여기](https://github.com/alpaca-finance/bsc-alpaca-contract/blob/c6fafa2a9f32604464ed3a5116384a476800e45c/solidity/contracts/6/protocol/Vault.sol#L207)를 보면됩니다. 실제 코드를 보면 BNB가 아니라 WBNB(Wrapped BNB)라는 것을 확인할 수 있는데, 이것에 대해서는 좀더 조사를해서 내용을 보완하겠습니다. 여기서 우리가 알아야할 것은 최초의 외부 지갑에서 돈이 흘러들어가는 것을 과정을 분석하기 위해서 Vault.sol 파일을 분석해야한다는 사실입니다.

~~~
// Solidity
address vaultContractAddress = '0xd7D069493685A581d27824Fc46EdA46B7EfC0063'; // BNB Vault
if (msg.value == 0) { // if no native token is sent, then it is a ERC20/BEP20 token deposit
	IERC20(tokenAddess).safeTransferFrom(address(msg.sender), address(this), amountToken);
}
// Allow transfer to vault
SafeToken.safeApprove(tokenAddess, vaultContractAddress, amountToken);
// Deposit to vault
IVault(vaultContractAddress).deposit(amountToken);
~~~

예치시 받는 ibToken의 가치는 아래와 코드의 계산과정을 거쳐서 결정됩니다.
Vault는 ERC20 토큰 컨트렉트를 상속하기 때문에 totalSupply() 함수는 [Openzeppelin 사이트](https://docs.openzeppelin.com/contracts/2.x/api/token/erc20#IERC20-totalSupply--)에서 확인 가능하다. totalToken()은 Vault내 BNB토큰의 전체 개수이고 totalSupply()는 전체 발행된 ibToken의 개수입니다.

~~~
// Solidity
address vaultContractAddress = '0xd7D069493685A581d27824Fc46EdA46B7EfC0063'; // BNB Vault
IVault vault = IVault(vaultContractAddress);
uint256 ibTokenAmount = ...;
uint256 ibTokenPrice = vault.totalToken()).div(vault.totalSupply();
uint256 underlyingTokenAmount = ibTokenAmount.mul(ibTokenPrice);
~~~

* Bob : 이자 농사 농부(Farmer), 그림.6에서는 BTC/BNB 페어에서 이자농사를 하기를 원하며, BTC를 예치하고, BNB를 빌리는 형태로 레버리지 이자 농사를 실시, 이 때 Bob의 전제는 BTC가격이 오를 것을 예상하고 농사를 하는 것이며, 앞에서 설명한 Long(롱) 포지션에 해당합니다. 소스코드에서도 이 동작을 확인하고 싶어서 찾고 있으며, 아직 확인하지 못 했습니다.
* Erin : 청산 봇(Liquidator bot), Bob과 같이 레버리지 이자 농사를 하는 경우 담보물의 가치 변동에 따른 청산이 발생할 수 있는데, 이 작업을 돕는 봇이며 청산시 보상으로 $$5%$$ 를 수수료를 챙기며, 이 수수료로 Alpaca 거버넌스 코는 Alpaca를 소각하는데 활용, Alpaca 토큰의 가격 상승의 동력을 만들어냅니다. 변동성이 강할때 청산이 많이 발생함으로 그 때 Alpaca Finance는 많은 수익 창출이 가능합니다. 코드상 확인이 필요합니다.
* Carlos : 보상 헌팅 봇(Bounty Hunter Bot), 각각의 풀에 보상들이 자동으로 재투자되는지 확인하고 감시하는 역할을 담당하는 봇이라고 하는데, 정확하게 확인이 필요할 것 같습니다. 모든 이자 농부들이게 복리 이자를 자동 지급하는 역할을 한다고 하며, 이 역할을 수행함으로서 보상의 약 $$3%$$ 의 수수료를 챙긴다고 되어 있습니다. 이 수수료를 Alpaca 개발팀에서 가져가게 됩니다.

위 Alpaca Finance내 참여자들의 구성을 볼때, 우리가 집중해서 분석해야할 코드는 Alice와 Bob인 것을 확인할 수 있습니다. 결국 Vault에 큰 돈이될 토큰이 있으므로 Alpaca 서비스에서 돈을 빼낸다면 Vault에 집중해야할 것으로 보이고, Yield Farming쪽에서 돈을 빼낼 방법을 고민할려면 연동된 서비스 코드까지 광범위하게 분석해야할 것입니다.

Alpaca Finance 서비스에서는 일반 시중은행에서 다양한 투자회사의 금융상품을 판매하는 것 처럼 다양한 연동서비스의 상품을 취급하고 있습니다. Alpaca Finance에서 제공하는 상품은 다음과 같습니다.

* PancakeSwap Farms(BNB)
* Mdex Farms(BNB)
* Biswap Farms(BNB)
* SpookySwap Farms(Fantom)
* WaultSwap Farms(Deprecated)

Alpaca 서비스의 contract 소스코드에서도 위 서비스를 확인할 수 있습니다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-06-22-bears-a-survey-on-dex-protocol-leveraged-yield-farming-part2/alpaca_project01.png"| relative_url}})  |
| 그림.2 Alpaca Finance 프로젝트 폴더, Alpaca Finance의 경우 외부 서비스의 풀을 그대로 활용하고 있음을 알 수 있다.|

또한 [이곳 문서](https://docs.alpacafinance.org/leveraged-yield-farming/pool-specific-parameters-1/pool-specific-parameters#pancakeswap-tusd-pairs-1)를 확인하면, PancakeSwap 풀중에서 Alpaca Finance에서 지원하는 서비스별 Contract주소를 확인할 수 있습니다.

Alpaca 서비스의 경우 자신들이 서비스하는 [토큰쌍 풀의 컨트렉트(Contract)주소](https://github.com/alpaca-finance/bsc-alpaca-contract/blob/c6fafa2a9f32604464ed3a5116384a476800e45c/.mainnet.json#L709)를 json형태로 유지관리 하고 있는 것을 확인할 수 있습니다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-06-22-bears-a-survey-on-dex-protocol-leveraged-yield-farming-part2/alpaca_pancake01.jpg"| relative_url}})  |
| 그림.3 Alpaca Finance와 PancakeSwap간의 Tokenflow |

## The Detailed Tokenflow(With respect to BNB)

소스코드 분석을 기반으로 EOA(개인지갑)에서 부터 Pancakeswap까지의 흐름을 도식화한 것입니다. 아래 그림.4에서 PancakeswapWork가 바로 Yield Farming에 관련된 핵심 컨트렉트입니다. 

| ![Image Alt 텍스트]({{"/assets/images_post/2022-06-22-bears-a-survey-on-dex-protocol-leveraged-yield-farming-part2/overview_alpaca_tokenflow.png"| relative_url}})  |
| 그림.4 EOA에서 PancakeSwap까지의 Tokenflow |

### 왜 Yield Farming 컨트렉트 파일의 접미사가 Worker일까?

* 소스코드를 분석하면 다양한 Worker들이 존재하는 것을 확인할 수 있음
* 연동되는 서비스별 Work가 존재함
* 개인적인 생각으로는 Vault의 자산 증식에 기여하는 일꾼이라는 개념으로 Worker라는 접미사를 붙인 것으로 추정함

### Overview를 통해 확인하는 Alpaca Finance의 수익 구조

* Alpaca Finance 서비스를 통해 Yield Farming을 위해 부족한 돈은 위 그림.4에서 처럼 Alpaca Finance내 Vault에서 대출 받게 됨. 
* 이는 대출 수수로 형태로 Vault의 자산을 늘이게 됨
* 또한 Vault에 유동성을 공급하는 사람들이 추가적인 이자수익을 획득하기 위해 ibToken를 FairLaunch를 통해 Alpaca Token를 받고, 이 Alpaca Token를 스테이킹하거나 유통하여 Alpaca Token을 유통시킴, 이 유통의 힘으로 Alpaca Finance의 가치 상승에 기여함
* Yield Farming을 하는 가운데, 청산이되는 농부들이 있을 것이며, 해당 청산을 통해 Vault 및 Alpaca Token의 가치를 향상 시켜서 이윤을 창출함
* Alpaca Token의 가치 상승, 에초에 개발자 그룹에 할당된 Alpaca Token의 매도를 통해, 개발자 그룹은 현금 확보가 가능함


# Conclusion

# References
* [https://medium.com/@versofinance/understanding-leveraged-yield-farming-72b5f8609a0a](https://medium.com/@versofinance/understanding-leveraged-yield-farming-72b5f8609a0a)
* [https://alphafinancelab.gitbook.io/alpha-homora-vbsc/what-is-leveraged-yield-farming](https://alphafinancelab.gitbook.io/alpha-homora-vbsc/what-is-leveraged-yield-farming)
* [https://docs.alpacafinance.org/help-center/alpaca-academy/lesson-3-liquidation-risk-in-leveraged-yield-farming](https://docs.alpacafinance.org/help-center/alpaca-academy/lesson-3-liquidation-risk-in-leveraged-yield-farming)
* [https://alphaventuredao.io/](https://alphaventuredao.io/)
* [https://docs.kleva.io/v/copy-of/undefined-1/undefined-3/3](https://docs.kleva.io/v/copy-of/undefined-1/undefined-3/3)
* [https://medium.com/alpaca-finance/introducing-alpaca-finance-d6e858896efd](https://medium.com/alpaca-finance/introducing-alpaca-finance-d6e858896efd)
* [https://thedefiant.io/leveraged-yield-farming/](https://thedefiant.io/leveraged-yield-farming/)
* [https://francium.io/app/calculator](https://francium.io/app/calculator)
* [https://github.com/alpaca-finance/bsc-alpaca-contract/tree/main/solidity/contracts](https://github.com/alpaca-finance/bsc-alpaca-contract/tree/main/solidity/contracts)
* [https://medium.com/immunefi/belt-finance-logic-error-bug-fix-postmortem-39308a158291](https://medium.com/immunefi/belt-finance-logic-error-bug-fix-postmortem-39308a158291)
* [rpc.info](rpc.info)

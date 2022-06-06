---
title:  "A Survey On DEX Protocol Part1: Leveraged Yield Farming"
excerpt: "DeFi 취약점 연구를 위한 DeFi 프로토콜 소개 시리즈, 그 첫번째 Leveraged Yield Farming에 대한 글입니다."

author: Panda
categories:
  - Background
  - Smart Contract
  - Bears Team
tags:
  - DeFi
  - Survey
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2022-06-14 7:00:00 +09:00
lastmod: 2022-06-14 20:00:00 +09:00
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

# Background
## Entities
### Lenders(대여자)
* 기존 금융체계에서 은행에 돈을 예금하는 것과 유사함
* 스테이킹과 달리 자유롭게 예치(Deposit), 출금(Withdraw)할 수 있음
* 대여자의 수익모델은 예치에 대한 이자(interest) 수령을 통한 모델임
* 대여자의 수익은 농부가 얼마나 랜딩풀을 활용하는지에 따라서 지급받는 이자수익도 달라지게 되며, 대여자의 경우 DeFi내에서 비교적 안정적인 장기수익을 기대할 수 있음
### Farmer(농부)
* 이자농사에 참여하는 대상자를 농부(Farmer)로 지칭함
* 이자농사를 위해 일정한 자기 자산을 담보물로 제공해야함
* 랜딩풀에서 자산을 대출(레버리지) 받음으로써 보다 큰 자산을 활용하여 이자농사에 참여할 수 있음
* 대출 비용이 클 수록 포지션청산 및 비영구적 손실에 노출될 가능성이 높음
* 개인적으로 이자 농사 금융상품의 모델에서 대여자가 대지주라면 농부는 소작농이라고 부르고 싶고, 소작농의 레버리지를 활용한 고배율 투자가 실패했을 때 그 포지션 청산비용이 다 대여자, 대지주에게 공급될 높은 APY 이자 지급에 활용된다. 물론 소작농이 고배율 레버리지 포지션을 잡 활용하면 큰 돈을 벌 수 있지만, 변동성이 높은 암호화폐 시장에서 돈일 잃는 쪽이 더 많을 것 같고, 인간의 탐욕을 기반으로하는 선물시장과 비슷한 구조라 생각이 듭니다.

### Staker(스테이커)
* 대여자는 자산을 랜딩풀에 예치하면 ibToken을 수령
* 이 ibToken을 Stacking Vault(금고)에 예치함으로써 이자수익을 더 할 수 있음
* 이 때 스테이킹에 대한 이자는 거버넌스토큰으로 지급함
* 일반적으로 Stacking을 하면 최소한의 Lockup 기간이 존재하며, 그 전에 Unstacking하면 패널티를 부과하는 곳도 존재함
* DeFi에서 스테이킹을 통한 유동성에 Lock을 걸려고 하는 이유는 뱅크런과 같은 것을 방지하기 위함이며, 일상생활에서 은행 예금의 경우 만기를 채우지 않은 상태에서 해지시 손해가 있는것과 동일한 관점임

## What is Yield Farming(이자농사)
이자 농사의 정의를 명확하게 정리한 문서는 찾지 못 했습니다. 하지만 여러 문서를 살펴본 결과, 다음과 같이 이자농사를 정의할 수 있을 것 같습니다.
* 첫번째 토큰을 예치하여 유동성을 공급하고, 이 것으로 부터 이자를 얻는 행위도 이자농사라고 할 수 있습니다. 가장 쉽고 가장 안전한 방법입니다. 유동성 공급시 인출할 때 시간적인 제한을 두는 DeFi도 있으므로, 이 번 루나사태와 같은 것이 발생하면, 막대한 손실을 볼 수 있습니다.
* 일반적인 DeFi에서 토큰쌍을 예치하고, 고수익의 이자를 얻을 수 있는 풀(Pool)을 서비스하는 DeFi가 많이 있습니다. 예를 들면 USDC-ETH와 같이 스테이블코인과 변동성이 높은 토큰을 쌍으로하는 풀입니다. USDC와 예치하는 USDC와 동일한 가치의 ETH를 입금하면 해당 풀에서 제공하는 이자 APR(Annual Percentage Rate)를 얻을 수 있습니다. 각 DeFi에서 제공하는 Pool서비스를 모아서 쉽게 예치할 수 있게 도와주는 서비스를 하는 서비스를 하는 곳도 있습니다. 이러한 풀을 묶어서 보여주는 것을 농장(Farm)이라고 지치하고 있으며 풀은 큰 농장에 있는 다양한 밭이라고 생각하시면 됩니다. 아래 그림은 Alpaca Finance에서 제공하는 농장(Farm) 서비스내 풀(Pool) 화면캡쳐입니다. 

| ![Image Alt 텍스트]({{"/assets/images_post/2022-06-02-bears-a-survey-on-dex-protocol-leveraged-yield-farming/alpaca_farm01.png"| relative_url}})  |
|:--:| 
| 그림.1 Alpaca Finance의 Farm서비스에서 제공하는 풀의 예시  |

* 그럼 항상 양쪽 토큰을 다 가지고 있어야 이자 수익을 얻을 수 있는 것은 아닙니다. USDC-ETH풀에 돈을 넣을 때 만약에 USDC만 있다면 나의 돈에 해당하는 ETH가 없을 때 바로 ETH 금고에서 대출을 받으면 됩니다. 대출을 받았을 경우 대출이자보다 높은 APR을 달성할 수 있어야 하며, 대출 이자도 경우에 따라 10%정도 될 수 있기 때문에 손해를 볼 수 있다는 생각을 하고 투자해야 합니다. 
* 여기에 또 암호화폐의 변동성이 큰 영향을 주게되는데, 만약에 ETH의 가격이 점점 내려가면 내가 같아야할 ETH의 가격이 싸졌기 때문에(공매도와 유사) 내가 갚아야할 돈이 적어지고 만약에 ETH의 가격이 오르면 내가 갚아야할 돈이 커지기 때문에 반드시 무엇을 예치하고 무슨 코인을 빌릴지 잘 생각해야 합니다. 정말 운이 없이 시장의 방향과 반대로 투자하게 되면 한 푼도 못 남길 수 있습니다.

## What is ibToken(interest bearing Token)
ibToken은 토큰을 예치했을때 받게되는 징표와 같은 것입니다. Interest Bearing (ib)라는 접두사가 붙은 이유는 이자 농사를 하는 사람이 청산당하거나, 대출을 같을 때의 제공하는 이자등이 해당 금고에 누적이되고 그 때 Vault(금고)에 상황에 따라 이자를 포함하는 토큰을 돌려 받기 때문에 붙은 접두사입니다. 현재상태 금고(Vault)의 지분률이라고 보시면 정확합니다.

코든 랜딩풀은 고유의 ibToken을 가지고 있습니다. 만약에 Alpaca의 경우 ibBNB, Kleva의 경우는 ibWEMIX와 같이 랜딩풀에 따라 ibToken을 민팅하게 됩니다.

여기서 DeFi 설계자들은 고민하게 됩니다. 랜딩풀에 적절한 수준의 유동성을 공급해야하는데, 사람들이 돈을 묶어 둘 필요가 있습니다. 그래서 ibToken을 스테이킹할 수 있게 하고, 스테이킹하면 ibToken에 대응하는 거버넌스 토큰을 지급하게 됩니다. 만약에 이 거버넌스코튼의 가격이 올라가면 대여자는 추가적인 수익을 얻을 수 있고, 장시간 돈을 맡기게되며 DeFi는 안정적인 유동성을 확보할 수 있습니다. 다만 스테이킹 서비스의 경우 최소 예치기간을 두는 곳이 많고, 해치기간 전에 스테이킹을 해제 못 하게 하거나 하더라고 패널티를 부과하는 곳도 있으니 주의가 필요합니다.

~~~
  function _deposit(uint256 amountToken) internal {
    uint256 total = totalToken().sub(amountToken);
    uint256 share = total == 0 ? amountToken : amountToken.mul(totalSupply()).div(total);
    _mint(msg.sender, share);
    require(totalSupply() > 1e17, "no tiny shares");
  }
~~~

## What is position(Liqudiation)

| ![Image Alt 텍스트]({{"/assets/images_post/2022-06-02-bears-a-survey-on-dex-protocol-leveraged-yield-farming/position.png"| relative_url}})  |
|:--:| 
| 그림.2 Binance Future 거래 화면  |


## What is Leveraged Yield Farming(레버리지 이자농사)

| ![Image Alt 텍스트]({{"/assets/images_post/2022-06-02-bears-a-survey-on-dex-protocol-leveraged-yield-farming/francium01.png"| relative_url}})  |
|:--:| 
| 그림.3 Francium Leverage Farming서비스 시뮬레이터에서 USDC를 공급하고 Solana를 빌렸을 경우  |


| ![Image Alt 텍스트]({{"/assets/images_post/2022-06-02-bears-a-survey-on-dex-protocol-leveraged-yield-farming/francium02.png.png"| relative_url}})  |
| 그림.4 Francium Leverage Farming서비스 시뮬레이터에서 Solana를 공급하고 USDC를 빌렸을 경우 |


# Tokenflow: Alpaca Finance
## PancakeSwap Farms
## Mdex Farms
## Biswap Farms
## SpookySwap Farms
## WaultSwap Farms



# Tokenflow: Kleva

# The Difference between Alpha, Alpaca Finance and Kleva

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

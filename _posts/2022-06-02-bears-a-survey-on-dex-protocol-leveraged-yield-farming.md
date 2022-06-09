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
DeFi에 대한 취약점 탐지를 위해 현재 DeFi서비스의 프로토콜을 이해하는 것이 중요하기 때문에, 실제 코드 감사에 앞서 DeFi서비스 프토토콜에 대해서 조사한 내용을 BEARS 세미에서 공유하는 목적으로 작성된 포스트입니다.

# Introudction
이 번 포스트는 "Leveraged Yield Farming" 프로토콜에 대한 포스트입니다. DeFi의 취약점을 코드 기반으로 탐색하고 공격하는 연구에 앞서 DeFi의 다양한 프로토콜을 이해하고 전체적인 토큰 플로우를 알아보는 시간을 먼저 가지는 것이 중요하다는 생각을 다양한 취약점 케이스 분석 글을 확인하던 중 들었습니다. 실제 취약점의 원인을 분석하면 그 원인 자체는 단순합니다. 개발자의 실수, 사용하지 말아야할 주소 사용, Reentrancy와 같은 고전 취약점 그러나, 이런 취약점을 이해하고 있더라도, DeFi 서비스가 어떤 유형에 포함되고 전체적인 토큰의 흐름을 알지 못 하면 실제 속칭 돈이 되는 취약점을 찾기가 어려울 수 있습니다. 따라서 DeFi 프로토콜을 먼저 이해하고, 해당 프로토콜에서 주로 발생하는 취약점을 케이스 스터디하고 해당 취야검 사례를 기반으로 유사한 서비스에 대해서 비슷한 취약점이 존재하는지 살펴보는 순으로 진행하는 것이 보다 적절한 접근으로 보입니다. 이 번 포스트는 DeFi 프로토콜의 첫 번째로 Leveraged Yield Farming에 대해서 알아보고 이후 포스트에서 해당 서비스와 연관된 [취약점 사례](https://medium.com/immunefi/belt-finance-logic-error-bug-fix-postmortem-39308a158291)를 다음 포스트에서 살펴보로록 하겠습니다.


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
이제 Leveraged Yield Farming에 들어가기전에 Position에 대해서 알아보겠습니다. 포지션은 시장을 바라보는 나의 시각이라고 보시면 되고 long position은 앞으로 투자 상품의 가격이 상승할꺼라는 것이고 short position은 가격이 하락할 것이라는 관점입니다. 아래 그림은 바이낸스 비트코인 선물시장이며 비트코인이 상승할 것이라면 long 포지션을 사고, 가격이 하락할 것이라면 short 포지션을 사면 됩니다. 포지션 거래와 꼭 함께 있는 것이 레버리지(leverage)입니다.

만약에 아래 그림에서 leverage를 10배를 하게되었을 때 내가 만약에 long포지션을 샀고, 비트코인이 10%가 올랐다면 100% 수익을 올릴수 있습니다. 만약 반대의 경우가 발생하면 -100%가 되는 것이고 이 때 청산(liqudiation)이 발생하게 됩니다. 일종의 홀짝 게임이라고 생각하시면 됩니다. 청산이되면 내가 포지션 구입 비용에 들어갔던 돈은 다른쪽(short 포지션 매수자) 수익과 바이낸스 거래 수수료로 사용됩니다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-06-02-bears-a-survey-on-dex-protocol-leveraged-yield-farming/position.png"| relative_url}})  |
|:--:| 
| 그림.2 Binance Future 거래 화면  |

Leveraged Yield Framing에서는 다들 예상하셨다시피 leveraged를 사용할 수 있으며, 이때 중요한게 이자농사 풀에 투입될 토큰중 어느 토큰을 빌릴지 잘 선택하는게 중요합니다.

## What is Leveraged Yield Farming(레버리지 이자농사)

내가 가진 담보토큰 보다 훨씬 큰돈을 가지고 이자농사를 하면 훨씬 큰 이자를 벌 수 있다는 돈에 대한 인간의 탐욕을 기반으로하는 서비스입니다. 더욱이 변동성이 큰 암호화폐 시장의 특징과 적절한 대여 토콘의 선택은 이익을 극대화할 수도 있고, 최악의 손실을 볼 수 도 있습니다.

프랑슘(Francium)이라고 솔라나 네트워크에서 Alpaca와 비슷한 서비스를 하는 곳입니다. 이 프랑슘에서는 Leveraged Yield Farming이전에 다양한 시뮬레이션을 할 수 있게해주고, 적절한 레버리지 및 대출 토큰 선택에 도움주고 있습니다. 레버리지 이자 농사를 할 때 가급적 한 쪽 토큰을 스테이블 토큰으로해서 변동성을 조금이라도 낮추는게 좋습니다. 변동성 높은 토큰 두개를 하는게 수익을 극대화할 수 있으나 반대의 경우도 생각해야합니다. 

현재 22년 6월과 같이 베어(Bear) 마켓인 경우는 아무래도 가격이 내려갈 가능성이 높은 Solana를 대출받아서 이자농사를 하는게 나중에 갚아야할 솔라나를 생각하면 좋은 선택이 될 수 있습니다. 그림 3에서와 같이 솔라나 가격이 상승하면 청산될 수 있습니다. 

| ![Image Alt 텍스트]({{"/assets/images_post/2022-06-02-bears-a-survey-on-dex-protocol-leveraged-yield-farming/francium01.png"| relative_url}})  |
|:--:| 
| 그림.3 Francium Leverage Farming서비스 시뮬레이터에서 USDC를 공급하고 Solana를 빌렸을 경우  |

그러나 솔라나가 지금 저점이라고 생각하고 솔라나를 공급하고 USDC를 대출받을 수 있습니다. 나중에 솔라나 가격이 올라가면 적은 솔나라로 USDC를 값을 수 있으니 이자 플러스 훨씬 이득이 됩니다. 그러나 솔라나 가격이 더 떨어진다면 청산되게 됩니다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-06-02-bears-a-survey-on-dex-protocol-leveraged-yield-farming/francium02.png"| relative_url}})  |
| 그림.4 Francium Leverage Farming서비스 시뮬레이터에서 Solana를 공급하고 USDC를 빌렸을 경우 |

즉 Leveraged Yield Farming의 경우 시장의 상태에 따라 받을 수 있는 이자를 극대화할 수 있으며, 실제 풀(Pool)에서 표시하고 있는 APR보다 훨씬 많은 이자 수익을 확보할 수 있습니다.

Kleva Protocol에서 예로 레버리지 이자 농사를 설명해보겠습니다.

지금 현재 2022년 6월6일 WEMIX의 경우 \$ 3.694이고 Klaytn의 경우 \$ 0.4069 대략 계산을 쉽게 하기 위해서 교환비가 1:10 정도 된다고 가정하겠습니다.

투자가 홍길동씨는 100 WEMIX를 가지고 있습니다. 100WEMIX를 가만히 거래소에 두기 보다, 장기적으로 BEAR 마킷이고 WEMIX 토큰을 매도할 생각이 없기 때문에 이자농사를 할 계획인데 단기적으로 이익을 극대화하기 위해 레버리지를 사용할 생각을 합니다.

* 홍길동씨의 자산은 100 WEMIX입니다. 최대 이익을 위해 레버리지 계수를 3을 선택합니다.
* 이제 홍길동씨의 자산은 300(100 * 3) WEMIX입니다. 이 300 WEMIX를 자산을 형성하기 위해 200 WEMIX를 대여풀에서 빌려야 합니다.
* 이 때 선택을 할 수 있습니다. WEMIX가 Klaytn 상승률보다 높아서 나중에 교환비가 1:100, 1:1000이 될 것이라고 생각하면 Klaytn을 대출하는게 훨씬 이득입니다. 만약에 반대라면 지금 WEMIX를 Klaytn으로 바꾸고, WEMIX를 대출받는게 더 좋습니다.
* 여기서는 WEMIX가 더 상승한다고 홍길동씨는 판단했기 때문에 200 WEMIX에 대응하는 2000 Klaytn을 빌렸습니다.
* 풀에 넣을 때에는 1:1 비율로 집어 넣어야 하기 때문에 500 Klaytn을 50 WEMIX로 바꾸고 150 WEMIX, 1500 Klaytn을 풀에 집어 넣었습니다. 
* 한 달뒤 이자풀에서 돈을 인출합니다. 이 때 교환비가 1:100이 되었습니다. 일단 1500 klaytn에서 부족한 500 Klaytn에 대한 5 WEMIX만 교환해서 2000 Klaytn과 이자를 갚고 풀 이자와 추가적인 가격변동에 의한 WEMIX 수익을 얻게 됩니다.

위의 경우는 정말 운이 좋은 경우라 볼 수 있고, 보통은 암호화폐 두개를 페어로 하는 것은 고려사항이 많아서 너무 위험한 것 같습니다. 개인적으로는 스테이블코인과 암호화폐 쌍이 보다 안전해 보입니다.

# Tokenflow: Alpaca Finance

## Configuration
### Types of nodes
* Archive nodes : Has data since genesis block.
* Full nodes : Receive copies of transactions. Has the current state of the blockchain.
* Light nodes : Doesn't have the entirety of the current blockchain state and depends on a full node, useful for low memory and computational devices.
* Miner nodes : Miner nodes verify transactions and add them to the blocks. They then mine those blocks and secure the blockchain with proof of work. 

### Yarn Compile and Yarn
Alpaca Finance의 프로젝트 설명을 보면 yarn을 실행하라고 되어 있습니다. 실제로 sudo apt install yarn으로 yarn을 설치하고 실행하면 에러가 발생할 수 있습이다 그럴 경우 아래의 명령어로 yarn을 일단 재설치해보시기 바랍니다.

~~~
sudo apt remove cmdtest
sudo apt remove yarn
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update
sudo apt install yarn
~~~

yarn과 npm install도 궁합이 안 좋은 것 같습니다. yarn을 재설치하고 난 뒤에 <span style="background-color:#fff5b1">"yarn error An unexpected error occurred: "Commit hash required".</span> 에러가 발생한다면 git clone으로 새로 alpaca finance 프로젝트를 다운로드 받아야 합니다.

yarn을 실행하다보면 아래와 같은 에러가 출력이 되는 경우가 있습니다. 이 에러를 해결하기 위해 많은 시간을 투자하였는데, Alpaca 프로젝트 관련자로 추정되는 사람이 한 답변 한 개를 찾을 수 있었습니다. <span style="background-color:#fff5b1">해결 방법은 yarn compile을 실행하고 다시 yarn을 실행</span>하라는 것이였습니다. 
~~~
$ tsc -p tsconfig.cjs.json
error TS18003: No inputs were found in config file '/mnt/d/Playground/BEARS/case/bsc-alpaca-contract/tsconfig.cjs.json'. Specified 'include' paths were '["./typechain/**/*.ts"]' and 'exclude' paths were '["node_modules","build","cache","artifacts"]'.
~~~

yarn compile을 하면 또 수많은 에러가 생성됩니다. 이 쯤되면 로컬 환경 구축을 통한 분석환경을 구축을 통한 분석을 해당 프로젝트 팀이 의도적으로 방해하고 있다라는 느낌이 듭니다. yarn compile 에러를 해결해야 하는데, WSL 환경이라서 생기는 문제일 수도 있을 것 같습니다.

### Hardhat Installation
만약에 Hardhat이 설치되어 있지 않으면 Hardhat이 없다는 에러가 출력이 될 것 입니다. Hardhat을 설치하는 방법은 아래와 같으며 Alpaca 프로젝트의 경우 NodeJS 패키지 매니저로 yarn을 설치하는 것이 좋습니다.

~~~
npm install --save-dev hardhat
npm install --save-dev @nomiclabs/hardhat-ethers ethers chai @nomiclabs/hardhat-waffle ethereum-waffle

yarn add hardhat
yarn add @nomiclabs/hardhat-ethers ethers chai @nomiclabs/hardhat-waffle ethereum-waffle
~~~

yarn으로 Hardhat 설치시 아래와 같은 에러가 발생한다면 <span style="background-color:#fff5b1">nvm install --lts</span> 명령을 실행 후 다시 설치시도하면 에러를 해결할 수 있습니다.

~~~
"error hardhat@2.9.8: The engine "node" is incompatible with this module. Expected version "^12.0.0 || ^14.0.0 || ^16.0.0". Got "17.0.1"
~~~


### Hardhat fork BSC mainnet
Alpaca finance github 프로젝트를 다운로드하면 DotEnv파일 예제가 다음과 같이 있습니다. Metamask에서 계정을 두개 만들고 각각의 PrivateKey를 Export해서 PRIVATE_KEY와 QA_PRIVATE_KEY를 설정해 줍니다. BSC_MAINET_ARCHIVE_RPC의 경우는 [rpc.info](rcp.info) 사이트에서 BSC Mainent주소를 입력하였습니다.

~~~
# Testnet Env
BSC_TESTNET_PRIVATE_KEY="xxx"
ETH_TESTNET_PRIVATE_KEY="xxx"
FANTOM_TESTNET_PRIVATE_KEY="xxx"

ETH_KOVAN_RPC="xxx"

# Mainnet Env
BSC_MAINNET_PRIVATE_KEY="xxx"
ETH_MAINNET_PRIVATE_KEY="xxx"
FANTOM_MAINNET_PRIVATE_KEY="xxx"

BSC_MAINNET_ARCHIVE_RPC="https://bsc-dataseed.binance.org/"

BSC_MAINNET_RPC="xxx"
ETH_MAINNET_RPC="xxx"
FTM_MAINNET_RPC="xxx"
FORK_RPC="http://127.0.0.1:8545"

DEPLOYER_ADDRESS="xxx"

TYPECHAIN_TARGET="ethers-v5"

QA_PRIVATE_KEY="xxx"
~~~

Hardhat의 경우 localhost RPC의 chainid는 31337이라고 합니다.

## Alpaca Finance

Alpaca Finance의 경우 FairLaunch 서비스라는 것을 강조하고 있습니다. 실제 FairLaunch관련 컨트렉트도 github에서 확인할 수 있습니다. 그 개념을 미뤄 짐작해 보면 다음과 같습니다. 

FairLaunch의 반대 경우를 생각해보면 되는데 바로 ICO 또는 PreSale을 생각하면 됩니다. 일반적으로 블록체인 프로젝트를 시작할 때 자금을 모으기 위해 ICO 또는 토큰 PreSale을 합니다. 소스의 몇몇이 초기 저렴하게 토큰을 구매해서 나중에 가격 상승의 혜택을 보게 됩니다. Alpaca Finance의 경우 이러한 ICO 또는 PreSale을 하지 않고 누구나 Lending 풀에 토큰을 예치하고 해당 ibToken을 스테이킹 함으로써 Alpaca 토큰을 가질 수 있습니다. 소수의 인원의 섬점 효과가 없이 누구나 공평하게 Alpaca 커너넌스 토큰을 소요할 수 있습니다. 이러한 특징을 FairLaunch 서비스로 표현하고 있습니다. [FairLaunch 소스코드](https://github.com/bears-team/bsc-alpaca-contract/blob/c6fafa2a9f32604464ed3a5116384a476800e45c/solidity/contracts/6/token/FairLaunch.sol#L47)를 보면 Alpaca토큰 동작과 관련있음을 확인할 수 있습니다.

## The Overview of Alpaca Finance

| ![Image Alt 텍스트]({{"/assets/images_post/2022-06-02-bears-a-survey-on-dex-protocol-leveraged-yield-farming/alpaca_farm02.png"| relative_url}})  |
| 그림.6 Alpaca Finance Leveraged Yield Farming 중심의 서비스 관계도 |

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

| ![Image Alt 텍스트]({{"/assets/images_post/2022-06-02-bears-a-survey-on-dex-protocol-leveraged-yield-farming/alpaca_project01.png"| relative_url}})  |
| 그림.5 Alpaca Finance 프로젝트 폴더, Alpaca Finance의 경우 외부 서비스의 풀을 그대로 활용하고 있음을 알 수 있다.|

또한 [이곳 문서](https://docs.alpacafinance.org/leveraged-yield-farming/pool-specific-parameters-1/pool-specific-parameters#pancakeswap-tusd-pairs-1)를 확인하면, PancakeSwap 풀중에서 Alpaca Finance에서 지원하는 서비스별 Contract주소를 확인할 수 있습니다.

Alpaca 서비스의 경우 자신들이 서비스하는 [토큰쌍 풀의 컨트렉트(Contract)주소](https://github.com/alpaca-finance/bsc-alpaca-contract/blob/c6fafa2a9f32604464ed3a5116384a476800e45c/.mainnet.json#L709)를 json형태로 유지관리 하고 있는 것을 확인할 수 있습니다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-06-02-bears-a-survey-on-dex-protocol-leveraged-yield-farming/alpaca_pancake.jpg"| relative_url}})  |
| 그림.7 Alpaca Finance와 PancakeSwap간의 Tokenflow |

# Tokenflow: Kleva
* TBA(코드 분석 기반)

# The Difference between Alpha, Alpaca Finance and Kleva
본인 자체 풀보다 다양한 DeFi와 연동하여 풀들을 사용자들이 여러 DeFi이 돌아다니지 않고 편하게 이자서비스를 할 수 있도록 하는 것이 핵심이고 자기들 대여풀에서 돈을 인출해서 해당 토큰 쌍 APR를 활용할 수 있도록 하여, 중간에 수수료를 챙기는 구조의 금융 서비스라고 생각하면 될 것 같습니다.

현재까지의 살펴본 결과로는 개인적으로 이 3걔의 서비스 Alpha, Alpaca, Kleva는 거의 동일한 서비스를 하는 것으로 판단되며, Kleva의 경우 Alpaca와 너무 비슷해서 동일한 코드에서 출발한 것으로 판단될 정도입니다.

# Conclusion

이 번 글에서는 DeFi Protocol중 Leveraged Yield Farming 서비스를 살펴보았습니다. 랜딩풀(Vault)라는 것을 제외하면 다른 토큰 쌍풀(Pool)의 경우는 다른 Swap서비스를 끌어다 서비스를 제공하는 구조였으며, 고배율 레버리지 기능을 제공 암호화폐시장에서의 변동성을 기반으로 이자 농사꾼들이 청산 및 레버리지 사용에 대한 이자농사를 기반으로 돈을 버는 서비스임을 확인할 수 있었습니다.


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

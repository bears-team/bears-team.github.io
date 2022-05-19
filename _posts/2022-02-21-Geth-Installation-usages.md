---
title: "Geth Installation과 기본 사용법"
author: Grizzly
categories:
  - Blockchain
  - Development environment
  - Background
tags:
  - Geth

sitemap :
changefreq : daily
priority : 1.0
# table of contents
toc: true # 오른쪽 부분에 목차를 자동 생성해준다.
toc_label: "table of content" # toc 이름 설정
toc_icon: "bars" # 아이콘 설정
toc_sticky: true # 마우스 스크롤과 함께 내려갈 것인지 설정
---

## Geth
Geth는 Go Ethereum의 약자로, 아마 이더리움 스마트 컨트랙트를 작성해본 경험이 있다면, 한 번쯤 들어봤거나 사용해본 적이 있을 것이다. Go 언어로 작성된 이더리움 클라이언트로써, 이더리움의 메인넷/테스트넷 등에 대한 접속, 스마트 컨트랙트 배포, 트랜잭션 전송 등 이더리움과 관련된 거의 모든 것을 할 수 있다. 이번 포스팅에서는 Geth를 설치하는 것부터 시작해서, 간단한 사용까지 다뤄보겠다.

## Installation
Geth의 설치에 대한 모든 내용은 [여기](https://geth.ethereum.org/docs/install-and-build/installing-geth)에 적혀있으니 참고하기 바란다.
여기에서는 필요한 것만 간단하게 소개한다.

이 포스팅의 테스트 환경은 Ubuntu 20.04이다.
Geth의 설치는 매우 간단하다. 쉘을 열고 아래 명령어를 차례로 입력하면, 큰 문제없이 설치가 가능하다.

~~~
$ sudo add-apt-repository -y ppa:ethereum/ethereum
$ sudo apt-get update
$ sudo apt-get install ethereum
~~~

Geth는 기본적으로 설치한 계정 디렉토리의 `.ethererum`을 기본 디렉토리로 사용한다.
개인적으로 메인넷에 접속하는 것보다는 연구에 필요한 테스트 환경을 구성하는데 Geth를 주로 사용하기 때문에 별로 사용할 일이 없었다.

## Görli 테스트넷 접속
이더리움은 (gas 비용이 어마무시한 메인넷 외에) 다양한 테스트넷이 운영되고 있다. 스마트 컨트랙트를 배포하기 전 테스트할 용도로 사용되기 때문에 당연히 테스트넷에서의 이더는 화폐로써 가치가 없다 (하지만, 생각보다 많은 양을 보유하기 힘들다). 대표적인 테스트넷과 사용되는 합의 알고리즘은 다음과 같다:

- Ropsten (Proof-of-Work)
- Görli (Proof-of-Authority)
- Kovan (Proof-of-Authority)
- Rinkeby (Proof-of-Authority)

많은 테스트넷들이 PoA 방식을 채택하고 있는데, 이 방식이 PoW에 비해 트랜잭션 처리량이 월등히 높기 때문이다.

### Initialize Clef
Clef는 일종의 지갑이나 계정 관리 서비스 정도로 생각하면 된다. 네트워크에 계정을 생성하고 관리할 수 있다. 우선, Clef를 사용하려면 다음과 같이 초기화를 해야 한다.

~~~
$ clef init
// master seceret 입력.
~~~

Clef 역시 Geth를 설치한 계정의 디렉토리 내에 `.clef`를 디폴트 디렉토리로 사용한다.
초기화를 한 후에는 다음과 같이 계정을 생성할 수 있다.

~~~
$ clef newaccount
// 계정의 패스워드를 입력.
~~~

위와 같은 방법으로 필요한 만큼 계정을 생성할 수 있다.

### Synchronize with network
이더리움의 메인넷이나 테스트넷에 접속하여 활동하려면, 우선 그동안 해당 네트워크에 생성된 모든 블록 정보를 동기화(라고 쓰고 다운로드라고 읽는다)를 해야 한다. 이 과정은 수준에 따라 엄청난 시간과 저장공간을 필요로 하게 된다. Geth에서 지원하는 sync 모드는 다음과 같다:

- full
- fast
- snap(default)
- light

여러 종류가 있지만, 쉽게 말해 full 모드를 제외하면 블록의 헤더 정보만 다운로드한다고 생각하면 된다. 만약 호기심에 full 모드로 네트워크를 동기화하면 하루 이상 블록정보를 다운로드하게 된다.

### Start Clef
앞서 생성한 계정 정보를 가지고 테스트넷에 접속하기 위해 Clef를 실행한다.

~~~
$ clef --keystore <GETH_DATA_DIR>/keystore --chainid 5
// Geth는 `.ethereum`을 기본 디렉토리로 사용하기 때문에 일반적으로 아래와 같은 형태로 사용하게 됨.
$ clef --keystore ~/.ethereum/keystore --chainid 5
~~~

위 명령어에서 `chainid`는 접속하려는 블록체인 네트워크의 번호이다. 5번은 Görli 테스트넷을 의미한다. 참고로 대표적으로 사용되는 `chainid`는 다음과 같다:

- 메인넷: 1
- Ropsten: 3
- Rinkeby: 4
- Görli: 5

### Start Geth
이제 Geth를 실행해서 테스트넷에 접속한다. 즉, 이더리움 클라이언트로 동작하는 과정이다. Clef가 실행 중인 상태로 아래 명령어를 실행한다.

~~~
$ geth --goerli --syncmode "light" --http --signer=<CLEF_LOCATION>/clef.ipc
// Clef는 `.clef`를 기본 디렉토리로 사용하기 때문에 일반적으로 아래와 같은 형태로 사용하게 됨.
$ geth --goerli --syncmode "light" --http --signer=~/.clef/clef.ipc
~~~

위 명령어를 해석해보면, Görli 테스트넷에 접속하는데, sync 모드는 `light`이고, 원격 접속을 허용(`--http`)하고 Clef와 통신은 IPC로 한다. 명령어 실행에 성공하면, `~/.ethereum/geth.ipc` 파일이 생성된다. 이 파일은 IPC를 이용해 Geth 콘솔에 접속해서 명령어를 실행하기 위해 사용된다. 또한, HTTP를 허용했기 때문에 같은 명령어를 원경에서 실행할 수 있는데, 원격 접속은 보안상의 이유로 IPC 방식에 비해 제약이 많다. 만약 원격에서 Geth가 지원하는 더 많은 기능을 사용하고 싶다면 이것저것 명시적으로 허용해야 한다. 이런 것들은 Geth를 실행할 때 명령어의 옵션으로 전달되며, Geth에서 사용할 수 있는 command line options는 [여기](https://geth.ethereum.org/docs/interface/command-line-options)에서 확인할 수 있다.

### Attach geth console
Geth가 성공적으로 실행되어 네트워크에 접속했다면, Geth console로 명령어를 전달할 수 있다. 여기에서 명령어는 네트워크에 기록된 모든 블록, 각 블록에 기록된 트랜잭션 등 이더리움에 대한 폭넓은 작업을 할 수 있다. 콘솔에 접속하기 위한 명령어는 다음과 같다.

~~~
$ geth attach <GETH_DATA_DIR>/geth.ipc
// Geth의 기본 디렉토리를 고려하면 일반적으로 아래와 같이 사용한다.
$ geth attach ~/.ethereum/get.ipc
~~~

콘솔에 접속하면, 자바스크립 코드를 실행할 수 있다. 예를 들어, 아래와 같은 함수를 실행하면 주어진 계정이 가진 잔액을 확인할 수 있다.
~~~
> web3.eth.getBalance('account address');
~~~

그 밖에도 web3.js의 모든 API를 지원하기 때문에 [여기](https://web3js.readthedocs.io/en/v1.7.0/)에서 필요한 함수를 검색해서 사용하기바란다.

## Dev network
아무리 테스트넷이라하더라도 실제로는 전세계에서 네트워크를 사용하고 있고, 많은 트랜잭션이 발생하기 때문에 스마트 컨트랙트의 초기 개발 단계에 적합하지 않을 수 있다. 개발 초기에는 오히려 마음대로 제어할 수 있고, 깨끗한 환경이 도움될 것이다. 이럴 때 사용할 수 있는 것이 Geth의 dev 네트워크이다.

dev 네트워크를 사용하려면, 우선, 네트워크에서 발생하는 블록 정보와 계정 정보 등을 저장할 디렉토리가 필요하다.

편의상 홈 디렉토리 밑에 `get-dev-chain` 디렉토리를 생성해서 진행해보겠다.
~~~
$ mkdir ~/geth-dev-chain
$ geth --datadir ~/geth-dev-chain --dev 
~~~

위와 같이 명령어를 실행하면, 간단하게 dev 네트워크를 동작시킬 수 있다. 만약 원격에서 이 네트워크에 접속을 허용하고 싶다면, `--http` 옵션을 추가하면 된다. 테스트넷에서와 마찬가지로 Geth 콘솔로 접속이 가능하다.

~~~
$ geth attach ~/geth-dev-chain/geth.ipc
~~~

dev 네트워크는 기본적으로 PoA 방식으로 동작하고, 처음 실행시키면, coinbase 계정이 1개 존재하고, 사용가능한 모든 이더를 보유하고 있다. 아래 명령어로 coinbase 계정의 주소와 보유한 이더의 양을 확인할 수 있다 (당연히 단위는 wei 이다).

~~~
> web3.eth.accounts
["0x485741e8dd34db6c90e191d6d619ce9a53e167ea"]
> web3.eth.getBalance('0x485741e8dd34db6c90e191d6d619ce9a53e167ea')
1.15792089237316195423570985008687907853269984665640564039457584007913129639927e+77
~~~

이 밖에도 선술한 것과 같이 Web3가 지원하는 많은 기술외에도 디버깅 등 유용한 작업을 해볼 수 있기 때문에 스마트 컨트랙트 초기 개발 단계에서 테스트 환경을 사용하기 적합하다.


## 마치며
이번 포스팅에서는 기본적인 Geth설치와 사용법에 대해 다뤘다. Geth는 이 글에 적힌 외에도 이더리움에 대한 정말 많은 기능을 지원하기 떄문에 이더리움 스마트 컨트랙트 개발자나 본인과 같은 연구자라면 반드시 이것저것 사용해보기 바란다. 원래 Web3를 이용해서 네트워크에 접속하거나 스마트 컨트랙트를 배포하고 트랜잭션을 발생시키는 것도 다뤄보려고 했으나, 포스팅의 취지에 어긋난다고 생각하여 적지 않았다. 이 내용에 대해서는 후속 포스팅에서 다뤄보도록 하겠다.

## References
- [https://geth.ethereum.org/docs/install-and-build/installing-geth](https://geth.ethereum.org/docs/install-and-build/installing-geth)
- [https://geth.ethereum.org/docs/interface/command-line-options](https://geth.ethereum.org/docs/interface/command-line-options)
- [https://web3js.readthedocs.io/en/v1.7.0/](https://web3js.readthedocs.io/en/v1.7.0/)
- [https://medium.com/mercuryprotocol/dev-highlights-of-this-week-cb33e58c745f](https://medium.com/mercuryprotocol/dev-highlights-of-this-week-cb33e58c745f)
- [https://medium.com/coinmonks/deploy-your-first-private-ethereum-smart-contract-using-geth-and-web3-js-2e495c1aadf4](https://medium.com/coinmonks/deploy-your-first-private-ethereum-smart-contract-using-geth-and-web3-js-2e495c1aadf4)
- [https://semode.tistory.com/314](https://semode.tistory.com/314)
- [https://medium.com/taipei-ethereum-meetup/beginners-guide-to-ethereum-3-explain-the-genesis-file-and-use-it-to-customize-your-blockchain-552eb6265145](https://medium.com/taipei-ethereum-meetup/beginners-guide-to-ethereum-3-explain-the-genesis-file-and-use-it-to-customize-your-blockchain-552eb6265145)
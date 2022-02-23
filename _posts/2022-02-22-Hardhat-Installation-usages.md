---
title: "Hardhat Installation과 기본 사용법"
author: Grizzly
categories:
  - Blockchain
  - Development environment
tags:
  - Hardhat

sitemap :
changefreq : daily
priority : 1.0
# table of contents
toc: true # 오른쪽 부분에 목차를 자동 생성해준다.
toc_label: "table of content" # toc 이름 설정
toc_icon: "bars" # 아이콘 설정
toc_sticky: true # 마우스 스크롤과 함께 내려갈 것인지 설정
---

## Hardhat
Hardhat은 이더리움 블록체인에서 스마트 컨트랙트를 개발하기 위한 프레임워크이다. 지금까지는 Truffle이 이 용도로 사용되어 왔지만, 개발자님들께서 많이 불편하셨나보다. 최근들어 Hardhat이 많이 사용되고 있고, 여기저기 연동되는 것도 점점 늘어나는 추세이다.

Hardhat은 자체적으로 스마트 컨트랙트 개발용 네트워크를 내장하고 있고, Solidity 언어로 작성된 스마트 컨트랙트를 테스트하고 배포하는 모든 사이클을 지원한다. 이번 포스팅은 Hardhat의 설치부터 기초적인 사용벙에 대해 다룬다.
Hardhat에 대한 보다 상세한 내용은 [공식 홈페이지](https://hardhat.org/)를 참조하기 바란다.

## Installation
Hardhat은 node.js 환경에서 동작하기 때문에 먼저 node.js를 설치해야 한다.
이 포스팅에서는 아래와 같이 node.js를 설치한다.

~~~
$ curl -fsSL https://deb.nodesource.com/setup_17.x | sudo -E bash -
$ sudo apt-get install -y nodejs
// Hardhat이 node.js의 버전에 의존적인데, 이 버전에 따라 일부 기능이 동작하지 않을 수 있음. 이럴 때는 대비해 아래와 같은 환경변수를 추가하면 유용하다.
$ export NODE_OPTIONS=--openssl-legacy-provider
$ npm install --save-dev hardhat
~~~

## Sample project
Hardhat이 설치되면, 손쉽게 Hardhat 프레임워크를 따르는 프로젝트를 생성할 수 있다. 어떤 디렉토리가 Hardhat 프로젝트로 인식되기 위해서는 반드시 해당 디렉토리 내에 `hardhat.config.js` 파일이 포함되어 있어야 한다. 즉, Hardhat 프로젝트를 생성하면, 이 파일과 함께 대략적인 프로젝트의 디렉토리 구조가 자동으로 생성된다.

### Create a sample project

~~~
$ npx hardhat
888    888                      888 888               888
888    888                      888 888               888
888    888                      888 888               888
8888888888  8888b.  888d888 .d88888 88888b.   8888b.  888888
888    888     "88b 888P"  d88" 888 888 "88b     "88b 888
888    888 .d888888 888    888  888 888  888 .d888888 888
888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
888    888 "Y888888 888     "Y88888 888  888 "Y888888  "Y888

Welcome to Hardhat v2.0.8

? What do you want to do? …
❯ Create a sample project
  Create an advanced sample project
  Create an advanced sample project that uses TypeScript
  Create an empty hardhat.config.js
  Quit
~~~

프로젝트를 생성할 때, 생성 수준을 선택할 수 있는데, 스마트 컨트랙트 개발에 익숙하지 않다면 `an advandced sample project`를 선택하는 것을 권장한다. 이 옵션으로 선택하면 이것저것 많은 파일들이 생성되는데, 이 파일들의 내용을 조금씩 수정하면서 실습하거나 개발하면 편리하다. 만약 어떤 디렉토리에 sample-proj라는 이름의프로젝트를 생성하면 그 구조는 아래와 같다.

~~~
$ tree ./ -L 1
./
├── node_modules
├── package.json
├── package-lock.json
└── sample-proj

2 directories, 2 files

$ tree ./sample-proj -L 3
./sample-proj
├── contracts
│   └── Greeter.sol
├── hardhat.config.js
├── README.md
├── scripts
│   └── deploy.js
└── test
    └── sample-test.js
~~~

`node_modules`는 기본적으로 설치된 의존성이 저장되는 디렉토리로써, node.js를 조금이라도 접해본적이 있다면 익숙할 것이다. 중요한 것은 역시 프로젝트 디렉토리 인데, 구조를 대략적으로 설명하면 다음과 같다.

- `contracts`: 이 프로젝트에서 작성한 스마트 컨트랙트가 위치한다.
- `hardhat.config.js`: 이 디렉토리를 Hardhat 프로젝트로 인식하게 하는 것 외에도 개발에 필요한 의존성, 초기 설정, 접속할 네트워크 정보 등이 적혀있다.
- `scripts`: 스마트 컨트랙트를 배포하기 위한 Javascript 파일이 위치한다.
- `test`: 작성한 스마트 컨트랙트를 배포하기 전, 구현한 함수들이 오류없이 동작하는지 테스트하기 위한 Javascript 파일이 위치한다.

프로젝트를 advanded 옵션으로 생성하면, 위와 같은 디렉토리 구조뿐만 아니라, 샘플 코드들이 자동으로 생성된다. `contracts`에 포함된 `Greeter.sol`은 일반적인 프로그램으로 치면 "Hello, world"를 출력하는 정도의 가장 단순한 스마트 컨트랙트이다.

### Test a sample contract
샘플 컨트랙트가 올바르게 동작하는지 확인해보자. `test`에는 스마트 컨트랙트를 테스트하기 위한 Javascript 파일이 포함되어 있는데, 이 파일은 Waffle 프레임워크로 작성되어 있다. 익숙해지면, 초기화->배포->동작확인 등과 같이 순치적으로 테스트를 진행해볼 수 있다. 여기에서는 디폴트로 생성된 `sample-test.js`를 실행하는 명령어만 살펴본다.

~~~
$ npx hardhat test


  Greeter
Deploying a Greeter with greeting: Hello, world!
Changing greeting from 'Hello, world!' to 'Hola, mundo!'
    ✓ Should return the new greeting once it's changed (357ms)


  1 passing (359ms)


~~~

에러가 없다면, 테스트하고자 하는 모든 기능이 정상적으로 동작하는 것을 의미한다. `sample-test.js`를 보면, 대략 어떻게 작성해야 하는지 쉽게 알 수 있기 때문에 이에 대한 설명은 생략하고, 보다 자세한 설명이 필요하다면 [공식 문서](https://hardhat.org/getting-started/)를 확인하기 바란다.

### Deploy a sample contract
이제 샘플 컨트랙트를 배포시켜보자. 컨트랙트를 배포하기 위해서는 먼저 테스트를 위한 블록체인 네트워크가 동작하고 있어야 한다. 앞서 Hardhat은 이를 위한 개발용 네트워크를 자체적으로 내장하고 있다고 말했는데, 아래와 같은 명령어로 동작시킬 수 있다.

~~~
$ npx hardhat node
Started HTTP and WebSocket JSON-RPC server at http://127.0.0.1:8545/

Accounts
========

WARNING: These accounts, and their private keys, are publicly known.
Any funds sent to them on Mainnet or any other live network WILL BE LOST.

Account #0: 0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266 (10000 ETH)
Private Key: 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

Account #1: 0x70997970c51812dc3a010c7d01b50e0d17dc79c8 (10000 ETH)
Private Key: 0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690d
...
~~~

이 명령어를 실행하면, 로컬 환경에 개발용 네트워크를 동작시키고, 디폴트로 10000 ETH를 가진 20개의 개정을 생성한다. 이 프로세스가 실행 중인 동안에는 개발용 네트워크를 이용할 수 있다. 

이제 스마트 컨트랙트를 배포해보자. 개발용 네트워크는 계속 실행되고 있어야 하기 때문에 다른 쉘을 실행한다. 앞서 컨트랙트의 배포는 scripts 디렉토리에 포함된 Javascript 파일이 담당한다고 했다. 이 파일을 이용해 아래와 같은 명령어를 실행해 개바용 네트워크에 컨트랙트를 배포한다.

~~~
$ npx hardhat run scripts/deploy.js --network localhost
Greeter deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
~~~

명령어 실행에 성공하면, 스마트 컨트랙트가 배포가되고, 개발용 네트워크를 실행한 쉘에서도 아래와 같은 메시지가 출력된다.

~~~
...
web3_clientVersion
eth_chainId
eth_accounts
eth_blockNumber
eth_chainId (2)
eth_estimateGas
eth_getBlockByNumber
eth_feeHistory
eth_sendTransaction
  Contract deployment: Greeter
  Contract address:    0xe7f1725e7734ce288f8367e1bb143e90bb3f0512
  Transaction:         0xf35ffcaaa97d66392192e1f2f422f9ddeb489771c86668ebc060142a95de569a
  From:                0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266
  Value:               0 ETH
  Gas used:            497026 of 497026
  Block #2:            0x0fe4cc9f42c0241553350c184b5e79ff4de791a1dc1a67f402e14794cd8ed6e9

  console.log:
    Deploying a Greeter with greeting: Hello, Hardhat!

eth_chainId
eth_getTransactionByHash
eth_chainId
eth_getTransactionReceipt
~~~

### Hardhat console
개발용이든 테스트넷이든 메인넷이든 이더리움 네트워크가 실행되고 있다면, 당연히 여기에 접속할 수 있어야 한다. Hardhat은 이런 네트워크에 접속할 수 있는 콘솔을 제공한다. 여기에서는 앞서 실행한 개발용 네트워크에 접속해본다.

~~~
$ npx hardhat console
Welcome to Node.js v12.22.7.
Type ".help" for more information.
>
~~~

이제 Hardhat에서 지원하는 많은 기능을 이용할 수 있다. 먼저, Hardhat Runtime Environment (HRE)를 임포트한다.

~~~
> const hardhat = require('hardhat');
> hardhat
Environment {
  config: {
    solidity: { compilers: [Array], overrides: {} },
    networks: { hardhat: [Object], localhost: [Object], ropsten: [Object] },
    gasReporter: { enabled: false, currency: 'USD' },
    etherscan: { apiKey: undefined },
    defaultNetwork: 'hardhat',
...
> hardhat.
hardhat.__defineGetter__            hardhat.__defineSetter__            hardhat.__lookupGetter__
hardhat.__lookupSetter__            hardhat.__proto__                   hardhat.hasOwnProperty
hardhat.isPrototypeOf               hardhat.propertyIsEnumerable        hardhat.toLocaleString
hardhat.toString                    hardhat.valueOf

hardhat._checkTypeValidation        hardhat._resolveArgument            hardhat._resolveValidTaskArguments
hardhat._runTaskDefinition          hardhat.constructor                 hardhat.injectToGlobal

hardhat._extenders                  hardhat.artifacts                   hardhat.config
hardhat.ethers                      hardhat.hardhatArguments            hardhat.network
hardhat.run                         hardhat.tasks                       hardhat.waffle
~~~

예를 들어, 개발용 네트워크에 생성된 계정을 확인하고 싶다면, 아래와 같은 명령어를 사용할 수 있다.

~~~
> let signers = await hardhat.getSigners()
> signers[0]
SignerWithAddress {
  _isSigner: true,
  address: '0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266',
  _signer: JsonRpcSigner {
    _isSigner: true,
    provider: EthersProviderWrapper {
      _isProvider: true,
      _events: [],
      _emitted: [Object],
...
~~~

## 마치며
이번 포스팅은 Hardhat의 설치부터 간단한 사용법 등을 살펴보았다. Hardhat이 지원하는 상세한 기능과 사용법은 공식 홈페이지를 통해 확인하기 바란다. 하지만, 대략 상기 내용을 숙지하고 있다면, 스마트 컨트랙트를 개발함에 있어 어떤 기능이 필요한지 생각해낼 수 있다.

## References
- [https://hardhat.org/](https://hardhat.org/)
- [https://hardhat.org/getting-started/](https://hardhat.org/getting-started/)
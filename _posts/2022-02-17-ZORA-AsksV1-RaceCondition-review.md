---
title: "ZORA AsksV1 Module Race Condition Review"
author: IceBear
categories:
  - Security 
  - Vulnerability
tags:
  - ZORA
  - NFT 
  - RaceCondition
sitemap :
changefreq : daily
priority : 1.0

# table of contents
toc: true # 오른쪽 부분에 목차를 자동 생성해준다.
toc_label: "table of content" # toc 이름 설정
toc_icon: "bars" # 아이콘 설정
toc_sticky: true # 마우스 스크롤과 함께 내려갈 것인지 설정
---

1월 24일 0x Protocol 팀이 ZORA의 AsksV1 모듈에 대한 Race Condition 취약점을 발견하여 제보하였다.
다행히 금전적인 피해는 없었고 ZORA 측에서 바로 패치하였다.

## ZORA 란?

[ZORA]란 NFT 마켓플레이스를 위한 프로토콜이다. 
개발자들이 쉽게 NFT 마켓플레이스를 제작할 수 있도록 다양한 기능을 제공한다.

그 중에 Asks 모듈이 있는데 이는 NFT 판매자가 특정 가격에 NFT를 판매할 수 있게 해주는 모듈이다.

## 취약점 분석

AsksV1 모듈은 사용자가 특정 가격에 판매 중인 NFT를 볼 수 있게 해주고, 구매하고자 하는 사용자가 
AsksV1 컨트랙트의 함수를 호출함으로써 거래도 가능하게 해준다.
해당 함수가 호출되면 ZORA 컨트랙트는 구매 금액을 구매자 계정으로부터 판매자 계정으로 전달해준다.
그리고 구매한 NFT를 판매자 계정에서 구매자 계정으로 전달한다.
사용하는 함수는 다음과 같다.

~~~
/// @notice Creates the ask for a given NFT
/// @param _tokenContract The address of the ERC-721 token to be sold
/// @param _tokenId The ID of the ERC-721 token to be sold
/// @param _askPrice The price to fill the ask
/// @param _askCurrency The address of the ERC-20 token required to fill, or address(0) for ETH
/// @param _sellerFundsRecipient The address to send funds once the ask is filled
/// @param _findersFeeBps The bps of the ask price (post-royalties) to be sent to the referrer of the sale
function createAsk(
   address _tokenContract,
   uint256 _tokenId,
   uint256 _askPrice,
   address _askCurrency,
   address _sellerFundsRecipient,
   uint16 _findersFeeBps
) external
~~~
~~~
/// @notice Fills the ask for a given NFT, transferring the ETH/ERC-20 to the seller and NFT to the buyer
/// @param _tokenContract The address of the ERC-721 token
/// @param _tokenId The ID of the ERC-721 token
/// @param _finder The address of the ask referrer
function fillAsk(
   address _tokenContract,
   uint256 _tokenId,
   address _finder
) external payable
~~~

사용자가 NFT 구매를 위해서는 ZORA가 자신 계정의 금액을 가져갈 수 있도록 허용해야 한다.
일반적인 패턴으로는 금액 제한없이 ZORA 컨트랙트에 의해 금액을 가져갈 수 있게 하는데, 왜냐하면 사용자가 
UI상에서 한번만 허용해주면 되기 때문에 유저 편의성을 위해서이다.

취약점은 바로 여기에 있는데, 구매자가 NFT를 구매하고자 할때 악의적인 판매자가 거래가 이루어지기 전에
구매자의 계정 잔액만큼 NFT의 가격을 높이게 되면(높은 가스비를 지불해서) 구매자의 계정 잔액 만큼을 NFT 구매에 
지불하게 할 수 있다.

## 취약점 패치

패치는 구매하고자 하는 금액(토큰 수량 및 토큰 종류)을 명시적으로 전달하는 방식으로 이루어졌다.
~~~
/// @notice Fills the ask for a given NFT, transferring the ETH/ERC-20 to the seller and NFT to the buyer
/// @param _tokenContract The address of the ERC-721 token
/// @param _tokenId The ID of the ERC-721 token
/// @param _fillCurrency The address of the ERC-20 token using to fill
/// @param _fillAmount The amount to fill the ask
/// @param _finder The address of the ask referrer
function fillAsk(
   address _tokenContract,
   uint256 _tokenId,
   address _fillCurrency,
   uint256 _fillAmount,
   address _finder
) external payable
~~~

## 참고

* https://zora.mirror.xyz/JeFZXnWb6jfJPon1rruXW-XJcoUVfgeNhu4XTYO3yFM

[ZORA]: https://zora.co

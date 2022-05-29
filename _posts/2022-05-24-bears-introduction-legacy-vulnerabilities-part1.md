---
title:  "The Introduction of Ethereum vulnerability Part1"
excerpt: "이더리움 네트워크에서 과거에 주로 발생했던 취약점 유형을 정리한 발표자료입니다."

author: Panda
categories:
  - Security
  - Smart Contract
  - Bears Team
tags:
  - Private Smart Contract
  - Solidity
  - Review
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2022-05-29 7:00:00 +09:00
lastmod: 2022-05-29 20:00:00 +09:00
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
이 자료는 BEARS 스터디 그룹에서 초기에 세미나 했던 Ethereum Solidity기반 취약점을 유형별로 정리한 자료입니다. 슬라이드와 해당 슬라이드에 대한 설명으로 구성할 계획이며, 슬라이드에 대한 설명은 틈틈히 작성하도록 하겠습니다.

# Presentation

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide1.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide2.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide3.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide4.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide5.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide6.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide7.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide8.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide9.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide10.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide11.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide12.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide13.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide14.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide15.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide16.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide17.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide18.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide19.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide20.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide21.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide22.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide23.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide24.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide25.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide26.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide27.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide28.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide29.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide30.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide31.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide32.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide33.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide34.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide35.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide36.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide38.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide39.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide40.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide41.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide42.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide43.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide44.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide45.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide46.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide47.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide48.png"| relative_url}})  |

# References
* [https://blog.sigmaprime.io/solidity-security.html](https://blog.sigmaprime.io/solidity-security.html)
* [https://dasp.co/](https://dasp.co/)
* [https://consensys.github.io/smart-contract-best-practices/known_attacks/#reentrancy](https://consensys.github.io/smart-contract-best-practices/known_attacks/#reentrancy)
* [https://consensys.github.io/smart-contract-best-practices/known_attacks/#reentrancy](https://consensys.github.io/smart-contract-best-practices/known_attacks/#reentrancy)
* [https://docs.soliditylang.org/en/latest/contracts.html?highlight=fallback#fallback-function](https://docs.soliditylang.org/en/latest/contracts.html?highlight=fallback#fallback-function)
* [https://docs.soliditylang.org/en/latest/contracts.html?highlight=fallback#fallback-function](https://docs.soliditylang.org/en/latest/contracts.html?highlight=fallback#fallback-function)
* [https://docs.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats](https://docs.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats)
* [https://docs.microsoft.com/en-us/windows-hardware/drivers/driversecurity/threat-modeling-for-drivers#the-dread-approach-to-threat-assessment](https://docs.microsoft.com/en-us/windows-hardware/drivers/driversecurity/threat-modeling-for-drivers#the-dread-approach-to-threat-assessment)
* [https://mixbytes.io/blog/collisions-solidity-storage-layouts](https://mixbytes.io/blog/collisions-solidity-storage-layouts)
* [https://medium.com/coinmonks/ethernaut-lvl-6-walkthrough-how-to-abuse-the-delicate-delegatecall-466b26c429e4](https://medium.com/coinmonks/ethernaut-lvl-6-walkthrough-how-to-abuse-the-delicate-delegatecall-466b26c429e4)
* [s://github.com/randao/randao](s://github.com/randao/randao)
* [https://swende.se/blog/Breaking_the_house.html](https://swende.se/blog/Breaking_the_house.html)
* [https://blog.positive.com/zeronights-ico-hacking-contest-writeup-63afb996f1e3](https://blog.positive.com/zeronights-ico-hacking-contest-writeup-63afb996f1e3)
* [https://blog.positive.com/predicting-random-numbers-in-ethereum-smart-contracts-e5358c6b8620](https://blog.positive.com/predicting-random-numbers-in-ethereum-smart-contracts-e5358c6b8620)
* [https://cryptomarketpool.com/front-running-a-solidity-smart-contract/](https://cryptomarketpool.com/front-running-a-solidity-smart-contract/)
* [https://medium.com/swlh/exploring-commit-reveal-schemes-on-ethereum-c4ff5a777db8](https://medium.com/swlh/exploring-commit-reveal-schemes-on-ethereum-c4ff5a777db8)
* [https://gus-tavo-guim.medium.com/reentrancy-attack-on-smart-contracts-how-to-identify-the-exploitable-and-an-example-of-an-attack-4470a2d8dfe4](https://gus-tavo-guim.medium.com/reentrancy-attack-on-smart-contracts-how-to-identify-the-exploitable-and-an-example-of-an-attack-4470a2d8dfe4)
* [tps://github.com/sraj50/unexpected-ether](tps://github.com/sraj50/unexpected-ether)
* [s://www.bookstack.cn/read/ethereumbook-en/spilt.5.c2a6b48ca6e1e33c.md](s://www.bookstack.cn/read/ethereumbook-en/spilt.5.c2a6b48ca6e1e33c.md)
* [https://medium.com/loom-network/how-to-secure-your-smart-contracts-6-solidity-vulnerabilities-and-how-to-avoid-them-part-1-c33048d4d17d](https://medium.com/loom-network/how-to-secure-your-smart-contracts-6-solidity-vulnerabilities-and-how-to-avoid-them-part-1-c33048d4d17d)
* [https://randomoracle.wordpress.com/2018/04/27/ethereum-solidity-and-integer-overflows-programming-blockchains-like-1970/](https://randomoracle.wordpress.com/2018/04/27/ethereum-solidity-and-integer-overflows-programming-blockchains-like-1970/)
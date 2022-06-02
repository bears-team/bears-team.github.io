---
title:  "The Introduction of Ethereum vulnerability Part3"
excerpt: "이더리움 네트워크에서 과거에 주로 발생했던 취약점 유형을 정리한 발표자료입니다."

author: IceBear
categories:
  - Security
  - Smart Contract
  - Bears Team
  - Polygon
tags:
  - Solidity
  - Immunefi
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
이 자료는 BEARS 스터디 그룹에서 IceBear님이 초기에 세미나 했던 Ethereum Solidity기반 취약점을 유형별로 정리한 자료입니다. 슬라이드와 해당 슬라이드에 대한 설명으로 구성할 계획이며, 슬라이드에 대한 설명은 틈틈히 작성하도록 하겠습니다.

# Presentation

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure1.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure2.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure3.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure4.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure5.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure6.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure7.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure8.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure9.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure10.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure11.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure12.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure13.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure14.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure15.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure16.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure17.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure18.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure19.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure20.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure21.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure22.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure23.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure24.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure25.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure26.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure27.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure28.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure29.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure30.png"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure31.png"| relative_url}})  |

<!--
| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part3/figure32.png"| relative_url}})  |
-->

# References
* [https://blog.sigmaprime.io/solidity-security.html](https://blog.sigmaprime.io/solidity-security.html)
* [https://docs.soliditylang.org/en/v0.8.9/](https://docs.soliditylang.org/en/v0.8.9/) 
* [https://ethereum.org/en/developers/docs/evm/](https://ethereum.org/en/developers/docs/evm/) 
* [https://takenobu-hs.github.io/downloads/ethereum_evm_illustrated.pdf](https://takenobu-hs.github.io/downloads/ethereum_evm_illustrated.pdf)
* [https://medium.com/immunefi/polygon-double-spend-bug-fix-postmortem-2m-bounty-5a1db09db7f1](https://medium.com/immunefi/polygon-double-spend-bug-fix-postmortem-2m-bounty-5a1db09db7f1)
* [https://gerhard-wagner.medium.com/double-spending-bug-in-polygons-plasma-bridge-2e0954ccadf1](https://gerhard-wagner.medium.com/double-spending-bug-in-polygons-plasma-bridge-2e0954ccadf1)

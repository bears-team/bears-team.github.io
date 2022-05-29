---
title:  "The Introduction of Ethereum vulnerability Part2"
excerpt: "이더리움 네트워크에서 과거에 주로 발생했던 취약점 유형을 정리한 발표자료입니다."

author:
  - Grizzly
  - Panda
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
#  ![Le logo de Jekyll]({{"/assets/images_posts/2021-09-23-cve-2021-31956-part1/1.PNG"| #relative_url}})
#{% endfigure %}
---
# Executive Summary
이 자료는 BEARS 스터디 그룹에서 Grizzly님이 초기에 세미나 했던 Ethereum Solidity기반 취약점을 유형별로 정리한 자료입니다. 슬라이드와 해당 슬라이드에 대한 설명으로 구성할 계획이며, 슬라이드에 대한 설명은 틈틈히 작성하도록 하겠습니다.

# Presentation

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part2/Slide1.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide2.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide3.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide4.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide5.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide6.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide7.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide8.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide9.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide10.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide11.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide12.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide13.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide14.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide15.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide16.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide17.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide18.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide19.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide20.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide21.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide22.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide23.PNG"| relative_url}})  |

| ![Image Alt 텍스트]({{"/assets/images_post/2022-05-24-bears-introduction-legacy-vulnerabilities-part1/Slide24.PNG"| relative_url}})  |


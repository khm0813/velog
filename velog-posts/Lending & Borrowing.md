<h3 id="defi-lending--borrowing-프로토콜-심층-분석"><strong>DeFi Lending &amp; Borrowing 프로토콜 심층 분석</strong></h3>
<ul>
<li><p>Lending &amp; Borrowing 프로토콜이란?
  •    블록체인에서 은행 역할을 하는 스마트컨트랙트
  •    Lending (대출해주기) = 내가 자산을 예치(supply) → 이자는 프로토콜이 줌
  •    Borrowing (빌리기) = 담보(Collateral)를 맡기고 다른 자산을 빌림 → 이자는 내가 냄
  즉, 유저는 예금주+차용인이 될 수 있고, 프로토콜은 자동으로 예치자와 차입자를 연결해 줌.
  대표 예: Aave, Compound 같은 DeFi 프로토콜</p>
<pre><code>    interface ILendingProtocol {
        // 담보로 자산 예치 (Deposit)
        function supply(address asset, uint amount) external;

        // 담보를 빼오기 (Withdraw)
        function withdraw(address asset, uint amount) external;

        // 자산 빌리기 (Borrow)
        function borrow(address asset, uint amount) external;

        // 빌린 자산 갚기 (Repay)
        function repay(address asset, uint amount) external;

        // 계정 상태 조회 (얼마 예치했고, 빌렸고, 청산 위험은?)
        function getAccountData(address user) external view returns (
            uint totalCollateralValue,
            uint totalDebtValue,
            uint healthFactor
        );
    }</code></pre><h4 id="중요한-개념">중요한 개념</h4>
</li>
</ul>
<ol>
<li>담보를 먼저 넣어야만 대출이 가능하다
→ 은행에서 담보 없이는 대출 불가한 것과 동일</li>
<li>Health Factor(HF)
•    프로토콜이 자동으로 관리하는 “안전 지표”
•    HF = (담보가치 × 담보인정비율) ÷ 대출금액
•    HF ≤ 1 → 청산</li>
<li>인터페이스 호출은 함수 호출과 같다
•    supply → 예금하기
•    borrow → 대출받기
•    repay → 대출 갚기
•    withdraw → 예금 빼기<pre><code>•    은행:
  •    돈을 맡기면 예금자 → 이자 받음
  •     담보 맡기고 대출 받으면 차용자 → 이자 냄
•    Lending 프로토콜:
  •    supply() = 예금 창구
  •    borrow() = 대출 창구
  •    repay() = 대출 상환 창구
  •    withdraw() = 예금 인출 창구
  •    getAccountData() = 내 통장 상태 확인 창구</code></pre></li>
</ol>
<blockquote>
<p>👉 정리하면, Lending &amp; Borrowing 인터페이스는 은행의 “입출금/대출 창구” 같은 함수들의 모음 HyperEVM 환경에서 호출해서 담보-차입-청산 위험 계산까지 연결하는 게 핵심 과제</p>
</blockquote>
<h4 id="defi-쪽에서-제일-많이-쓰이는-3대-대출-프로토콜">DeFi 쪽에서 제일 많이 쓰이는 3대 대출 프로토콜</h4>
<ol>
<li>Aave<blockquote>
<p>“은행 ATM+고급 대출 서비스”
단순히 빌리고 예치하는 것만이 아니라, 플래시 론(Flash Loan) 같은 특수 기능으로 유명함.</p>
</blockquote>
<ol>
<li>일반 대출
•    예: 1000 USDC를 예치 → ETH 차입</li>
<li>이자 농사(Leveraged Yield Farming)
•    담보 맡기고 다른 토큰 빌려서 파밍 포지션 확대</li>
<li>플래시 론
•    담보 없이 한 트랜잭션 내에서 빌리고 갚는 대출
•    예: 디파이 사이에서 아비트라지, 포지션 리밸런싱, 담보 교체 등
핵심 기술
•    플래시 론: Aave만의 시그니처. 담보 없이도 빌릴 수 있는데, 반드시 같은 블록 안에서 갚아야 함.
•    이자율 모델: Utilization(자금 사용률)에 따라 자동 조정.
→ 예: 대출 많이 일어나면 이자율 ↑, 여유자금 많으면 이자율 ↓
•    aToken: 예치하면 aUSDC, aETH 같은 토큰을 받음. → 자동으로 이자가 붙는 “예금증서”</li>
</ol>
</li>
<li>Compound<blockquote>
<p>“은행 예금+대출의 기본 모델”
디파이 대출 프로토콜의 시조격. UI/UX 단순하고 구조가 깔끔함.</p>
</blockquote>
<ol>
<li>기본 대출 서비스
•    예: USDC를 예치하고 DAI를 빌림</li>
<li>DeFi 원리 배우기
•    초급 개발자가 “대출 프로토콜이 어떻게 작동하는지” 공부하기 가장 좋은 구조</li>
<li>DeFi 통합 서비스의 인프라
•    다른 서비스들이 Compound 위에 얹어서 대출 기능을 제공
핵심 기술
•    cToken 구조
•    예치 → cUSDC, cETH 받음 → 시간이 지나면 교환비율이 올라가서 이자 수익 반영됨
•    이자율 곡선 모델
•    단순 Utilization 기반 모델 (Aave보다 단순)
•    Governance (COMP 토큰)
•    파라미터(이자율, 담보비율, 지원 토큰)를 거버넌스 투표로 결정</li>
</ol>
</li>
<li>Venus (특히 BNB Chain 기반)<blockquote>
<p>“BNB 체인의 은행”
Aave/Compound와 비슷하지만, <strong>BNB 체인(BSC)</strong>에 맞춰 최적화됨. BNB 체인에서 대출+스테이블코인 발행까지 담당.
핵심 기술
 •    vToken 구조</p>
<pre><code> •    Compound의 cToken 개념을 그대로 차용 (예: vBNB, vUSDC)</code></pre><p> •    BNB 체인 네이티브 최적화</p>
<pre><code> •    저가스, 빠른 트랜잭션 처리 → BNB 체인 사용자 친화</code></pre><p> •    리스크 관리 이슈</p>
<pre><code> •    과거 청산/가격 조작 이슈도 있었음 → 오라클·거버넌스 중요성 학습 포인트</code></pre></blockquote>
</li>
</ol>
<h3 id="lp-파밍--leverage-lp-파밍">LP 파밍 &amp; Leverage LP 파밍</h3>
<ol>
<li>Liquidity Pool (LP) 파밍
 •    AMM(Automated Market Maker) 풀에 두 가지 자산을 예치 → 풀에서 거래가 발생할 때마다 수수료 일부를 나눠 가짐.
 •    LP를 하면 →<pre><code> 1.    거래 수수료 (Swap fee)
 2.    프로토콜 보상 (예: 거버넌스 토큰 인센티브)</code></pre>→ 두 가지 수익을 얻음.
 •    예: USDC-ETH 풀에 $1000 넣으면, 사람들이 ETH↔USDC 교환할 때 수수료 일부가 내 몫으로 들어옴.</li>
<li>레버리지 (Leverage)
 •    원래 가진 돈보다 더 크게 투자하기 위해 돈을 빌려서 포지션을 키우는 것.
 •    예: 내가 $100을 담보로 맡기고, 프로토콜에서 $200을 빌려 총 $300 규모로 투자 → “3배 레버리지”.</li>
</ol>
<p>레버리지 LP 파밍이란?
👉 내 돈(담보) + 빌린 돈을 합쳐서 훨씬 큰 규모로 LP를 하는 것.
    •    담보: 보통 스테이블코인(USDC/USDT)이나 네이티브 토큰(ETH, BNB 등)
    •    빌린 돈: Lending 프로토콜에서 차입
    •    사용처: Uniswap v3, PancakeSwap v3 같은 AMM 풀</p>
<p>즉, 은행에서 돈 빌려서 농장에 씨앗을 더 뿌려, 더 많은 수확(수수료)을 얻는 방식</p>
<p> 왜 사용하는가?
    •    수익 극대화
        •    원래 $100만 넣었으면 10% 수익 = $10
        •    레버리지 3배(= $300 투자)면 같은 상황에서 $30 수익
    •    리스크도 커짐
        •    손실도 3배 확대됨
        •    가격이 움직여서 청산가에 닿으면 담보가 날아갈 수 있음</p>
<p>사용 방법 (실제 플로우)</p>
<p>예시: USDC를 가지고 있고, ETH-USDC 풀에 레버리지 LP 파밍을 하고 싶다
    1.    담보 예치 (Supply)
        •    HyperEVM의 Lending 프로토콜에 100 USDC 예치
    2.    자산 대출 (Borrow)
        •    예치 담보를 기반으로 ETH와 USDC를 빌림 (예: 100 USDC + 0.05 ETH)
    3.    풀에 예치 (Provide Liquidity)
        •    내가 가진 자산(자기자본 + 빌린 자산)을 합쳐서 ETH-USDC 풀에 넣음
    4.    수수료 및 보상 획득 (Farm)
        •    풀에서 거래가 발생할 때마다 수수료 발생
        •    프로토콜에서 추가로 보상 토큰을 줄 수도 있음
    5.    리스크 관리 (Risk Monitoring)
        •    내 담보 가치와 차입금액을 비교해 Health Factor(HF) 관리
        •    가격이 불리하게 움직이면 청산 위험
    6.    포지션 정리 (Close)
        •    LP에서 자산 인출 → 빌린 자산 상환 → 남은 것이 내 수익</p>
<p>핵심 계산 요소
    1.    Health Factor (HF)
        •    HF = (담보 가치 × 담보비율) ÷ 차입금
        •    HF ≤ 1 → 청산 당함
    2.    청산가 (Liq Price)
        •    자산 가격이 일정 수준 이하로 떨어지면 → 담보 강제 청산
    3.    Impermanent Loss (IL, 무상손실)
        •    LP 특유의 손실: 가격이 변동하면 “원래 그냥 들고 있었을 때보다 손해” 보는 경우 발생
        •    레버리지 시 IL도 배로 확대됨</p>
<p>장점 vs 단점</p>
<p>✅ 장점
    •    수익률 극대화 (특히 스테이블-스테이블 풀에서 안전하게 레버리지 파밍 많이 함)
    •    자산 효율을 극대화</p>
<p>❌ 단점
    •    청산 위험 (가격 변동이 담보보다 크면 담보 손실)
    •    Impermanent Loss 확대
    •    빌린 자산에 대한 이자 계속 발생 → 수익을 깎음</p>
<p>실제 사용자 경험 (UX 관점)
    •    사용자는 단순히 <strong>“몇 배 레버리지로 LP 파밍할래?”</strong>를 선택
    •    플랫폼이 자동으로:
        1.    담보 예치
        2.    자산 대출
        3.    적절히 스왑해서 풀에 예치
    •    이후, 플랫폼이 수익률/APY, 청산가, 알림 등을 보여줌</p>
<p>예시 (숫자로 보기)
    •    자기자본: 100 USDC
    •    레버리지: 3배 → 총 300 USDC 규모 LP
    •    풀 APR: 20% (연간 수수료 수익률)
    •    대출 이자: 8%</p>
<p>👉 계산
    •    총 수익: 300 × 20% = 60
    •    대출 이자: 200 × 8% = 16
    •    순이익: 60 - 16 = 44
    •    원래 자본대비 = 44% (vs 레버리지 없을 때 20%)</p>
<p>⚠️ 단, 가격이 흔들려 청산 오면 담보 100 USDC 날아감. </p>
<h3 id="uniswap--prjx">Uniswap &amp; PRJX</h3>
<p>🔹 Uniswap v3</p>
<p>개념
    •    이더리움 계열(ETH, Arbitrum, Optimism 등)에서 가장 많이 쓰이는 AMM(자동화 마켓메이커)
    •    v3는 Uniswap의 3세대 버전으로, 핵심 기능은 Concentrated Liquidity (집중 유동성)</p>
<p>핵심 기능
    •    가격 범위 지정 유동성 공급
→ LP가 “0.9 ~ 1.1달러 구간에서만 유동성 공급” 가능 → 자본 효율 대폭 ↑
    •    Non-fungible LP Position (NFT 기반 LP)
→ v2에서는 LP 토큰이 ERC20이었는데, v3에서는 각 LP 포지션이 “범위가 다르기 때문에 NFT”로 발행됨
    •    수수료 티어 선택
→ 풀 만들 때 0.05%, 0.3%, 1% 등 다양한 수수료율 선택 가능</p>
<p>생태계 위치
    •    Ethereum 메인넷 + L2(Arbitrum, Optimism, Polygon 등)에서 돌아가는 가장 큰 DEX(탈중앙 거래소)
    •    많은 프로토콜들이 Uniswap v3 풀을 기반으로 가격 피드/유동성을 활용</p>
<p>🔹 PRJX v3</p>
<p>개념
    •    HyperEVM 생태계 안에서 돌아가는 Uniswap v3와 유사한 구조의 AMM DEX
    •    즉, HyperEVM이란 별도 체인/생태계 안에서 “Uniswap v3 같은 서비스”를 제공하는 플랫폼이라고 보면 됨.</p>
<p>핵심 기능
    •    Uniswap v3 모델 차용
→ Concentrated Liquidity, LP NFT, 범위 지정 등 거의 동일한 컨셉
    •    HyperEVM 네이티브 통합
→ HyperEVM에서 발행되는 토큰(HL Core, HyperEVM Native Token) 중심으로 풀 구성
    •    HyperEVM Lending/파밍 서비스와 연계
→ PRJX v3 풀에서 레버리지 LP 파밍을 할 수 있도록 Lending 프로토콜과 바로 연결되도록 설계</p>
<p>생태계 위치
    •    HyperEVM DeFi 생태계의 핵심 DEX 역할
    •    Uniswap과 직접 연결되지 않고, 독립적으로 운영됨 (체인 자체가 다름)</p>
<p>비교 정리</p>
<p><img alt="" src="https://velog.velcdn.com/images/khm0813/post/ed9a6d36-4af2-43c0-a843-67e9b9376563/image.png" /></p>
<p>결론
    •    Uniswap v3 = 글로벌 표준 DEX (Ethereum/L2 기반)
    •    PRJX v3 = HyperEVM 체인 전용 DEX (Uniswap v3 모델을 따르지만, 별도 생태계)</p>
<p>👉 둘 다 <strong>“집중 유동성 AMM”</strong>이라는 같은 서비스를 제공하지만, 돌아가는 블록체인 생태계 자체가 달라서 서로 직접 연결되지는 않음.</p>
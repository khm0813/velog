<h3 id="hyperevm란">HyperEVM란?</h3>
<p>Hyperliquid가 만든 자체 EVM(이더리움 가상머신) 호환 체인입니다.</p>
<ul>
<li>이더리움과 같은 스마트컨트랙트 플랫폼이지만, 초고속, 초저비용에 최적화된 아키텍처를 자랑함</li>
<li>Hyperliquid DEX(거래소), Vault, TradingBot 등 온체인 거래의 모든 활동이 HyperEVM 위에서 동작</li>
</ul>
<p>⸻</p>
<h3 id="hyperevm의-핵심-목표">HyperEVM의 핵심 목표</h3>
<p>•    이더리움과 100% 호환 (Solidity, EVM 기반 도구, 지갑 모두 호환)하지만 이더리움 L1(메인넷)과 비교해 훨씬 빠르고, 수수료가 거의 0에 수렴
•    블록 생성/확정이 1<del>2초 내외 (일반적으로는 12초</del>1분이 걸리는 네트워크도 많음)
•    모든 온체인 거래·포지션 내역이 EVM 블록에 영구 저장</p>
<p>⸻</p>
<h3 id="hyperevm의-구조-dual-block-architecture-듀얼-블록-구조">HyperEVM의 구조: Dual-Block Architecture (듀얼 블록 구조)</h3>
<p>HyperEVM은 독특하게 두 가지 종류의 블록을 사용함</p>
<h4 id="l1-block">L1 Block</h4>
<blockquote>
<p>HyperEVM의 메인 블록
모든 온체인 거래(트레이드, Vault, 스테이킹, 등등)의 기록
EVM 호환 스마트컨트랙트 실행 및 외부 연동의 기준 블록</p>
</blockquote>
<h4 id="l2-block">L2 Block</h4>
<blockquote>
<p>사용자 체험/거래 속도를 극한으로 끌어올리기 위한 “빠른 미리보기 블록”
거래가 최종 확정되기 전, 사용자 경험(UX)을 위해 빠르게 미리 보여줌
트레이더 입장에서는 거의 실시간 거래 체감 가능!
단, 영구 기록은 반드시 L1에서만 확정</p>
</blockquote>
<p>일반적으로 Web3에서는 L1, L2라는 용어가 “메인체인/확장체인” 의미이지만
Hyperliquid의 Dual Block 구조는 <strong>L2(빠른 체감 반영) → L1(최종 확정)</strong>의 내부구조임</p>
<h3 id="hyperevm의-블록-동기화확정-플로우">HyperEVM의 블록 동기화/확정 플로우</h3>
<ol>
<li>유저가 거래(트레이드, Vault 입출금 등) 요청</li>
<li>즉시 L2 블록에 반영 → 유저는 즉시 체결된 것처럼 볼 수 있음</li>
<li>L2 블록이 L1 블록으로 Merge(확정)될 때, 실제 온체인 영구 기록이 됨
•    Hyperliquid 거래소의 모든 상태/잔고는 “L1 블록”에서 최종 결정</li>
</ol>
<p>⸻</p>
<h3 id="hyperevm에서-할-수-있는-일">HyperEVM에서 할 수 있는 일</h3>
<p>•    온체인 거래, Vault 입출금, TradingBot 자동매매 등
•    스마트컨트랙트 개발/배포(솔리디티/Foundry 등 지원)
•    메타마스크, ethers.js, web3.js 모두 지원
•    Hyperliquid의 TradingBot도 EVM 스마트컨트랙트 형태로 직접 온체인 트레이딩 가능
•    EVM 생태계의 모든 툴/서비스와 연동
•    예) 온체인 내역 분석, Dune Analytics, Etherscan(지원시), AI Bot 등</p>
<h4 id="실전-개발활용-포인트">실전 개발/활용 포인트</h4>
<p>•    HyperEVM 노드 RPC/엔드포인트
•    Metamask, web3.js, ethers.js 등 표준 방식 연결 가능
•    공식 docs에서 RPC URL, chainId 등 확인 후 바로 연결
•    스마트컨트랙트 배포/인터랙션
•    Foundry, Hardhat, Remix, MetaMask 등으로 이더리움과 동일하게 사용
•    컨트랙트 배포 시 별도 가스비 필요 없음(매우 저렴)
•    온체인 이벤트, 내역 분석
•    HyperEVM Explorer, Dune, custom indexer 등으로 블록별 데이터 추출 및 모니터링</p>
<h4 id="개발자가-꼭-알아둘-점-qa">개발자가 꼭 알아둘 점 (Q&amp;A)</h4>
<p>•    Q. 이더리움과 완전히 똑같이 개발해도 되나?
→ 거의 대부분 OK! (Solidity, ABI, RPC, 트랜잭션, 등등)
단, Hyperliquid만의 특화 기능(L2/L1 구조, 가스비, 이벤트 등)은 docs 참고
•    Q. 실제 자산(USDC 등)을 HyperEVM으로 브릿지/입출금 할 수 있나?
→ 예, Hyperliquid 거래소 내에서 입금/출금, Vault 이동 등 모두 HyperEVM 기반
•    Q. 거래 체감속도와 블록 확정속도가 다른 이유는?
→ Dual Block 구조(L2: 체감, L1: 확정) 덕분에
“즉시 체결” UX와 “온체인 영구기록”을 동시에 보장</p>
<h4 id="hyperevm-한눈에-이해">HyperEVM 한눈에 이해</h4>
<p>•    HyperEVM은 “온체인 초실시간 트레이딩”을 위한 Hyperliquid만의 EVM 호환 초고속 블록체인
•    모든 활동은 L2에서 즉시 처리, L1에서 확정 기록
•    스마트컨트랙트 개발/배포, 온체인 자동매매, 데이터 추출 등 기존 이더리움 경험이 있다면 “거의 그대로” 바로 개발/적용 가능</p>
<h3 id="스마트-컨트랙트smart-contract란">스마트 컨트랙트(Smart Contract)란?</h3>
<p>스마트 컨트랙트는 블록체인 위에서 자동으로 실행되는 디지털 계약입니다.
1990년대 후반, 니크 재보(Nick Szabo)가 처음 개념을 제시했으며, 본격적인 실체는 이더리움(Ethereum) 등 다양한 블록체인 플랫폼에서 구현되고 있습니다.</p>
<h3 id="스마트-컨트랙트의-정의">스마트 컨트랙트의 정의</h3>
<p>•    자동화된 계약: 특정 조건이 충족되면, 미리 정의된 명령(로직)을 자동 실행
•    탈중앙화: 중개자(은행, 변호사 등) 없이 신뢰를 확보
•    투명성/불변성: 모든 계약 조건과 이행 내역이 블록체인에 영구 기록
•    코드로 구현: 계약 조건이 프로그래밍 언어(대표적으로 Solidity)로 작성됨</p>
<blockquote>
<p>“계약 내용을 코드로 옮긴 다음, 조건을 충족하면 중개자 없이 스스로 실행되는 ‘로봇 계약’”</p>
</blockquote>
<p>⸻</p>
<h3 id="스마트-컨트랙트의-동작-원리">스마트 컨트랙트의 동작 원리</h3>
<ol>
<li>작성: 개발자가 계약의 논리를 스마트 컨트랙트 언어(Solidity, Vyper 등)로 코딩</li>
<li>배포: 블록체인(이더리움 등)에 배포(Deploy)</li>
<li>실행: 사용자 또는 외부 서비스가 트랜잭션을 발생시키면, 조건이 충족될 때마다 코드가 자동 실행</li>
<li>검증/기록: 모든 결과와 상태 변화가 블록체인에 영구 저장</li>
</ol>
<h3 id="스마트-컨트랙트의-실제-예시">스마트 컨트랙트의 실제 예시</h3>
<ol>
<li><p>토큰 발행
대표적으로 ERC-20(이더리움 표준) 토큰은 스마트 컨트랙트로 만들어집니다.</p>
<blockquote>
<ol>
<li>개발자가 MyToken이라는 스마트컨트랙트 배포</li>
<li>사용자가 토큰을 송금(transfer)할 때, 컨트랙트가 자동으로 거래 내역을 검증 및 기록</li>
<li>누구나 토큰 잔액, 송금 내역을 블록체인에서 확인 가능</li>
</ol>
</blockquote>
</li>
<li><p>자동 지급(escrow)
예를 들어, 두 사람이 NFT를 사고팔 때 “돈이 들어오면, NFT를 자동 이전”이라는 조건이 충족되어야만 거래가 완료</p>
</li>
<li><p>탈중앙화 거래소(DEX)
Uniswap, SushiSwap 등은 거래소의 모든 논리가 스마트컨트랙트로 구현
사용자가 DEX에 토큰을 예치/교환하면, 컨트랙트가 수수료, 교환 비율 등을 자동 계산 후 실행</p>
</li>
</ol>
<h3 id="실제-스마트컨트랙트-코드-예시-solidity">실제 스마트컨트랙트 코드 예시 (Solidity)</h3>
<p>아래는 이더리움 메인넷에서 동작 가능한 간단한 토큰 스마트컨트랙트 코드 예시입니다.
가장 기본적인 ERC-20 토큰 기능 일부만 추려 소개합니다.</p>
<pre><code>// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SimpleToken {
    string public name = &quot;SimpleToken&quot;;
    string public symbol = &quot;SIM&quot;;
    uint8 public decimals = 18;
    uint256 public totalSupply;

    mapping(address =&gt; uint256) public balanceOf;

    event Transfer(address indexed from, address indexed to, uint256 value);

    constructor(uint256 initialSupply) {
        totalSupply = initialSupply * 10 ** uint256(decimals);
        balanceOf[msg.sender] = totalSupply; // 계약 배포자가 모든 토큰 보유
    }

    function transfer(address to, uint256 value) public returns (bool success) {
        require(balanceOf[msg.sender] &gt;= value, &quot;잔액 부족&quot;);
        balanceOf[msg.sender] -= value;
        balanceOf[to] += value;
        emit Transfer(msg.sender, to, value);
        return true;
    }
}</code></pre><blockquote>
<p>totalSupply: 토큰 총 발행량
balanceOf: 주소별 토큰 보유량
transfer: 다른 주소로 토큰 송금
event Transfer: 블록체인에 송금 내역 기록</p>
</blockquote>
<h3 id="코드-배포-및-활용">코드 배포 및 활용</h3>
<p>이더리움 개발 환경(예: Remix IDE)에서 코드를 배포할 수 있습니다.
배포 후엔, 컨트랙트 주소를 통해 누구나 해당 토큰을 주고받을 수 있습니다.</p>
<h3 id="스마트-컨트랙트의-한계와-이슈">스마트 컨트랙트의 한계와 이슈</h3>
<p>코드 오류/취약점: 버그나 보안 취약점이 있으면, 해킹 피해 발생 가능 (대표적 예: The DAO 해킹 사건)
수정 불가: 블록체인에 올린 코드는 변경·삭제가 어려움
법적 문제: 법적 해석, 책임 소재가 불분명한 경우가 많음</p>
<p>⸻</p>
<h3 id="결론-및-전망">결론 및 전망</h3>
<p>스마트 컨트랙트는 “코드를 신뢰할 수 있다면, 사람이나 조직 없이도 자동으로 계약을 집행할 수 있다”는 혁신적인 기술입니다.
탈중앙화 금융(DeFi), NFT, DAO, 보험, 공급망 등 다양한 산업에서 스마트 컨트랙트가 활용되고 있습니다.</p>
<h1 id="abi와-스마트-컨트랙트-가이드">ABI와 스마트 컨트랙트 가이드</h1>
<h2 id="개요">개요</h2>
<p>이 문서는 Web3 생태계에서 ABI(Application Binary Interface)와 스마트 컨트랙트의 개념, 왜 필요한지, 그리고 HyperRange 프로젝트에서 어떻게 사용되는지에 대해 설명합니다. ABI는 dApp이 블록체인에 배포된 스마트 컨트랙트와 상호작용할 수 있게 해주는 핵심 기술입니다.</p>
<h2 id="목차">목차</h2>
<ol>
<li><a href="https://api.velog.io/rss/@khm0813#1-abi%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80">ABI란 무엇인가</a></li>
<li><a href="https://api.velog.io/rss/@khm0813#2-%EC%99%9C-abi%EA%B0%80-%ED%95%84%EC%9A%94%ED%95%9C%EA%B0%80">왜 ABI가 필요한가</a></li>
<li><a href="https://api.velog.io/rss/@khm0813#3-abi-%ED%8C%8C%EC%9D%BC%EC%9D%98-%EA%B5%AC%EC%A1%B0%EC%99%80-%ED%98%95%EC%8B%9D">ABI 파일의 구조와 형식</a></li>
<li><a href="https://api.velog.io/rss/@khm0813#4-hyperrange%EC%97%90%EC%84%9C%EC%9D%98-abi-%EC%82%AC%EC%9A%A9">실제 프로젝트에서의 ABI 사용</a></li>
<li><a href="https://api.velog.io/rss/@khm0813#5-%EC%8A%A4%EB%A7%88%ED%8A%B8-%EC%BB%A8%ED%8A%B8%EB%9E%99%ED%8A%B8-%EC%83%81%ED%98%B8%EC%9E%91%EC%9A%A9">스마트 컨트랙트 상호작용</a></li>
<li><a href="https://api.velog.io/rss/@khm0813#6-abi-%EA%B4%80%EB%A6%AC%EC%99%80-%EC%B5%9C%EC%A0%81%ED%99%94">ABI 관리와 최적화</a></li>
<li><a href="https://api.velog.io/rss/@khm0813#7-%EC%8B%A4%EC%A0%9C-%EC%82%AC%EC%9A%A9-%EC%98%88%EC%8B%9C">실제 사용 예시</a></li>
</ol>
<hr />
<h2 id="1-abi란-무엇인가">1. ABI란 무엇인가</h2>
<h3 id="기본-개념">기본 개념</h3>
<p><strong>ABI (Application Binary Interface)</strong>는 스마트 컨트랙트와 상호작용하기 위한 인터페이스 정의입니다. 마치 API 문서처럼, 컨트랙트가 어떤 함수를 가지고 있고, 어떤 매개변수를 받으며, 어떤 결과를 반환하는지 정의합니다.</p>
<h3 id="abi의-역할">ABI의 역할</h3>
<pre><code class="language-typescript">// ABI 없이는 이런 식으로 호출할 수 없음
const result = await contract.someFunction(param1, param2);

// ABI가 있어야 가능
const contract = new Contract(address, abi, provider);
const result = await contract.someFunction(param1, param2);</code></pre>
<h3 id="abi-vs-api">ABI vs API</h3>
<ul>
<li><strong>API</strong>: 웹 서비스와의 통신 인터페이스</li>
<li><strong>ABI</strong>: 스마트 컨트랙트와의 통신 인터페이스</li>
<li><strong>공통점</strong>: 둘 다 서로 다른 시스템 간의 통신 규약</li>
</ul>
<hr />
<h2 id="2-왜-abi가-필요한가">2. 왜 ABI가 필요한가</h2>
<h3 id="1-타입-안전성">1. 타입 안전성</h3>
<p>블록체인은 바이너리 데이터만 처리할 수 있어서, JavaScript/TypeScript의 타입 정보를 직접 전달할 수 없습니다.</p>
<pre><code class="language-typescript">// ❌ ABI 없이는 타입을 알 수 없음
const balance = await contract.balanceOf(&quot;0x123...&quot;); // 어떤 타입이 반환되는가?

// ✅ ABI가 있으면 타입을 정확히 알 수 있음
const balance: BigInt = await contract.balanceOf(&quot;0x123...&quot;);</code></pre>
<h3 id="2-함수-시그니처-매칭">2. 함수 시그니처 매칭</h3>
<p>스마트 컨트랙트의 함수는 해시로 식별되므로, ABI 없이는 어떤 함수를 호출할지 정확히 지정할 수 없습니다.</p>
<pre><code class="language-typescript">// 함수 시그니처: balanceOf(address)
// Keccak256 해시: 0x70a08231

// ABI 없이는 이 해시를 직접 사용해야 함
const result = await contract.call(&quot;0x70a08231&quot;, [address]);

// ABI가 있으면 함수명으로 직접 호출 가능
const result = await contract.balanceOf(address);</code></pre>
<h3 id="3-매개변수-인코딩디코딩">3. 매개변수 인코딩/디코딩</h3>
<p>블록체인은 모든 데이터를 바이너리로 처리하므로, JavaScript 객체를 바이너리로 변환하고, 결과를 다시 JavaScript 객체로 변환해야 합니다.</p>
<pre><code class="language-typescript">// ABI가 자동으로 처리해주는 것들:
// 1. address → 32바이트 바이너리
// 2. uint256 → 32바이트 바이너리  
// 3. string → 바이너리 + 길이
// 4. 배열 → 길이 + 각 요소</code></pre>
<h3 id="4-이벤트-구독">4. 이벤트 구독</h3>
<p>스마트 컨트랙트에서 발생하는 이벤트를 구독하고 파싱하기 위해 필요합니다.</p>
<pre><code class="language-typescript">// ABI 없이는 이벤트 데이터를 파싱할 수 없음
contract.on(&quot;Transfer&quot;, (from, to, amount) =&gt; {
  // from, to, amount가 어떤 타입인지 모름
});

// ABI가 있으면 타입을 정확히 알 수 있음
contract.on(&quot;Transfer&quot;, (from: string, to: string, amount: BigInt) =&gt; {
  // 타입 안전성 보장
});</code></pre>
<hr />
<h2 id="3-abi-파일의-구조와-형식">3. ABI 파일의 구조와 형식</h2>
<h3 id="abi-json-구조">ABI JSON 구조</h3>
<p>ABI는 JSON 배열 형태로 정의되며, 각 요소는 함수, 이벤트, 에러, 또는 생성자를 나타냅니다.</p>
<h4 id="함수-정의">함수 정의</h4>
<pre><code class="language-json">{
  &quot;inputs&quot;: [
    {
      &quot;internalType&quot;: &quot;address&quot;,
      &quot;name&quot;: &quot;spender&quot;,
      &quot;type&quot;: &quot;address&quot;
    },
    {
      &quot;internalType&quot;: &quot;uint256&quot;, 
      &quot;name&quot;: &quot;amount&quot;,
      &quot;type&quot;: &quot;uint256&quot;
    }
  ],
  &quot;name&quot;: &quot;approve&quot;,
  &quot;outputs&quot;: [
    {
      &quot;internalType&quot;: &quot;bool&quot;,
      &quot;name&quot;: &quot;&quot;,
      &quot;type&quot;: &quot;bool&quot;
    }
  ],
  &quot;stateMutability&quot;: &quot;nonpayable&quot;,
  &quot;type&quot;: &quot;function&quot;
}</code></pre>
<h4 id="이벤트-정의">이벤트 정의</h4>
<pre><code class="language-json">{
  &quot;anonymous&quot;: false,
  &quot;inputs&quot;: [
    {
      &quot;indexed&quot;: true,
      &quot;internalType&quot;: &quot;address&quot;,
      &quot;name&quot;: &quot;from&quot;,
      &quot;type&quot;: &quot;address&quot;
    },
    {
      &quot;indexed&quot;: true,
      &quot;internalType&quot;: &quot;address&quot;,
      &quot;name&quot;: &quot;to&quot;, 
      &quot;type&quot;: &quot;address&quot;
    },
    {
      &quot;indexed&quot;: false,
      &quot;internalType&quot;: &quot;uint256&quot;,
      &quot;name&quot;: &quot;amount&quot;,
      &quot;type&quot;: &quot;uint256&quot;
    }
  ],
  &quot;name&quot;: &quot;Transfer&quot;,
  &quot;type&quot;: &quot;event&quot;
}</code></pre>
<h3 id="주요-필드-설명">주요 필드 설명</h3>
<h4 id="inputsoutputs">inputs/outputs</h4>
<ul>
<li><strong>internalType</strong>: Solidity 타입 (address, uint256, string 등)</li>
<li><strong>name</strong>: 매개변수/반환값의 이름</li>
<li><strong>type</strong>: ABI 타입 (address, uint256, bytes 등)</li>
</ul>
<h4 id="statemutability">stateMutability</h4>
<ul>
<li><strong>pure</strong>: 상태를 읽지도 쓰지도 않음</li>
<li><strong>view</strong>: 상태를 읽기만 함 (가스 비용 없음)</li>
<li><strong>nonpayable</strong>: 상태를 변경하지만 가스 비용만 지불</li>
<li><strong>payable</strong>: ETH를 받을 수 있음</li>
</ul>
<h4 id="indexed-이벤트-전용">indexed (이벤트 전용)</h4>
<ul>
<li><strong>true</strong>: 이벤트 로그에서 검색 가능한 인덱스</li>
<li><strong>false</strong>: 검색 불가능한 데이터</li>
</ul>
<hr />
<h2 id="4-실제-프로젝트에서의-abi-사용">4. 실제 프로젝트에서의 ABI 사용</h2>
<h3 id="abi-파일-구조">ABI 파일 구조</h3>
<p>실제 프로젝트의 <code>abi/</code> 폴더에는 다음과 같은 파일들이 있습니다:</p>
<pre><code>abi/
├── contracts.ts          # 컨트랙트 주소와 설정
├── ERC20.json           # ERC20 토큰 표준 ABI
├── Factory.json         # Uniswap V3 Factory ABI
├── Pool.json            # Uniswap V3 Pool ABI
└── PositionManager.json # Uniswap V3 Position Manager ABI</code></pre><h3 id="contractsts---컨트랙트-주소-관리">contracts.ts - 컨트랙트 주소 관리</h3>
<pre><code class="language-typescript">export const CONTRACTS = {
  CHAIN_ID: 999,

  // 핵심 컨트랙트들
  POSITION_MANAGER: &quot;0xEaD19AE861c29bBb2101E834922B2FEee69B9091&quot;,
  FACTORY: &quot;0x0000000000000000000000000000000000000000&quot;,

  // 풀 정보
  POOLS: {
    HYPE_USDT0: {
      address: &quot;0x0000000000000000000000000000000000000000&quot;,
      token0: &quot;0x5555555555555555555555555555555555555555&quot;, // WHYPE
      token1: &quot;0xB8CE59FC3717ada4C02eaDF9682A9e934F625ebb&quot;, // USDT0
      fee: 3000, // 0.3%
      tickSpacing: 60
    }
  },

  // 토큰 정보
  TOKENS: {
    HYPE: {
      address: &quot;0x5555555555555555555555555555555555555555&quot;,
      symbol: &quot;HYPE&quot;,
      decimals: 18,
      name: &quot;Hyperliquid Token (Wrapped HYPE)&quot;
    }
  }
};</code></pre>
<h3 id="abi-파일들의-역할">ABI 파일들의 역할</h3>
<h4 id="erc20json">ERC20.json</h4>
<pre><code class="language-typescript">// 표준 ERC20 토큰 기능
- balanceOf(address): 잔액 조회
- approve(spender, amount): 승인
- transfer(to, amount): 전송
- allowance(owner, spender): 승인된 금액</code></pre>
<h4 id="factoryjson">Factory.json</h4>
<pre><code class="language-typescript">// 풀 생성 및 관리
- getPool(tokenA, tokenB, fee): 풀 주소 조회
- owner(): 팩토리 소유자 조회</code></pre>
<h4 id="positionmanagerjson">PositionManager.json</h4>
<pre><code class="language-typescript">// 유동성 포지션 관리
- mint(): 새 포지션 생성
- collect(): 수수료 수집
- burn(): 포지션 소각</code></pre>
<h3 id="왜-이런-구조가-필요한가">왜 이런 구조가 필요한가</h3>
<h4 id="1-모듈화">1. 모듈화</h4>
<ul>
<li><strong>표준 ABI</strong>: ERC20, ERC721 등 재사용 가능한 표준</li>
<li><strong>커스텀 ABI</strong>: 프로젝트 특화 기능</li>
<li><strong>설정 분리</strong>: 주소와 ABI를 별도로 관리</li>
</ul>
<h4 id="2-유지보수성">2. 유지보수성</h4>
<ul>
<li><strong>ABI 업데이트</strong>: 컨트랙트 변경 시 ABI만 수정</li>
<li><strong>주소 관리</strong>: 네트워크별 주소를 중앙에서 관리</li>
<li><strong>타입 안전성</strong>: TypeScript와 함께 사용하여 컴파일 타임 검증</li>
</ul>
<h4 id="3-확장성">3. 확장성</h4>
<ul>
<li><strong>새 컨트랙트 추가</strong>: ABI 파일만 추가하면 됨</li>
<li><strong>새 네트워크 지원</strong>: contracts.ts만 수정</li>
<li><strong>테스트 환경</strong>: 테스트넷 주소를 쉽게 변경</li>
</ul>
<hr />
<h2 id="5-스마트-컨트랙트-상호작용">5. 스마트 컨트랙트 상호작용</h2>
<h3 id="컨트랙트-인스턴스-생성">컨트랙트 인스턴스 생성</h3>
<pre><code class="language-typescript">import { Contract } from 'ethers';
import ERC20_ABI from './abi/ERC20.json';
import { CONTRACTS } from './abi/contracts';

// ERC20 토큰 컨트랙트 인스턴스 생성
const tokenContract = new Contract(
  CONTRACTS.TOKENS.HYPE.address,  // 컨트랙트 주소
  ERC20_ABI,                      // ABI
  provider                         // Provider (읽기 전용)
);

// 또는 서명자와 함께 (쓰기 가능)
const tokenContractWithSigner = new Contract(
  CONTRACTS.TOKENS.HYPE.address,
  ERC20_ABI,
  signer                          // Signer (쓰기 가능)
);</code></pre>
<h3 id="읽기-전용-함수-호출">읽기 전용 함수 호출</h3>
<pre><code class="language-typescript">// 잔액 조회
const balance = await tokenContract.balanceOf(userAddress);
console.log('잔액:', ethers.formatEther(balance), 'HYPE');

// 토큰 정보 조회
const name = await tokenContract.name();
const symbol = await tokenContract.symbol();
const decimals = await tokenContract.decimals();

// 승인된 금액 조회
const allowance = await tokenContract.allowance(owner, spender);</code></pre>
<h3 id="상태-변경-함수-호출">상태 변경 함수 호출</h3>
<pre><code class="language-typescript">// 토큰 승인
const approveTx = await tokenContractWithSigner.approve(
  spenderAddress,
  ethers.parseEther(&quot;100&quot;)
);

// 트랜잭션 완료 대기
const receipt = await approveTx.wait();
console.log('승인 완료:', receipt.transactionHash);

// 토큰 전송
const transferTx = await tokenContractWithSigner.transfer(
  recipientAddress,
  ethers.parseEther(&quot;10&quot;)
);</code></pre>
<h3 id="이벤트-구독">이벤트 구독</h3>
<pre><code class="language-typescript">// Transfer 이벤트 구독
tokenContract.on(&quot;Transfer&quot;, (from, to, amount, event) =&gt; {
  console.log(`${from}에서 ${to}로 ${ethers.formatEther(amount)} 전송`);
  console.log('트랜잭션 해시:', event.transactionHash);
});

// 특정 이벤트만 구독
tokenContract.on(&quot;Approval&quot;, (owner, spender, amount) =&gt; {
  console.log(`${owner}가 ${spender}에게 ${ethers.formatEther(amount)} 승인`);
});

// 이벤트 구독 해제
tokenContract.off(&quot;Transfer&quot;);</code></pre>
<h3 id="에러-처리">에러 처리</h3>
<pre><code class="language-typescript">try {
  const balance = await tokenContract.balanceOf(invalidAddress);
} catch (error) {
  if (error.code === 'CALL_EXCEPTION') {
    console.error('컨트랙트 호출 실패:', error.message);
  } else if (error.code === 'INSUFFICIENT_FUNDS') {
    console.error('가스비 부족');
  } else {
    console.error('알 수 없는 오류:', error);
  }
}</code></pre>
<hr />
<h2 id="6-abi-관리와-최적화">6. ABI 관리와 최적화</h2>
<h3 id="abi-최적화-전략">ABI 최적화 전략</h3>
<h4 id="1-필요한-함수만-포함">1. 필요한 함수만 포함</h4>
<pre><code class="language-typescript">// ❌ 전체 ABI 사용 (용량 증가)
import FULL_ABI from './abi/FullContract.json';

// ✅ 필요한 함수만 포함한 최적화된 ABI
const OPTIMIZED_ABI = [
  {
    &quot;inputs&quot;: [{&quot;internalType&quot;: &quot;address&quot;, &quot;name&quot;: &quot;account&quot;, &quot;type&quot;: &quot;address&quot;}],
    &quot;name&quot;: &quot;balanceOf&quot;,
    &quot;outputs&quot;: [{&quot;internalType&quot;: &quot;uint256&quot;, &quot;name&quot;: &quot;&quot;, &quot;type&quot;: &quot;uint256&quot;}],
    &quot;stateMutability&quot;: &quot;view&quot;,
    &quot;type&quot;: &quot;function&quot;
  }
];</code></pre>
<h4 id="2-abi-번들링">2. ABI 번들링</h4>
<pre><code class="language-typescript">// 여러 ABI를 하나로 묶기
export const COMBINED_ABI = {
  ERC20: ERC20_ABI,
  FACTORY: FACTORY_ABI,
  POOL: POOL_ABI,
  POSITION_MANAGER: POSITION_MANAGER_ABI
};

// 사용 시
const contract = new Contract(address, COMBINED_ABI.ERC20, provider);</code></pre>
<h4 id="3-동적-abi-로딩">3. 동적 ABI 로딩</h4>
<pre><code class="language-typescript">// 필요한 ABI만 동적으로 로드
async function loadABI(contractName: string) {
  const abi = await import(`./abi/${contractName}.json`);
  return abi.default;
}

// 사용 시
const erc20ABI = await loadABI('ERC20');
const contract = new Contract(address, erc20ABI, provider);</code></pre>
<h3 id="abi-검증과-테스트">ABI 검증과 테스트</h3>
<h4 id="1-abi-유효성-검사">1. ABI 유효성 검사</h4>
<pre><code class="language-typescript">function validateABI(abi: any[]): boolean {
  if (!Array.isArray(abi)) return false;

  return abi.every(item =&gt; {
    if (item.type === 'function') {
      return item.name &amp;&amp; item.inputs &amp;&amp; item.outputs;
    } else if (item.type === 'event') {
      return item.name &amp;&amp; item.inputs;
    }
    return true;
  });
}

// 사용 시
if (!validateABI(ERC20_ABI)) {
  throw new Error('잘못된 ABI 형식');
}</code></pre>
<h4 id="2-컨트랙트-연결-테스트">2. 컨트랙트 연결 테스트</h4>
<pre><code class="language-typescript">async function testContractConnection(
  address: string, 
  abi: any[], 
  provider: Provider
): Promise&lt;boolean&gt; {
  try {
    const contract = new Contract(address, abi, provider);

    // 간단한 함수 호출로 연결 테스트
    if (abi.some(item =&gt; item.name === 'name')) {
      await contract.name();
    } else if (abi.some(item =&gt; item.name === 'balanceOf')) {
      await contract.balanceOf(ethers.ZeroAddress);
    }

    return true;
  } catch (error) {
    console.error('컨트랙트 연결 실패:', error);
    return false;
  }
}</code></pre>
<h3 id="abi-버전-관리">ABI 버전 관리</h3>
<h4 id="1-버전별-abi-관리">1. 버전별 ABI 관리</h4>
<pre><code class="language-typescript">// 버전별 ABI 관리
export const ABI_VERSIONS = {
  v1: {
    ERC20: ERC20_V1_ABI,
    FACTORY: FACTORY_V1_ABI
  },
  v2: {
    ERC20: ERC20_V2_ABI,
    FACTORY: FACTORY_V2_ABI
  }
};

// 사용 시
const abi = ABI_VERSIONS.v2.ERC20;</code></pre>
<h4 id="2-abi-마이그레이션">2. ABI 마이그레이션</h4>
<pre><code class="language-typescript">// ABI 변경 시 자동 마이그레이션
async function migrateABI(oldABI: any[], newABI: any[]) {
  const changes = detectABIChanges(oldABI, newABI);

  if (changes.breaking) {
    console.warn('Breaking changes detected:', changes);
    // 사용자에게 알림 또는 자동 처리
  }

  return newABI;
}</code></pre>
<hr />
<h2 id="7-실제-사용-예시">7. 실제 사용 예시</h2>
<h4 id="1-토큰-잔액-조회">1. 토큰 잔액 조회</h4>
<pre><code class="language-typescript">// ConnectButton 컴포넌트에서
const { data: hyPEBalance } = useBalance({
  address: address,
  query: { enabled: isConnected &amp;&amp; isHyperEVM(chainId) }
});

// 내부적으로 ERC20 ABI를 사용하여 balanceOf 호출
// eth_call(contract_address, &quot;0x70a08231&quot; + address_padded)</code></pre>
<h4 id="2-풀-정보-조회">2. 풀 정보 조회</h4>
<pre><code class="language-typescript">// Factory 컨트랙트를 통한 풀 주소 조회
const factoryContract = new Contract(
  CONTRACTS.FACTORY,
  FACTORY_ABI,
  provider
);

const poolAddress = await factoryContract.getPool(
  CONTRACTS.TOKENS.HYPE.address,
  CONTRACTS.TOKENS.USDT0.address,
  3000 // 0.3% 수수료
);</code></pre>
<h4 id="3-포지션-생성">3. 포지션 생성</h4>
<pre><code class="language-typescript">// PositionManager를 통한 새 포지션 생성
const positionManager = new Contract(
  CONTRACTS.POSITION_MANAGER,
  POSITION_MANAGER_ABI,
  signer
);

const mintTx = await positionManager.mint({
  token0: CONTRACTS.TOKENS.HYPE.address,
  token1: CONTRACTS.TOKENS.USDT0.address,
  fee: 3000,
  tickLower: -1000,
  tickUpper: 1000,
  amount0Desired: ethers.parseEther(&quot;100&quot;),
  amount1Desired: ethers.parseEther(&quot;100&quot;),
  amount0Min: 0,
  amount1Min: 0,
  recipient: userAddress,
  deadline: Math.floor(Date.now() / 1000) + 1800 // 30분
});</code></pre>
<h3 id="일반적인-abi-사용-패턴">일반적인 ABI 사용 패턴</h3>
<h4 id="1-컨트랙트-팩토리-패턴">1. 컨트랙트 팩토리 패턴</h4>
<pre><code class="language-typescript">class ContractFactory {
  private static contracts = new Map();

  static getContract(address: string, abi: any[], provider: Provider) {
    const key = `${address}_${JSON.stringify(abi)}`;

    if (!this.contracts.has(key)) {
      this.contracts.set(key, new Contract(address, abi, provider));
    }

    return this.contracts.get(key);
  }
}

// 사용 시
const tokenContract = ContractFactory.getContract(
  tokenAddress,
  ERC20_ABI,
  provider
);</code></pre>
<h4 id="2-abi-캐싱">2. ABI 캐싱</h4>
<pre><code class="language-typescript">// ABI를 메모리에 캐싱
const abiCache = new Map();

async function getCachedABI(contractName: string) {
  if (abiCache.has(contractName)) {
    return abiCache.get(contractName);
  }

  const abi = await import(`./abi/${contractName}.json`);
  abiCache.set(contractName, abi.default);

  return abi.default;
}</code></pre>
<h4 id="3-에러-처리와-재시도">3. 에러 처리와 재시도</h4>
<pre><code class="language-typescript">async function callWithRetry(
  contract: Contract,
  method: string,
  params: any[],
  maxRetries: number = 3
) {
  for (let i = 0; i &lt; maxRetries; i++) {
    try {
      return await contract[method](...params);
    } catch (error) {
      if (i === maxRetries - 1) throw error;

      // 일시적인 오류인 경우 잠시 대기 후 재시도
      if (error.code === 'NETWORK_ERROR') {
        await new Promise(resolve =&gt; setTimeout(resolve, 1000 * (i + 1)));
        continue;
      }

      throw error;
    }
  }
}</code></pre>
<hr />
<h2 id="결론">결론</h2>
<p>ABI는 Web3 생태계에서 스마트 컨트랙트와 상호작용하기 위한 필수적인 인터페이스입니다.</p>
<h3 id="핵심-포인트">핵심 포인트</h3>
<ol>
<li><strong>타입 안전성</strong>: JavaScript와 블록체인 간의 타입 변환 보장</li>
<li><strong>함수 시그니처</strong>: 스마트 컨트랙트 함수를 쉽게 호출</li>
<li><strong>이벤트 처리</strong>: 블록체인 이벤트를 실시간으로 구독하고 파싱</li>
<li><strong>모듈화</strong>: 재사용 가능한 ABI 구조로 유지보수성 향상</li>
<li><strong>최적화</strong>: 필요한 함수만 포함하여 성능 향상</li>
</ol>
<h3 id="hyperrange에서의-활용">HyperRange에서의 활용</h3>
<ul>
<li><strong>ERC20 표준</strong>: 토큰 잔액 조회, 승인, 전송</li>
<li><strong>Uniswap V3</strong>: 풀 생성, 유동성 제공, 포지션 관리</li>
<li><strong>컨트랙트 주소 관리</strong>: 중앙화된 주소 관리로 유지보수성 향상</li>
</ul>
<h3 id="다음-단계">다음 단계</h3>
<p>ABI에 대한 이해를 바탕으로 다음 주제들을 학습해보세요:</p>
<ul>
<li><strong>스마트 컨트랙트 배포</strong>: ABI 생성과 배포 과정</li>
<li><strong>가스 최적화</strong>: ABI를 통한 효율적인 함수 호출</li>
<li><strong>이벤트 인덱싱</strong>: 대량 이벤트 데이터 처리</li>
<li><strong>ABI 자동 생성</strong>: 컴파일러를 통한 ABI 자동 생성</li>
</ul>
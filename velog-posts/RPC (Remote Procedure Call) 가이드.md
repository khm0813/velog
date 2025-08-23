<h1 id="rpc-remote-procedure-call-가이드">RPC (Remote Procedure Call) 가이드</h1>
<h2 id="개요">개요</h2>
<p>이 문서는 Web3 생태계에서 RPC(Remote Procedure Call)의 개념, 작동 원리, 그리고 프로젝트에서 어떻게 사용되는지에 대해 설명합니다. RPC는 블록체인과 상호작용하는 핵심 기술로, dApp이 블록체인 네트워크와 통신할 수 있게 해줍니다.</p>
<h2 id="목차">목차</h2>
<ol>
<li><a href="https://api.velog.io/rss/@khm0813#1-rpc%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80">RPC란 무엇인가</a></li>
<li><a href="https://api.velog.io/rss/@khm0813#2-web3%EC%97%90%EC%84%9C-rpc%EC%9D%98-%EC%97%AD%ED%95%A0">Web3에서 RPC의 역할</a></li>
<li><a href="https://api.velog.io/rss/@khm0813#3-rpc-%ED%86%B5%EC%8B%A0-%EB%B0%A9%EC%8B%9D">RPC 통신 방식</a></li>
<li><a href="https://api.velog.io/rss/@khm0813#4-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%EC%97%90%EC%84%9C%EC%9D%98-rpc-%EC%82%AC%EC%9A%A9">프로젝트에서의 RPC 사용</a></li>
<li><a href="https://api.velog.io/rss/@khm0813#5-rpc-%EC%A0%9C%EA%B3%B5%EC%9E%90-%EC%A2%85%EB%A5%98">RPC 제공자 종류</a></li>
<li><a href="https://api.velog.io/rss/@khm0813#6-rpc-%EB%B3%B4%EC%95%88%EA%B3%BC-%EC%B5%9C%EC%A0%81%ED%99%94">RPC 보안과 최적화</a></li>
<li><a href="https://api.velog.io/rss/@khm0813#7-%EC%8B%A4%EC%A0%9C-%EC%82%AC%EC%9A%A9-%EC%98%88%EC%8B%9C">실제 사용 예시</a></li>
</ol>
<hr />
<h2 id="1-rpc란-무엇인가">1. RPC란 무엇인가</h2>
<h3 id="기본-개념">기본 개념</h3>
<p><strong>RPC (Remote Procedure Call)</strong>는 한 프로그램이 다른 프로그램의 프로시저(함수)를 원격으로 호출할 수 있게 해주는 기술입니다. 마치 로컬 함수를 호출하는 것처럼 원격 서버의 함수를 실행할 수 있습니다.</p>
<h3 id="전통적인-rpc-vs-web3-rpc">전통적인 RPC vs Web3 RPC</h3>
<h4 id="전통적인-rpc">전통적인 RPC</h4>
<pre><code class="language-typescript">// 로컬 함수 호출
function localFunction() {
  return &quot;Hello World&quot;;
}

// 전통적인 RPC 호출
const result = await rpcClient.call(&quot;remoteFunction&quot;);</code></pre>
<h4 id="web3-rpc">Web3 RPC</h4>
<pre><code class="language-typescript">// 블록체인 상태 조회
const balance = await ethereum.request({
  method: 'eth_getBalance',
  params: [address, 'latest']
});

// 트랜잭션 전송
const txHash = await ethereum.request({
  method: 'eth_sendTransaction',
  params: [transactionObject]
});</code></pre>
<h3 id="왜-rpc가-필요한가">왜 RPC가 필요한가</h3>
<ul>
<li><strong>분산 시스템</strong>: 블록체인은 분산된 네트워크로, 직접 접근이 불가능</li>
<li><strong>상태 조회</strong>: 잔액, 블록 정보, 스마트 컨트랙트 상태 등을 조회</li>
<li><strong>트랜잭션 전송</strong>: 새로운 트랜잭션을 네트워크에 전파</li>
<li><strong>이벤트 구독</strong>: 블록체인 이벤트를 실시간으로 수신</li>
</ul>
<hr />
<h2 id="2-web3에서-rpc의-역할">2. Web3에서 RPC의 역할</h2>
<h3 id="블록체인과의-통신-다리">블록체인과의 통신 다리</h3>
<p>RPC는 dApp과 블록체인 네트워크 사이의 <strong>통신 다리</strong> 역할을 합니다.</p>
<pre><code class="language-mermaid">graph LR
    A[dApp] --&gt; B[RPC Node]
    B --&gt; C[블록체인 네트워크]
    C --&gt; B
    B --&gt; A</code></pre>
<h3 id="주요-기능들">주요 기능들</h3>
<h4 id="1-상태-조회-read-operations">1. 상태 조회 (Read Operations)</h4>
<pre><code class="language-typescript">// 계정 잔액 조회
const balance = await provider.getBalance(address);

// 블록 정보 조회
const block = await provider.getBlock(blockNumber);

// 스마트 컨트랙트 상태 조회
const contract = new Contract(address, abi, provider);
const result = await contract.someFunction();</code></pre>
<h4 id="2-트랜잭션-전송-write-operations">2. 트랜잭션 전송 (Write Operations)</h4>
<pre><code class="language-typescript">// 트랜잭션 전송
const tx = await wallet.sendTransaction({
  to: recipientAddress,
  value: ethers.parseEther(&quot;1.0&quot;)
});

// 스마트 컨트랙트 함수 호출
const tx = await contract.someFunction(params);</code></pre>
<h4 id="3-이벤트-구독-event-listening">3. 이벤트 구독 (Event Listening)</h4>
<pre><code class="language-typescript">// 이벤트 구독
contract.on(&quot;Transfer&quot;, (from, to, amount) =&gt; {
  console.log(`${from}에서 ${to}로 ${amount} 전송`);
});

// 블록 이벤트 구독
provider.on(&quot;block&quot;, (blockNumber) =&gt; {
  console.log(`새 블록 생성: ${blockNumber}`);
});</code></pre>
<hr />
<h2 id="3-rpc-통신-방식">3. RPC 통신 방식</h2>
<h3 id="http-rpc">HTTP RPC</h3>
<p>가장 일반적인 방식으로, HTTP 요청을 통해 블록체인과 통신합니다.</p>
<pre><code class="language-typescript">// HTTP POST 요청 예시
const response = await fetch('https://rpc.hyperliquid.xyz/evm', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    jsonrpc: '2.0',
    method: 'eth_getBalance',
    params: [address, 'latest'],
    id: 1
  })
});

const result = await response.json();</code></pre>
<h3 id="websocket-rpc">WebSocket RPC</h3>
<p>실시간 통신이 필요한 경우 사용하는 양방향 통신 방식입니다.</p>
<pre><code class="language-typescript">// WebSocket 연결
const ws = new WebSocket('wss://rpc.hyperliquid.xyz/evm');

// 메시지 전송
ws.send(JSON.stringify({
  jsonrpc: '2.0',
  method: 'eth_subscribe',
  params: ['newHeads'],
  id: 1
}));

// 메시지 수신
ws.onmessage = (event) =&gt; {
  const data = JSON.parse(event.data);
  console.log('새 블록:', data);
};</code></pre>
<h3 id="json-rpc-20-표준">JSON-RPC 2.0 표준</h3>
<p>Web3 RPC는 JSON-RPC 2.0 표준을 따릅니다.</p>
<pre><code class="language-typescript">// 요청 형식
{
  &quot;jsonrpc&quot;: &quot;2.0&quot;,
  &quot;method&quot;: &quot;method_name&quot;,
  &quot;params&quot;: [param1, param2],
  &quot;id&quot;: 1
}

// 응답 형식
{
  &quot;jsonrpc&quot;: &quot;2.0&quot;,
  &quot;result&quot;: &quot;result_data&quot;,
  &quot;id&quot;: 1
}

// 에러 응답
{
  &quot;jsonrpc&quot;: &quot;2.0&quot;,
  &quot;error&quot;: {
    &quot;code&quot;: -32601,
    &quot;message&quot;: &quot;Method not found&quot;
  },
  &quot;id&quot;: 1
}</code></pre>
<hr />
<h2 id="4-프로젝트에서의-rpc-사용">4. 프로젝트에서의 RPC 사용</h2>
<h3 id="wagmi-설정에서의-rpc">Wagmi 설정에서의 RPC</h3>
<p><code>apps/web/src/lib/wagmi.ts</code> 파일에서 RPC 설정을 확인할 수 있습니다.</p>
<pre><code class="language-typescript">export const hyperEVM = defineChain({
  id: 999,
  name: 'HyperEVM',
  rpcUrls: {
    default: { http: ['https://rpc.hyperliquid.xyz/evm'] },
  },
  // ... 기타 설정
});

export const config = createConfig({
  chains: [hyperEVM],
  // ... 기타 설정
  transports: {
    [hyperEVM.id]: http(hyperEVM.rpcUrls.default.http[0]),
  },
});</code></pre>
<h3 id="rpc-url의-의미">RPC URL의 의미</h3>
<ul>
<li><strong><a href="https://rpc.hyperliquid.xyz/evm">https://rpc.hyperliquid.xyz/evm</a></strong>: HyperEVM 네트워크의 공식 RPC 엔드포인트</li>
<li><strong>HTTP 프로토콜</strong>: 안전한 HTTPS 연결을 통해 데이터 전송</li>
<li><strong>evm 경로</strong>: 이더리움 가상 머신 호환성을 나타냄</li>
</ul>
<h3 id="viem의-http-transport">Viem의 HTTP Transport</h3>
<pre><code class="language-typescript">import { http, createConfig } from 'wagmi'

const config = createConfig({
  // ... 기타 설정
  transports: {
    [hyperEVM.id]: http(hyperEVM.rpcUrls.default.http[0]),
  },
})</code></pre>
<p><strong>왜 HTTP Transport를 사용하는가:</strong></p>
<ul>
<li><strong>간단함</strong>: 설정이 간단하고 직관적</li>
<li><strong>안정성</strong>: HTTP는 검증된 프로토콜</li>
<li><strong>호환성</strong>: 대부분의 RPC 제공자가 HTTP 지원</li>
<li><strong>캐싱</strong>: HTTP 레벨에서 캐싱 가능</li>
</ul>
<hr />
<h2 id="5-rpc-제공자-종류">5. RPC 제공자 종류</h2>
<h3 id="1-공식-rpc-노드">1. 공식 RPC 노드</h3>
<p><strong>장점:</strong></p>
<ul>
<li>신뢰성 높음</li>
<li>공식 지원</li>
<li>안정적인 성능</li>
</ul>
<p><strong>단점:</strong></p>
<ul>
<li>속도가 느릴 수 있음</li>
<li>사용자 수가 많아 혼잡할 수 있음</li>
</ul>
<p><strong>예시:</strong></p>
<pre><code class="language-typescript">// Ethereum Mainnet
https://mainnet.infura.io/v3/YOUR_API_KEY

// HyperEVM
https://rpc.hyperliquid.xyz/evm</code></pre>
<h3 id="2-서드파티-rpc-제공자">2. 서드파티 RPC 제공자</h3>
<p><strong>장점:</strong></p>
<ul>
<li>빠른 속도</li>
<li>추가 기능 제공</li>
<li>전용 지원</li>
</ul>
<p><strong>단점:</strong></p>
<ul>
<li>비용 발생 가능</li>
<li>중앙화된 서비스</li>
</ul>
<p><strong>주요 제공자들:</strong></p>
<ul>
<li><strong>Infura</strong>: 가장 인기 있는 RPC 제공자</li>
<li><strong>Alchemy</strong>: 개발자 친화적 기능 제공</li>
<li><strong>QuickNode</strong>: 고성능 노드 제공</li>
<li><strong>GetBlock</strong>: 다양한 체인 지원</li>
</ul>
<h3 id="3-자체-rpc-노드">3. 자체 RPC 노드</h3>
<p><strong>장점:</strong></p>
<ul>
<li>완전한 제어권</li>
<li>데이터 프라이버시</li>
<li>커스터마이징 가능</li>
</ul>
<p><strong>단점:</strong></p>
<ul>
<li>유지보수 비용</li>
<li>기술적 복잡성</li>
<li>안정성 확보 어려움</li>
</ul>
<h3 id="rpc-제공자-선택-기준">RPC 제공자 선택 기준</h3>
<h4 id="1-성능-요구사항">1. 성능 요구사항</h4>
<pre><code class="language-typescript">// 고성능이 필요한 경우
const fastRPC = 'https://fast-rpc-provider.com';

// 안정성이 중요한 경우
const reliableRPC = 'https://official-rpc.com';</code></pre>
<h4 id="2-비용-고려사항">2. 비용 고려사항</h4>
<pre><code class="language-typescript">// 무료 티어 사용
const freeRPC = 'https://free-rpc-provider.com';

// 유료 서비스 사용
const paidRPC = 'https://paid-rpc-provider.com';</code></pre>
<h4 id="3-기능-요구사항">3. 기능 요구사항</h4>
<pre><code class="language-typescript">// 기본 기능만 필요한 경우
const basicRPC = 'https://basic-rpc.com';

// 고급 기능이 필요한 경우
const advancedRPC = 'https://advanced-rpc.com';</code></pre>
<hr />
<h2 id="6-rpc-보안과-최적화">6. RPC 보안과 최적화</h2>
<h3 id="보안-고려사항">보안 고려사항</h3>
<h4 id="1-rpc-url-보호">1. RPC URL 보호</h4>
<pre><code class="language-typescript">// 환경 변수로 관리
const RPC_URL = process.env.NEXT_PUBLIC_RPC_URL;

// API 키 보호
const RPC_URL_WITH_KEY = `${RPC_URL}?apiKey=${process.env.RPC_API_KEY}`;</code></pre>
<h4 id="2-요청-검증">2. 요청 검증</h4>
<pre><code class="language-typescript">// 입력값 검증
function validateAddress(address: string): boolean {
  return /^0x[a-fA-F0-9]{40}$/.test(address);
}

// 요청 제한
const rateLimiter = new Map();
function checkRateLimit(userId: string): boolean {
  const now = Date.now();
  const userRequests = rateLimiter.get(userId) || [];
  const recentRequests = userRequests.filter(time =&gt; now - time &lt; 60000);

  if (recentRequests.length &gt;= 100) return false;

  recentRequests.push(now);
  rateLimiter.set(userId, recentRequests);
  return true;
}</code></pre>
<h3 id="성능-최적화">성능 최적화</h3>
<h4 id="1-배치-요청">1. 배치 요청</h4>
<pre><code class="language-typescript">// 여러 요청을 하나로 묶기
const batchRequest = [
  { jsonrpc: '2.0', method: 'eth_getBalance', params: [address1, 'latest'], id: 1 },
  { jsonrpc: '2.0', method: 'eth_getBalance', params: [address2, 'latest'], id: 2 },
  { jsonrpc: '2.0', method: 'eth_getBalance', params: [address3, 'latest'], id: 3 }
];

const response = await fetch(RPC_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(batchRequest)
});</code></pre>
<h4 id="2-캐싱-전략">2. 캐싱 전략</h4>
<pre><code class="language-typescript">// 메모리 캐싱
const cache = new Map();
const CACHE_TTL = 5 * 60 * 1000; // 5분

async function getCachedBalance(address: string): Promise&lt;string&gt; {
  const cacheKey = `balance_${address}`;
  const cached = cache.get(cacheKey);

  if (cached &amp;&amp; Date.now() - cached.timestamp &lt; CACHE_TTL) {
    return cached.data;
  }

  const balance = await provider.getBalance(address);
  cache.set(cacheKey, {
    data: balance,
    timestamp: Date.now()
  });

  return balance;
}</code></pre>
<h4 id="3-연결-풀링">3. 연결 풀링</h4>
<pre><code class="language-typescript">// HTTP 연결 재사용
const httpAgent = new http.Agent({
  keepAlive: true,
  maxSockets: 10,
  maxFreeSockets: 5
});

const response = await fetch(RPC_URL, {
  method: 'POST',
  agent: httpAgent,
  // ... 기타 설정
});</code></pre>
<hr />
<h2 id="7-실제-사용-예시">7. 실제 사용 예시</h2>
<h3 id="프로젝트에서의-rpc-사용-패턴">프로젝트에서의 RPC 사용 패턴</h3>
<h4 id="1-지갑-연결-시">1. 지갑 연결 시</h4>
<pre><code class="language-typescript">// ConnectButton 컴포넌트에서
const { connect, connectors } = useConnect();

// MetaMask 연결 시 RPC를 통해 네트워크 정보 조회
const handleConnect = async () =&gt; {
  try {
    await connect({ connector: connectors[0] });
    // RPC를 통해 연결된 네트워크의 체인 ID 확인
    const chainId = await window.ethereum.request({ method: 'eth_chainId' });
    console.log('연결된 네트워크:', chainId);
  } catch (error) {
    console.error('연결 실패:', error);
  }
};</code></pre>
<h4 id="2-잔액-조회-시">2. 잔액 조회 시</h4>
<pre><code class="language-typescript">// useBalance 훅 사용
const { data: hyPEBalance } = useBalance({
  address: address,
  query: { enabled: isConnected &amp;&amp; isHyperEVM(chainId) }
});

// 내부적으로 RPC 호출
// eth_getBalance(address, 'latest')</code></pre>
<h4 id="3-네트워크-전환-시">3. 네트워크 전환 시</h4>
<pre><code class="language-typescript">// switchToHyperEVM 함수에서
export async function switchToHyperEVM() {
  try {
    // RPC를 통해 네트워크 전환 요청
    await window.ethereum.request({
      method: 'wallet_switchEthereumChain',
      params: [{ chainId: `0x${hyperEVM.id.toString(16)}` }]
    });
  } catch (switchError: any) {
    if (switchError.code === 4902) {
      // RPC를 통해 새 네트워크 추가
      await window.ethereum?.request({
        method: 'wallet_addEthereumChain',
        params: [{
          chainId: `0x${hyperEVM.id.toString(16)}`,
          chainName: hyperEVM.name,
          rpcUrls: hyperEVM.rpcUrls.default.http,
          // ... 기타 설정
        }]
      });
    }
  }
}</code></pre>
<h3 id="일반적인-rpc-호출-패턴">일반적인 RPC 호출 패턴</h3>
<h4 id="1-계정-정보-조회">1. 계정 정보 조회</h4>
<pre><code class="language-typescript">// 계정 잔액
const balance = await provider.getBalance(address);

// 트랜잭션 수
const txCount = await provider.getTransactionCount(address);

// 계정 코드 (컨트랙트 여부 확인)
const code = await provider.getCode(address);</code></pre>
<h4 id="2-블록-정보-조회">2. 블록 정보 조회</h4>
<pre><code class="language-typescript">// 최신 블록 번호
const blockNumber = await provider.getBlockNumber();

// 특정 블록 정보
const block = await provider.getBlock(blockNumber);

// 블록 해시로 블록 조회
const blockByHash = await provider.getBlock(blockHash);</code></pre>
<h4 id="3-트랜잭션-정보-조회">3. 트랜잭션 정보 조회</h4>
<pre><code class="language-typescript">// 트랜잭션 상세 정보
const tx = await provider.getTransaction(txHash);

// 트랜잭션 영수증
const receipt = await provider.getTransactionReceipt(txHash);

// 트랜잭션 상태 확인
const status = receipt.status; // 1: 성공, 0: 실패</code></pre>
<hr />
<h2 id="결론">결론</h2>
<p>RPC는 Web3 생태계의 핵심 기술로, dApp이 블록체인과 상호작용할 수 있게 해줍니다.</p>
<h3 id="핵심-포인트">핵심 포인트</h3>
<ol>
<li><strong>통신 다리</strong>: dApp과 블록체인 네트워크 사이의 필수적인 연결</li>
<li><strong>표준화</strong>: JSON-RPC 2.0 표준을 따르는 일관된 인터페이스</li>
<li><strong>다양한 프로토콜</strong>: HTTP, WebSocket 등 다양한 통신 방식 지원</li>
<li><strong>보안과 성능</strong>: 적절한 보안 조치와 성능 최적화가 중요</li>
<li><strong>프로젝트 적용</strong>: HyperEVM 네트워크와의 안전하고 효율적인 통신</li>
</ol>
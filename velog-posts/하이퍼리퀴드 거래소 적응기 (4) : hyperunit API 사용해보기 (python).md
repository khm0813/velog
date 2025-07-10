<p><strong>Hyperunit</strong>은 다양한 체인의 자산을 Hyperliquid 거래소에서 자유롭게 온보딩/오프보딩할 수 있게 해주는 공식 브릿지입니다.<br />이 글에서는 FastAPI로 이더리움 지갑을 생성한 뒤,<br /><strong>Hyperunit 공식 API</strong>를 사용해 <strong>ETH·SOL 입금주소</strong>를 만드는 실전 예제를 공유합니다.</p>
<hr />
<h2 id="목차">목차</h2>
<ol>
<li>Hyperunit API 입금주소 생성 방식 (공식 문서 인용)</li>
<li>FastAPI에서 Hyperliquid용 이더리움 지갑 생성</li>
<li>Hyperunit API로 ETH·SOL 입금주소 발급</li>
<li>실제 코드 예시 (with httpx)</li>
<li>결과 예시 및 실전 팁</li>
</ol>
<hr />
<h3 id="1-hyperunit-api-입금주소-생성-방식">1. Hyperunit API 입금주소 생성 방식</h3>
<blockquote>
<p>Hyperunit 공식 문서에 따르면,<br />입금주소 생성은 아래와 같이 단순한 <strong>GET 방식 REST API</strong>로 처리할 수 있습니다.</p>
</blockquote>
<pre><code class="language-http">GET /gen/:src_chain/:dst_chain/:asset/:dst_addr

src_chain: 자산이 원래 존재하는 체인 (예: ethereum, solana)
dst_chain: 자산을 받을 곳 (항상 hyperliquid)
asset: 코인명 (eth, sol, btc 등)
dst_addr: 받을 사람(=내 지갑 주소, Hyperliquid 형식)
</code></pre>
<blockquote>
<p><a href="https://docs.hyperunit.xyz/docs/integration/api#generating-deposit-addresses">https://docs.hyperunit.xyz/docs/integration/api#generating-deposit-addresses</a></p>
</blockquote>
<h3 id="2-fastapi에서-hyperliquid용-이더리움-지갑-만들기">2. FastAPI에서 Hyperliquid용 이더리움 지갑 만들기</h3>
<p>우리는 web3 라이브러리를 이용해서
“MetaMask와 100% 호환되는 이더리움 지갑”을 생성할 수 있습니다.</p>
<pre><code class="language-python">from web3 import Account

# 지갑 생성
acct = Account.create()
print(&quot;내 주소:&quot;, acct.address)
print(&quot;Private Key:&quot;, acct.key.hex())</code></pre>
<blockquote>
<p>Hyperliquid 지갑은 이더리움 EOA(Externally Owned Account) 표준을 따릅니다.</p>
</blockquote>
<h3 id="3-hyperunit-api로-eth·sol-입금주소-발급">3. Hyperunit API로 ETH·SOL 입금주소 발급</h3>
<p>API 요청 예시:
ETH: GET <a href="https://api.hyperunit.xyz/gen/ethereum/hyperliquid/eth/%7B%EB%82%B4_%EC%A3%BC%EC%86%8C%7D">https://api.hyperunit.xyz/gen/ethereum/hyperliquid/eth/{내_주소}</a>
SOL: GET <a href="https://api.hyperunit.xyz/gen/solana/hyperliquid/sol/%7B%EB%82%B4_%EC%A3%BC%EC%86%8C%7D">https://api.hyperunit.xyz/gen/solana/hyperliquid/sol/{내_주소}</a></p>
<p>응답은 아래와 같이, 실제 해당 체인에서 입금 가능한 입금주소와 안전성 검증용 signatures를 함께 반환합니다.</p>
<pre><code class="language-json">{
  &quot;address&quot;: &quot;0x3F34447e9bCE5406A74FD493fb701F2C609e391f&quot;,
  &quot;signatures&quot;: {
    &quot;field-node&quot;: &quot;...&quot;,
    &quot;hl-node&quot;: &quot;...&quot;,
    &quot;unit-node&quot;: &quot;...&quot;
  },
  &quot;status&quot;: &quot;OK&quot;
}</code></pre>
<blockquote>
<p>signatures란?
입금주소가 Hyperunit 가디언 노드 합의로 생성된 “진짜 안전한 주소”임을 보증하는 디지털 서명입니다.
자세한 검증 로직은 공식 문서에 설명되어 있습니다.
<a href="https://docs.hyperunit.xyz/docs/integration/signature-verification">https://docs.hyperunit.xyz/docs/integration/signature-verification</a></p>
</blockquote>
<h3 id="4-실제-코드-예시-fastapi--httpx">4. 실제 코드 예시 (FastAPI + httpx)</h3>
<p>아래 예시는 FastAPI에서 지갑을 생성하고 Hyperunit API로 ETH/SOL 입금주소를 요청하고 JSON으로 한 번에 반환하는 API 예제입니다.</p>
<pre><code class="language-python">from fastapi import APIRouter
from web3 import Account
import httpx

router = APIRouter()

@router.get(&quot;/gen_wallet_and_deposit_addresses&quot;)
async def gen_wallet_and_deposit_addresses():
    # 1. 지갑 생성
    acct = Account.create()
    address = acct.address

    # 2. Hyperunit API로 입금주소 요청
    async with httpx.AsyncClient() as client:
        eth_resp = await client.get(
            f&quot;https://api.hyperunit.xyz/gen/ethereum/hyperliquid/eth/{address}&quot;
        )
        sol_resp = await client.get(
            f&quot;https://api.hyperunit.xyz/gen/solana/hyperliquid/sol/{address}&quot;
        )
        eth_data = eth_resp.json()
        sol_data = sol_resp.json()

    # 3. 결과 반환
    return {
        &quot;wallet&quot;: {
            &quot;address&quot;: address,
            &quot;private_key&quot;: acct.key.hex()
        },
        &quot;deposit_address&quot;: {
            &quot;ETH&quot;: eth_data,
            &quot;SOL&quot;: sol_data,
        }
    }</code></pre>
<p>API를 호출하면 아래와 같은 JSON을 받을 수 있습니다.</p>
<pre><code class="language-json">{
  &quot;wallet&quot;: {
    &quot;address&quot;: &quot;0xAE0473f7E8F04927057711A7FB0351CA7fD7ef72&quot;,
    &quot;private_key&quot;: &quot;e63b19...e016b&quot;
  },
  &quot;deposit_address&quot;: {
    &quot;ETH&quot;: {
      &quot;address&quot;: &quot;0x3F34447e9bCE5406A74FD493fb701F2C609e391f&quot;,
      &quot;signatures&quot;: { ... },
      &quot;status&quot;: &quot;OK&quot;
    },
    &quot;SOL&quot;: {
      &quot;address&quot;: &quot;5tXPQEXtbppaZ4yX3gg2HV8mLFS1PzuWBy4rPBYZwWVq&quot;,
      &quot;signatures&quot;: { ... },
      &quot;status&quot;: &quot;OK&quot;
    }
  }
}</code></pre>
<blockquote>
<p>  •    이 주소는 Hyperunit 가디언 노드의 합의로 생성되므로, 공식 서명(signatures) 검증 로직을 통해 안전하게 신뢰할 수 있습니다.
    •    입금 시에는 반드시 ETH는 Ethereum 메인넷, SOL는 Solana 메인넷에서 송금해야 합니다.
    •    Arbitrum 등 L2에서 보내면 입금 반영이 안 되고 자산이 사라질 수 있습니다!
    •    입금 주소와 signatures는 공식 문서의 Python/JS 검증 예제대로 반드시 검증하세요.</p>
</blockquote>
<p>Hyperunit의 간단한 API 연동만으로 모든 유저에게 개인 ETH/SOL 입금주소를 발급해줄 수 있습니다.</p>
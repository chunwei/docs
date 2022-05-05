<!--
 * @Author: Chunwei Lu
 * @Date: 2022-05-05 14:27:52
 * @LastEditTime: 2022-05-05 14:28:57
 * @LastEditors: Chunwei Lu
 * @FilePath: /docs/tutorial/page-2.md
-->
---
description: Tabs
---

# Tabs

{% tabs %}
{% tab title="JS" %}

```javascript
const ABI = []; // Add ABI of 0xdAC17F958D2ee523a2206206994597C13D831ec7
const options = {
  chain: "eth",
  address: "0xdAC17F958D2ee523a2206206994597C13D831ec7",
  function_name: "balanceOf",
  abi: ABI,
  params: { who: "0x3355d6E71585d4e619f4dB4C7c5Bfe549b278299" },
};
const allowance = await Moralis.Web3API.native.runContractFunction(options);
```

{% endtab %}

{% tab title="React" %}

```javascript
import React from "react";
import { useMoralisWeb3Api } from "react-moralis";
const GetAddressBalanceOfUSDT = () => {
  // 0xdAC17F958D2ee523a2206206994597C13D831ec7 = contract address of USDT
  const { native } = useMoralisWeb3Api();
  const ABI = []; // Add ABI of 0xdAC17F958D2ee523a2206206994597C13D831ec7
  const options = {
    chain: "eth",
    address: "0xdAC17F958D2ee523a2206206994597C13D831ec7",
    function_name: "balanceOf",
    abi: ABI,
    params: { who: "0x3355d6E71585d4e619f4dB4C7c5Bfe549b278299" },
  };
  const { fetch, data, error, isLoading } = useMoralisWeb3ApiCall(
    native.runContractFunction,
    { ...options }
  );
  return (
    // Use your custom error component to show errors
    <div style={{ height: "100vh", overflow: "auto" }}>
      <div>
        {error && <ErrorMessage error={error} />}
        <button
          onClick={() => {
            fetch({ params: options });
          }}
        >
          Fetch data
        </button>
        {data && <pre>{JSON.stringify(data)}</pre>}
      </div>
    </div>
  );
};
```

{% endtab %}

{% tab title="curl" %}

```bash
curl -X 'POST' \
 'https://deep-index.moralis.io/api/v2/0xdAC17F958D2ee523a2206206994597C13D831ec7/function?chain=eth&function_name=balanceOf' \
 -H 'accept: application/json' \
 -H 'X-API-Key: MY-API-KEY' \
 -H 'Content-Type: application/json' \
 -d '{
"abi": MY-CONTRACT-ABI ,
"params": {"who": "0x3355d6E71585d4e619f4dB4C7c5Bfe549b278299" }
}'
```

{% endtab %}{% endtabs %}
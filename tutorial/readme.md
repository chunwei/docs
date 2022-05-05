# README

### 开发者指南

#### 目录

* 安装
* 配置
  * 配置环境变量
  * 初始化SDK
* 资产
  * 创建合集
  * 铸造NFT
* 订单
  * 创建订单
  * 接受订单
  * 取消订单
  * 查询订单

#### 安装

```
npm install element-js-tron
```

sdk依赖tronlink钱包进行链上操作

在浏览器环境下直接使用TronLink注入到window.tronWeb

在node环境下使用，还需安装 tronweb,

```
npm install tronweb
```

#### 配置

**配置环境变量**

如在node环境下使用，需要配置环境变量

1. 在项目根目录下新建.env文件，内容如下

```
# 用于操作的钱包私钥
PK_TAKER            =  xxxxx
PK_MAKER            =  xxxxx

# trongrid网络到APIKEY
TRON_PRO_API_KEY    =  xxxxx-xxxx-xxxxx-xxxxx-xxxxx
```

注意：将.env加入到.gitignore中，避免被提交

1. 安装dotenv

```
npm install dotenv
```

1. 在项目文件中引入dotenv，并配置

```js
const dotenv = require('dotenv');
dotenv.config();
```

**初始化SDK**

```js
/* 0. 引入SDK */
import sdk from "element-js-tron";
const { ElementOrders, Network, AssetFactory, ElementSchemaName } = sdk;
/* 1. 初始化tronWeb */
/* 浏览器环境下 */
// const {tronWeb} = window;
/* node环境下 */
const tronWeb = new TronWeb({
    fullHost: 'https://api.shasta.trongrid.io',
    privateKey: process.env.PK_MAKER,
    headers: { "TRON-PRO-API-KEY": process.env.TRON_PRO_API_KEY },
  });

/* 2. 初始化SDK */
const networkName = Network.Shasta;
const sdk_orders = new ElementOrders(tronWeb, { networkName });

/* 3. 登录，获取后续api操作所需token */
await sdk_orders.login();
// 然后就可以进行订单操作了

/* 4. 如需创建资产，可通过以下两种方法获取工厂类 */
/* 4.1 从sdk_orders中获取工厂类 */
const sdk_fab = sdk_orders.assetFactory
/* 4.2 new AssetFactory */
// const sdk_fab = new AssetFactory(tronWeb, { networkName })
/* 然后就可以 */
// sdk_fab.createAsset721(...); // 创建721资产合约
// sdk_fab.mint721(...);        // 铸造721资产
```

#### 资产

**创建合集**

sdk\_fab.createAsset721(name,symbol,baseUrl,options)

方法说明

```js
/**
 * 部署ERC721合约，返回合约地址
 * @param {string} name - 合约名称
 * @param {string} symbol - 合约符号
 * @param {string} baseUrl - 合约metadata的baseUrl
 * @param {object} [options]  - 可选参数，参考 https://cn.developers.tron.network/reference/tronweb-createsmartcontract
 * @returns {Promise<{txHash,txSend}>}  Promise对象，包括交易hash和广播事件接受器，可以通过await txSend获取创建的合约地址
 */
async createAsset721(name: string, symbol: string, baseUrl: string, options?: any) 
```

方法使用示例

```js
const sdk_fab = ...
const createRes = await sdk_fab.createAsset721(
    'MyNFTSymbol',  // Symbol
    'MyNFTName',    // Name
    'https://ipfs.io/ipfs/QmPMc4tcBsMqLRuCQtPmPe84bpSjrC3Ky7t3JWuHXYB4aS/'  // metadata json url
)
const output = await createRes.txSend.catch((err) => console.log(err))
// 得到新创建的合约地址
const nftContract = output.contract_address  
```

**铸造NFT**

sdk\_fab.mint721(nftContract, toAddress,url)

方法说明

```js
/**
 * 
 * @param {string} assetAddr - NFT合约地址，指定在哪个合约中铸造NFT
 * @param {string} to - NFT token接收者地址
 * @param {string} [url] - 可选，NFT metadata的baseUrl
 * @returns Promise
 */
async mint721(assetAddr, to, url) 
```

方法使用示例

```js
const nftContract = ...
const to = ...
const url = 'https://ipfs.io/ipfs/QmPMc4tcBsMqLRuCQtPmPe84bpSjrC3Ky7t3JWuHXYB4aS/1' 
const defer = await sdk_fab.mint721(nftContract, to)
defer.txSend
    .on('confirmation', (r) => console.log('confirmation', r))
    .on('error', (r) => console.log('error', r))
await defer.txSend
```

#### 订单

**创建订单**

* tronWeb使用`maker`账号
* 准备订单参数
* 调用创建方法

```js
/* 准备订单参数 */
const sellOrderParams = {
  asset: {
    tokenId: '1', // asset token id
    tokenAddress: 'TGTjfy2KqDdVBLuYYy3bky31Kw9DSxy36Y', // contract address
    schemaName: ElementSchemaName.ERC721, // asset type ERC721、ERC1155
    collection: {
      transferFeeAddress: 'TRX6ZfaY16SjZMRxHZy4TZQysq62akvm5T',  // 版税收取地址
      elementSellerFeeBasisPoints: 300  // 版税收取比例，10000为100%
    }
  },
  quantity: 1,  // 购买数量，ERC721取1
  startAmount: 10, // 价格
  paymentToken: {  // 支付Token
    name: 'WTRX', // TRX、WTRX
    symbol: 'WTRX', // TRX、WTRX
    address: 'TF2ipqRnrDzuPrANJCJggmaM753FCE1ZWE', //TRX 设为 NULL_ADDRESS, WTRX 设为Token合约地址
    decimals: 6
  },
  // listingTime: Math.round(Date.now() / 1000 + 1), // 上架时间，默认为当前时间
  // expirationTime: Math.round(Date.now() / 1000 + 7 * 24 * 60 * 60)  // 订单过期时间，默认取0，表示永不过期
}

/* 调用创建方法，得到订单 */
const sellOrder = await sdk_orders.createSellOrder(sellOrderParams)
order = sellOrder
console.log(sellOrder)
```

**接受订单**

* tronWeb使用`taker`账号
* 生成对手单
* 调用撮合方法

```js
/* 生成对手单 */
// 使用前一步创建的订单
order = sellOrder  
// 或者使用订单查询接口查询到的订单, 详见下文【查询订单】
// orders = await sdk_orders.ordersQuery(queryParams)
// order = orders[0]
const {buy,sell} = sdk_orders.makeMatchingOrder(order)

/* 调用撮合方法 */
const res = await sdk_orders.account.orderMatch({ buy, sell }).catch(console.log)
```

**取消订单**

* 查询获得准备取消的订单
* 调用取消方法

```js
const order = await sdk_orders.ordersQuery(queryParams)
const res = await sdk_orders.account.orderCancel(order[0])
```

**查询订单**

```js
interface ChainInfo {
  chain?: string
  chainId?: string
}
interface OrderQueryParams extends ChainInfo {
  assetContractAddress: string // 目标资产合约地址
  tokenId: string // 目标资产token id
  orderType: OrderType // 订单类型 0:买入 1:卖出
}

// 查询订单,返回订单列表
const orders = await sdk_orders.ordersQuery(queryParams) 
```

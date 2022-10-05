---
sidebar_position: 1
id: sdk-example
---

# Zincirler Arası Transfer

Connext protokolüyle etkileşim için ana giriş noktası "xcall"dır. Bu yöntem bir çapraz zincir etkileşimini başlatır ve kullanıcının tek yapması gereken hedef zincirde tamamlanmasını beklemektir. *Bu işlemden sonra gerekli kullanıcı etkileşimi yoktur*!

Connext SDK, geliştiricilerin standart Node.js veya web ortamlarında Connext protokolüyle etkileşim kurmasına olanak tanır.
--- 

## Giriş

Bu örnekte, Goerli'deki bir cüzdandan Optimism-Goerli'deki bir hedef adrese para aktarmak için bir "xcall"ın nasıl yürütüleceğini göstereceğiz.

### 1. Setup

[Node.js](https://nodejs.dev/en/learn/how-to-install-nodejs/) yükleyin ve **Node.js v16** kullanın. Sürüm değiştirmeyi kolaylaştıracak bir düğüm sürüm yöneticisi olan `nvm`yi kurmak için talimatları izleyin.

Bir proje klasörü oluşturun ve paketi başlatın. Proje bilgilerini istediğiniz gibi doldurun.

```bash npm2yarn
mkdir node-examples && cd node-examples
npm init
```

TypeScript kullanacağız, bu yüzden aşağıdakini kurun ve `tsconfig.json` dosyasını oluşturun.
```bash npm2yarn
npm install --save-dev @types/node @types/chai @types/mocha typescript 
npx tsc --init # yarn tsc --init
```
Üst düzey beklemeyi kullanmak istiyoruz, bu nedenle derleyici seçeneklerini buna göre ayarlayacağız.

```json title="tsconfig.json"
{
  "compilerOptions": {
    "outDir": "./dist",
    "target": "es2017",
    "module": "esnext",
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true
  },
  "exclude": ["node_modules"]
}
```

"type" ve "scriptleri" kök düzeyinde girdiler olarak "package.json"a ekleyin - bunlar zaten mevcut olabilir, bu nedenle bunları aşağıdakiyle değiştirin.

```json title="package.json"
  "type": "module",
  "scripts": {
    "build": "tsc && node dist/xtransfer.js",
    "xtransfer": "node dist/xtransfer.js"
  }
```
Bu örnekte tüm kodları yazacağımız `src` dizininde `xtransfer.ts` oluşturun.

```bash
mkdir src && touch src/xtransfer.ts
```

### 2. Bağımlılıkların yüklenmesi

Connext SDK ve ethers'in en son beta sürümünü yükleyin.

```bash npm2yarn
npm install @connext/nxtp-sdk@beta
npm install ethers
```

### 3. Kod

Bu, örneğin tam kodudur. Yorumları okuyun ve "<...>" arasındaki tüm yer tutucuları değiştirin.

```ts title="src/xtransfer.ts"
import { create, NxtpSdkConfig } from "@connext/nxtp-sdk";
import { ethers } from "ethers";

// Instantiate a Wallet object using your private key (i.e. from Metamask) and use it as a Signer.
const privateKey = "<PRIVATE_KEY>";
let signer = new ethers.Wallet(privateKey);

// Connext to a Provider on the sending chain. You can use a provider like Infura (https://infura.io/) or Alchemy (https://www.alchemy.com/).
const provider = new ethers.providers.JsonRpcProvider("<GOERLI_RPC_URL>");
signer = signer.connect(provider);
const signerAddress = await signer.getAddress();

// Construct the `NxtpSdkConfig`. Values like Domain IDs and token addresses are already filled in for you. You can reference these in the "Resources" tab of the docs. 
const nxtpConfig: NxtpSdkConfig = {
  logLevel: "info",
  signerAddress: signerAddress,
  chains: {
    "1735353714": {
      providers: ["<GOERLI_RPC_URL>"],
      assets: [
        {
          name: "TEST",
          symbol: "TEST",
          address: "0x7ea6eA49B0b0Ae9c5db7907d139D9Cd3439862a1",
        },
      ],
    },
    "1735356532": {
      providers: ["<OPTIMISM_GOERLI_RPC_URL>"],
      assets: [
        {
          name: "TEST",
          symbol: "TEST",
          address: "0x68Db1c8d85C09d546097C65ec7DCBFF4D6497CbF",
        },
      ],
    },
  },
};

// Create the SDK instance.
const {nxtpSdkBase} = await create(nxtpConfig);

const amount = 1000000000000000000; // (1 TEST)

// Construct the arguments that will be passed into `xcall`.
const callParams = {
  to: signerAddress, // the address that should receive the funds
  callData: "0x", // empty calldata for a simple transfer
  originDomain: "1735353714", // send from Goerli
  destinationDomain: "1735356532", // to Optimism-Goerli
  agent: signerAddress, // address allowed to execute transaction on destination side in addition to relayers
  recovery: signerAddress, // fallback address to send funds to if execution fails on destination side
  forceSlow: false, // option to force slow path instead of paying 0.05% fee on fast liquidity transfers
  receiveLocal: false, // option to receive the local bridge-flavored asset instead of the adopted asset
  callback: ethers.constants.AddressZero, // zero address because we don't expect a callback
  callbackFee: "0", // fee paid to relayers; relayers don't take any fees on testnet
  relayerFee: "0", // fee paid to relayers; relayers don't take any fees on testnet
  destinationMinOut: (amount * 0.97).toString(), // the minimum amount that the user will accept due to slippage from the StableSwap pool (3% here)
};

const xCallArgs = {
  params: callParams,
  transactingAsset: "0x7ea6eA49B0b0Ae9c5db7907d139D9Cd3439862a1", // the Goerli Test Token
  transactingAmount: amount.toString(), 
  originMinOut: (amount * 0.97).toString() // the minimum amount that the user will accept due to slippage from the StableSwap pool (3% here)
};

// Approve the asset transfer. This is necessary because funds will first be sent to the Connext contract before being bridged.
const approveTxReq = await nxtpSdkBase.approveIfNeeded(
  xCallArgs.params.originDomain,
  xCallArgs.transactingAsset,
  xCallArgs.transactingAmount
)
const approveTxReceipt = await signer.sendTransaction(approveTxReq);
await approveTxReceipt.wait();

// Send the xcall
const xcallTxReq = await nxtpSdkBase.xcall(xCallArgs);
xcallTxReq.gasLimit = ethers.BigNumber.from("20000000"); 
const xcallTxReceipt = await signer.sendTransaction(xcallTxReq);
console.log(xcallTxReceipt);
const xcallResult = await xcallTxReceipt.wait();
```

### 4. Çalıştırın

Şimdi çapraz zincir aktarımını başlatmak için çalıştırabiliriz!

```bash npm2yarn
npm run build
npm run xtransfer
```

### 5. 'xcall' çağrısını takip edin

Bu talimatları izleyerek bu xcall'in durumunu takip etmek için yukarıda kaydettiğimiz işlem makbuzundan "hash" işlemini kullanabiliriz.

[xcall çağrısını takip etme](../xcall-status)

Optimism-Goerli tarafında transfer göründükten sonra, transfer edilen tokenlar hedef cüzdanda görünmelidir.

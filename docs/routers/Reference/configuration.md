---
sidebar_position: 5
---

# Yapılandırma

Yönlendirici, [docker-compose deposunun](https://github.com/connext/nxtp-router-docker-compose) kök dizinindeki "config.json" yapılandırma dosyasını kullanarak yapılandırmayı kabul eder.

Adresler ve URL'ler için [testnet](../../developers/testing-against-testnet.md) veya mainnet (yakında) kılavuzlarına bakın.

JSON şeması aşağıdaki anahtarları kabul eder:

- `sequencerUrl`: Sıralayıcının URL'si.
- "redis": _Gerektirir_. Harici bir redis örneğini yapılandırmak için aşağıdaki anahtarları içeren nesne:
   - "host": Redis örneğinin ana bilgisayar adı.
   - "port": Redis örneğinin bağlantı noktası.
- "server": _Gerekli_. HTTP sunucusunu yapılandırmak için aşağıdaki anahtarları içeren nesne:
   - `adminToken`: _Gerekli_. Yönetici isteklerinin kimliğini doğrulamak için kullanılan gizli belirteç.
   - "port": _İsteğe bağlı_. Yönlendiricinin dinleyeceği bağlantı noktası. Varsayılan "8080"dir.
   - "host": _İsteğe bağlı_. Yönlendiricinin dinleyeceği ana bilgisayar. Varsayılan olarak "0.0.0.0" olarak ayarlanır.
   - "requestLimit": _İsteğe bağlı_. Kullanılmamış.
- "web3SignerUrl": _Önerilen_. Çalışan bir [Web3Signer](https://docs.web3signer.consensys.net/en/latest/) örneğinin URL'si. Bu, özel anahtar depolaması için önerilen yaklaşımdır.
- "mnemonic": _İsteğe bağlı, Cesaretini kıran_. Özel anahtarı oluşturmak için kullanılan anımsatıcı. Anımsatıcıyı doğrudan yapılandırma dosyasında kullanmak güvenli değildir ve önerilmez.
- "chains": _Gerekli_. Zincir konfigürasyonu. Değer olarak aşağıdaki nesne şemasına sahip [Etki Alanı Kimlikleri](./developers/testing-against-testnet#domain-ids) tarafından anahtarlanan aşağıdakilere sahip bir JSON nesnesi:
   - "providers": _Gerekli_. Bir zincir için sağlayıcı URL'leri dizisi. En az 1 URL kullanın, ancak ek URL'ler sağlayıcı sorunlarına karşı daha fazla yedek koruma sağlar.
- "assets": _Gerekli_. Bir dizi varlık. Her varlık, aşağıdaki anahtarlara sahip bir JSON nesnesidir:
    - "assetId": _Gerekli_. Varlık kimliği (ERC20 belirteç adresi). Yerel varlıklar için "0x0000000000000000000000000000000000000000" kullanın.
    - "name": _Gerekli_. Varlık Adı.
    - "mainnetEquivalent": _İsteğe bağlı_. Fiyat belirlemeleri için ETH ana ağındaki eşdeğer varlık. Varsayılan olarak Connext'in yapılandırmasıdır.
  - "subgraph": _İsteğe bağlı_. Alt grafikleri yapılandırmak için aşağıdaki anahtarları içeren bir nesne.
    - "runtime": _İsteğe bağlı_. Bir zincir için alt grafik URL'leri dizisi. Ek URL'ler, alt yazı sorunlarına karşı daha fazla yedek koruma sağlar. Sağlanmazsa, varsayılan olarak Connext'in barındırılan alt grafiklerini kullanır.
    - "analytics": _İsteğe bağlı_. Bir zincir için alt grafik URL'leri dizisi. Ek URL'ler, alt yazı sorunlarına karşı daha fazla yedek koruma sağlar. Sağlanmazsa, varsayılan olarak Connext'in barındırılan alt grafiklerini kullanır.
    - "maxLag": _İsteğe bağlı_. Alt grafiğin en son blok numarasının, sağlayıcının en son blok numarasının arkasında olmasına izin verecek blok sayısı. Varsayılan olarak zincir başına önerilen değerdir.
- "deployments": _İsteğe bağlı_. Dağıtımları yapılandırmak için aşağıdaki anahtarları içeren bir nesne.
     - "connext": _İsteğe bağlı_. Connext.sol sözleşmesinin adresi. Sağlanmazsa, varsayılan olarak en son dağıtılan sözleşmeler kullanılacaktır.
   - "confirmations": _İsteğe bağlı_. Bir işlemin zincirde geçerli sayılması için gereken teyit sayısı. Varsayılan değerler [burada](https://github.com/connext/chaindata/blob/29cc0250aff398cdf9326dcb7698d291f3e3015a/crossChain.json).
   - "minGas": _İsteğe bağlı_. Wei'de belirtilen, müzayedelere katılmak için yönlendiricinin imzalayan adresinin elinde bulundurması gereken minimum gaz miktarı. Varsayılan "1000000000000000000" (0,1 Eter) olarak ayarlanmıştır.
   - "benzin İstasyonları": _İsteğe bağlı_. Benzin istasyonu URL'leri dizisi, varsayılan olarak RPC'nin gaz tahminini kullanır.
- "subgraphPollInterval": _İsteğe bağlı_. Alt grafik yoklama aralığını milisaniye cinsinden kontrol edin. 15_000 (15 saniye) varsayılandır.
- `maxSlippage`: _İsteğe bağlı_. Transferlerde izin verilecek maksimum kayma miktarı.

## Örnek Yapılandırma Dosyası

_Bunlar örnek RPC URL'leridir. Lütfen kendi RPC URL'lerinizi alın!_

```json
{
  "chains": {
    "1111": {
      "assets": [
        {
          "address": "0x3FFc03F05D1869f493c7dbf913E636C6280e0ff9",
          "name": "TEST"
        }
      ],
      "providers": ["https://rinkeby.infura.io/v3/...", "https://rpc.ankr.com/eth_rinkeby"]
    },
    "3331": {
      "providers": [
        "https://goerli.infura.io/v3/..."
      ],
      "assets": [
        {
          "address": "0x3FFc03F05D1869f493c7dbf913E636C6280e0ff9",
          "name": "TEST"
        }
      ]
    }
  },
  "logLevel": "debug",
  "web3signer": "http://signer:9000",
  "sequencerUrl": "https://sequencer.testnet.connext.ninja",
  "server": { "adminToken": "blahblahblah" },
  "redis": {
    "host": "redis",
    "port": 6379
  }
}
```

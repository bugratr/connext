---
sidebar_position: 5
id: faq
---

# SSS
Bu, Amarok genel test ağı için Sıkça Sorulan Soruların bir listesidir.

## Hangi zincirler desteklenir?
Güncellenmiş bir liste [Desteklenen Zincirler](./info/chains#supported-chains) adresinde bulunabilir.

## Hangi varlıklar desteklenir?

[Dağıtılan Sözleşme Adresleri](./developers/testing-against-testnet#deployed-contract-addresses) adresindeki varlıklar için dağıtılan adreslere bakın.

## "Normal", "temsil", "kabul edilen" ve "yerel" varlıklar arasındaki fark nedir?

<img src="/img/faq/assets.png" alt="drawing" width="1000"/>

## Bir jetonun kurallı ayrıntılarını nasıl bulabilirim?

Bir belirtecin kurallı domainId ve tokenId'i, 'TokenRegistry'nin ['getTokenId'] (https://github.com/connext/nxtp/blob/3d0af2251b2d8d244d2617be6fb738c09a571022/packages/deployments/contracts/contracts/core/connext/helpers/TokenRegistry.sol#L176) işlevi çağrılarak bulunabilir.

Örnek:
- İlgilenilen token, Rinkeby'de TestERC20'dir (`0x3FFc03F05D1869f493c7dbf913E636C6280e0ff9`). Kanonik domainId ve tokenId'sini bulmak istiyoruz.
- [buradan](./developers/testing-against-testnet#deployed-contract-addresses) Rinkeby'deki TokenRegistry sözleşme adresini bulun.
- "getTokenId" çağrısı yapın (bu örnek, sözleşmeden okumak için Foundry'nin "cast"ini kullanır) 

  ```bash
  cast call --chain rinkeby 0x1A3BA482D98CCB858AEacB3B839f952390099cE6 "getTokenId(address)(uint32,bytes32)" "0x3FFc03F05D1869f493c7dbf913E636C6280e0ff9" --rpc-url <rinkeby_rpc_url>
  ```

Dönüş:

  ```bash
  3331 # the canonical domainId is Goerli
  0x00000000000000000000000026fe8a8f86511d678d031a022e48fff41c6a3e3b # the canonical bytes32 tokenId
  ```
- To get the address of the canonical token on Goerli, call the `getLocalAddress` function of Goerli's `TokenRegistry`
 
  ```bash
  cast call --chain goerli 0x51192fD98635FD32C2bfc0A2F4e362D864A4B8b1 "getLocalAddress(uint32,bytes32)(address)" "3331" "0x00000000000000000000000026fe8a8f86511d678d031a022e48fff41c6a3e3b" --rpc-url <goerli-rpc-url>
  ```

  Returns:

  ```bash
  0x7ea6eA49B0b0Ae9c5db7907d139D9Cd3439862a1 # the contract address of the canonical TestERC20
  ```

## Yalnızca hedef taraftaki hedef işlevini test etmek istersem ne olur?

Belirteç aktarımı söz konusu değilse, "transactingAssetId: address(0)" ve "amount: 0" öğelerini ayarlamanız yeterlidir.

## AMB sözleşmeleriyle ilgili herhangi bir şey yapmam gerekiyor mu?

Hayır, doğrudan AMB sözleşmelerini dağıtmanıza ve hatta bunlarla etkileşime girmenize gerek yoktur.

## Farklı alan kimliklerini nasıl bulabilirim?

[Alan Kimlikleri](./developers/testing-against-testnet#domain-ids) konusuna bakın.

## Connext sözleşmeleri nerede?

Bkz. [Dağıtılan Sözleşme Adresleri](./developers/testing-against-testnet#deployed-contract-addresses).

## Jeton çapraz zincirimi nasıl alırım?

Atmanız gereken birkaç adım var, bu yüzden lütfen yardım için bize ulaşın:

- Connext ekibi varlığınızı beyaz listeye alır (şimdilik bu gereklidir).
- Bazı varlıkları köprüden aktarıyorsunuz.
- Hedef varlık, benimsenen varlık olarak amb aromalı varlığa sahipse (mevcut bir havuz yok, henüz temsili bir varlık yok), Token Registry yeni bir jeton dağıtacak ve onu standart olarak ayarlayacaktır.

## Çağrı verilerinin boyut sınırları var mı?

(Yakında)

## Zincirimi Connext'e nasıl eklerim?

Connext, yeni zincirlere binmek için herhangi bir ücret talep etmez.
[Zincir Başlangıç Kılavuzumuzda](https://www.notion.so/connext/How-can-Connext-Bridge-add-my-Chain-fa8b43cac720467a88b5c94f81804091) yeni zincirleri işe alma sürecimiz hakkında bilgi edinebilirsiniz.

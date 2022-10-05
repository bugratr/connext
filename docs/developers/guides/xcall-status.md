---
sidebar_position: 6
---

# Bir xCall Çağrısını Takip Etme

Her "xcall", zincirler arası bir işlem aracılığıyla yaşam döngüsünü izlemek için kullanılabilen benzersiz bir "transferId" ile ilişkilendirilir.

## Alt grafları sorgulama

İşlem durumunu kontrol etmek için her zincirde barındırılan alt grafları sorgulayabilirsiniz.

| Zincir | Alt graf |
| --- | --- |
| Goerli | [v0-Goerli](https://thegraph.com/hosted-service/subgraph/connext/nxtp-amarok-runtime-v0-goerli) |
| Optimism-Goerli | [v0-Opt-Goerli](https://thegraph.com/hosted-service/subgraph/connext/amarok-runtime-v0-opt-goerli) |

1.  Connext sözleşmesiyle etkileşime giren işlem karmasını not edin

2. Kaynak etki alanı için barındırılan alt grafiğe gidin
     - İşlem hash'inize göre sorgulama
        
        ```graphql
        {
          originTransfers(
            where: {
              transactionHash: "<xcall_transaction_hash>"
            }
          ) {
            # Meta Data
            chainId
            transferId
            nonce
            to
            callData
            originDomain
            destinationDomain
            forceSlow
            receiveLocal
            recovery
            callback
            callbackFee
            # Asset Data
            transactingAsset
            transactingAmount
            status
          }
        }
        ```
        
3.  Hedef etki alanı için barındırılan alt grafiğe gidin
     - Origin etki alanı alt grafiğinden elde edilen 'transferId' ile sorgulama yapın
        
        ```graphql
        {
          destinationTransfers(
            where: {
              transferId: "<xcall_transfer_id>"
            }
          ) {
            # Meta Data
            chainId
            transferId
            nonce
            to
            callData
            originDomain
            destinationDomain
            forceSlow
            receiveLocal
            recovery
            callback
            callbackFee
            # Asset Data
            localAsset
            localAmount
            transactingAsset
            transactingAmount
            sponsorVaultRelayerFee
            # Executed event Data
            status
            routers {
              id
            }
            originSender
            # Executed Transaction
            executedCaller
            executedTransactionHash
            executedTimestamp
            executedGasPrice
            executedGasLimit
            executedBlockNumber
            # Reconciled Transaction
            reconciledCaller
            reconciledTransactionHash
            reconciledTimestamp
            reconciledGasPrice
            reconciledGasLimit
            reconciledBlockNumber
          }
        }
        ```

## Connextscan

Başka bir seçenek de, herhangi bir "transferId" durumunu izleyebilen [Connextscan](https://testnet.amarok.connextscan.io/) kullanmaktır.

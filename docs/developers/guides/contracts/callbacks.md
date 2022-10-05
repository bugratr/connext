---
sidebar_position: 1
id: callbacks
---

# Geri Çağrılar

## Giriş

Tanıttığımız harika bir özellik, kaynak etki alanındaki hedef etki alanından gelen aramaların sonuçlarına yanıt vermek için geri aramaları kullanma yeteneğidir. [Ayrıntılı özelliği buradan](https://github.com/connext/nxtp/discussions/883) okuyabilirsiniz. Bu örnek, Yetkilendirme örneğinin üzerine inşa edilecektir.

---

### Kaynak Kontrat

Geri arama işlemeyi etkinleştirmek için, kaynak etki alanındaki bazı sözleşmelerin 'ICallback' arabirimini uygulaması gerekir. Bu, ayrı bir sözleşme veya Kaynak sözleşmesinin kendisi olabilir.

Kaynak sözleşmemizin geri aramayı halletmesini sağlayacağız. Bunu yapmak için Source, 'ICallback' arayüzünü içe aktarmalı ve 'callback' parametresini bu arayüzü uygulayan sözleşmenin adresiyle değiştirmelidir. Ayrıca Connext PromiseRouter sözleşmesine bir referansa ihtiyacı olacaktır.

```solidity title="Source.sol"
import {IConnextHandler} from "nxtp/core/connext/interfaces/IConnextHandler.sol";
import {CallParams, XCallArgs} from "nxtp/core/connext/libraries/LibConnextStorage.sol";
//highlight-next-line
import {ICallback} from "nxtp/core/promise/interfaces/ICallback.sol";

contract Source {
  event CallbackCalled(bytes32 transferId, bool success, uint256 newValue);

  IConnextHandler public immutable connext;
  address public immutable promiseRouter;

  // The callback-handling contract needs a reference to the Connext PromiseRouter similarly 
  // to how the Target contract needs a reference to the Connext Executor. The contract 
  // implementing `callback` should only allow the Connext PromiseRouter to call it.
  modifier onlyPromiseRouter () {
    require(
      msg.sender == address(promiseRouter),
      "Expected PromiseRouter"
    );
    _;
  }

  constructor(
    IConnextHandler _connext, 
    //highlight-next-line
    address _promiseRouter
  ) {
    connext = _connext;
    //highlight-next-line
    promiseRouter = _promiseRouter;
  }

  function xChainUpdate(
    address to,
    uint32 originDomain,
    uint32 destinationDomain,
    uint256 newValue
  ) external payable {

    bytes4 selector = bytes4(keccak256("updateValue(uint256)"));
    bytes memory callData = abi.encodeWithSelector(selector, newValue);

    CallParams memory callParams = CallParams({
      to: to,
      callData: callData,
      originDomain: originDomain,
      destinationDomain: destinationDomain,
      agent: msg.sender, // address allowed to execute transaction on destination side in addition to relayers
      recovery: msg.sender, // fallback address to send funds to if execution fails on destination side
      forceSlow: true, // this must be true for authenticated calls
      receiveLocal: false, // option to receive the local bridge-flavored asset instead of the adopted asset
      //highlight-next-line
      callback: address(this), // this contract implements the callback
      callbackFee: 0, // fee paid to relayers for the callback; no fees on testnet
      relayerFee: 0, // fee paid to relayers for the forward call; no fees on testnet
      destinationMinOut: 0 // not sending funds so minimum can be 0
    });

    XCallArgs memory xcallArgs = XCallArgs({
      params: callParams,
      transactingAsset: address(0), // 0 address is the native gas token
      transactingAmount: 0, // not sending funds with this calldata-only xcall
      originMinOut: 0 // not sending funds so minimum can be 0
    });

    connext.xcall(xcallArgs);
  }

  // Required for contracts implementing the ICallback interface. 
  // This function is called to handle return data from the destination domain.
  //highlight-next-line
  function callback(
    bytes32 transferId,
    bool success,
    bytes memory data
  ) external onlyPromiseRouter {
    uint256 newValue = abi.decode(data, (uint256));
    emit CallbackCalled(transferId, success, newValue);
  }
```

Hedef etki alanından dönüş verileri geri arama ile birlikte gönderilir, böylece onunla istediğimizi yapabiliriz. Bu durumda, hedef etki alanında başarılı bir güncellemeyi doğrulamak için geri aramanın kaynak etki alanında kullanılabilmesi için gönderdiğimiz 'newValue' ile bir olayı ağa yayarız.

### Hedef Kontrat

Hedef tarafında, işlevin yalnızca bazı verileri döndürmesi gerekir.

```solidity
contract Target {
  uint256 public value;

  address public originContract; // the address of the source contract
  uint32 public originDomain; // the origin Domain ID
  address public executor; // the address of the Connext Executor contract

  // A modifier for authenticated functions.
  // Note: This is an important security consideration. If the target function
  //       is authenticated, it must check that the originating call is from
  //       the correct domain and contract. Also, the msg.sender must be the 
  //       Connext Executor address.
  modifier onlySource() {
    require(
      LibCrossDomainProperty.originSender(msg.data) == originContract &&
        LibCrossDomainProperty.origin(msg.data) == originDomain &&
        msg.sender == address(executor),
      "Expected origin contract on origin domain called by Executor"
    );
    _;
  }

  constructor(
    address _originContract, // address of the source contract
    uint32 _originDomain, // domain of the source contract
    IConnextHandler _connext // address of ConnextHandler
  ) {
    originContract = _originContract;
    originDomain = _originDomain;

    // Retrieve the address of the Connext Executor
    executor = _connext.executor();
  }

  function updateValue(uint256 newValue) 
    external onlySource  
    //highlight-next-line
    returns (uint256)
  {
    value = newValue;
    //highlight-next-line
    return newValue;
  }
}
```

Bu kadar! Connext şimdi geri arama yürütmesini, aktarıcılar tarafından işlenmek üzere kaynak etki alanına geri gönderecek.

**Not**: Başlangıç tarafı aktarıcılar henüz geri aramaları işlemek için ayarlanmamıştır. Bu kısa süre sonra eklenecektir!

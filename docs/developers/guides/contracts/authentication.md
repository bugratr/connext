---
sidebar_position: 1
id: authentication
---

# Yetkilendirme
## Giriş

Kimlik doğrulama, xApps oluştururken anlaşılması gereken kritik bir kavramdır. Akıllı sözleşmeler bağlamında, kimliği doğrulanmış bir çağrı, protokol geliştiricisi tarafından belirlenen izin kısıtlamalarını geçen bir çağrıdır. Çoğu durumda bu, belirli bir adres alt kümesinin belirli akıllı sözleşme işlevlerini çağırmasına izin veren bir değiştirici olarak kendini gösterir - başka bir deyişle, erişim kontrolünden bahsediyoruz.

Basit bir kimlik doğrulama örneği, yalnızca akıllı bir sözleşmenin sahibinin belirli görevleri gerçekleştirmesine izin veren bir "onlyOwner" değiştiricisidir. Bununla ilgili daha fazla bilgiyi [OpenZeppelin'in Sahip Olunabilir sözleşmelerinde](https://docs.openzeppelin.com/contracts/2.x/api/ownership) okuyabilirsiniz.

---

## Yetkiyi Kaldırma

Etki alanları arası çağrılar, "xcall" içinde kodlanmış "çağrı verileri" göndererek hedef taraf akıllı sözleşme işlevlerini hedefleyebilir. Kimliği doğrulanmamış işlevlerle açık ve herkes tarafından çağrılabilirler.

### Hedef Kontrat

Hedef etki alanında kimliği doğrulanmamış bir "updateValue" işlevine sahip bir hedef sözleşmemiz olduğunu varsayalım.  

```solidity
contract Target {
  uint256 public value;

  function updateValue(uint256 newValue) external {
    value = newValue;
  }
}
```

### Kaynak Kontrat

Kaynak sözleşmesi, Connext ile zincirler arası etkileşimi "xcall" aracılığıyla başlatır.

```solidity
import {IConnextHandler} from "nxtp/core/connext/interfaces/IConnextHandler.sol";
import {CallParams, XCallArgs} from "nxtp/core/connext/libraries/LibConnextStorage.sol";

contract Source {
  IConnextHandler public immutable connext;

  constructor(IConnextHandler _connext) {
    connext = _connext;
  }

  // The entry function that will update the value on the target contract.
  function xChainUpdate(
    address to, // the address of the target contract
    uint32 originDomain, // e.g. from Goerli (1735353714)
    uint32 destinationDomain, // to Optimism-Goerli (1735356532)
    uint256 newValue // value to update to
  ) external payable {

    // Encode the target function with its arguments
    bytes4 selector = bytes4(keccak256("updateValue(uint256)"));
    bytes memory callData = abi.encodeWithSelector(selector, newValue);

    CallParams memory callParams = CallParams({
      to: to,
      callData: callData,
      originDomain: originDomain,
      destinationDomain: destinationDomain,
      agent: msg.sender, // address allowed to execute transaction on destination side in addition to relayers
      recovery: msg.sender, // fallback address to send funds to if execution fails on destination side
      forceSlow: false, // option to force slow path instead of paying 0.05% fee on fast liquidity transfers
      receiveLocal: false, // option to receive the local bridge-flavored asset instead of the adopted asset
      callback: address(0), // zero address because we don't expect a callback
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
}
```

Bu sözleşmeler dağıtıldıktan sonra, herhangi bir kişi hedef etki alanının hedef sözleşmesinde depolanan değeri değiştirmek için kaynak etki alanından kaynak sözleşmede 'xChainUpdate'i çağırabilir!

---

## Yetkilendirme
Kimliği doğrulanmış işlevlerle xApp geliştiricisi, izin kontrollerini dikkatli bir şekilde uygulamalıdır. Hedef işlevi **_yalnızca kaynak sözleşmeyle_** çağrılabilir hale getirerek bunun nasıl çalıştığını görelim.

### Hedef Kontrat

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

  function updateValue(uint256 newValue) external onlySource {
    value = newValue;
  }
}
```

Hedef işlevde 'onlyExecutor' özel değiştiricisine dikkat edin. Kaynak çağrıdan gönderilen 'OriginSender' ve 'origin' (etki alanı), yapıcıda ayarlanan Origin değerleriyle eşleşip eşleşmediğini kontrol eder. Ayrıca, "msg.sender"ın Connext Executor sözleşmesi olduğundan emin olmalıdır - aksi takdirde, herhangi bir arama sözleşmesi, beklediğimiz kaynak bilgisini yanıltabilir. "xcall"ın başlangıç bilgisi, "msg.data"nın Connext'in "LibCrossDomainProperty" kitaplık yöntemlerine geçirilmesiyle elde edilebilir.

### Kaynak Kontrat

Kaynak sözleşme, "forceSlow: true" dışında, yukarıdaki kimliği doğrulanmamış örnekle tamamen aynıdır. Bu parametreyle ilgili notlara bakın [buradan](../xcall-params.md).

```solidity
import {IConnextHandler} from "nxtp/core/connext/interfaces/IConnextHandler.sol";
import {CallParams, XCallArgs} from "nxtp/core/connext/libraries/LibConnextStorage.sol";

contract Source {
    IConnextHandler public immutable connext;

  constructor(IConnextHandler _connext) {
    connext = _connext;
  }

  // The entry function that will update the value on the target contract.
  function xChainUpdate(
    address to, // the address of the target contract
    uint32 originDomain, // e.g. from Goerli (1735353714)
    uint32 destinationDomain, // to Optimism-Goerli (1735356532)
    uint256 newValue // value to update to
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
      //highlight-next-line
      forceSlow: true, // this must be true for authenticated calls
      receiveLocal: false, // option to receive the local bridge-flavored asset instead of the adopted asset
      callback: address(0), // zero address because we don't expect a callback
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
}
```

Artık hedef sözleşme yalnızca kaynak sözleşmenin `xChainUpdate` işlevinden bir zincirler arası çağrı ile güncellenebilir!

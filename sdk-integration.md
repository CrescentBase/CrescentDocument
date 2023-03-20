# SDK Integration

## Crescent Wallet SDKs

Crescent Wallet supports multi-platform SDK.&#x20;

| SDK         | Status     | Document                                                              |
| ----------- | ---------- | --------------------------------------------------------------------- |
| Android SDK | Done       | [Download](https://github.com/CrescentBase/CrescentDocument/releases) |
| iOS SDK     | Done       | [Download](https://github.com/CrescentBase/CrescentDocument/releases) |
| Unity SDK   | Developing | Developing                                                            |

### &#x20;Android SDK

#### **Installation and Usage**

1. Put `crescent-sdk.aar` into the `libs` folder of the project.
2. Initialize the SDK

```javascript
CrescentConfigure config = new CrescentConfigure();
config.style  =  "";  //Custom Styles
config.paymasterUrl = "https://..."; //Paymaster Address
CrescentSdk.getInstance().init(config)；
```

3. Connect wallet

```javascript
CrescentSdk.getInstance().connect(this, new ConnectCallback() {
    @Override
    public void onConnectSuccess(UserInfo info) {
String email = info.email
String address = info.address
    }


	@Override
	    public void onConnectFaill() {
	    }
	});
```

4. Send transaction

```javascript
    // TransactionInfo Related type definition
	public class TransactionInfo {
	    private String chainId;
	    private String to;
	    private String value;
	    private String data;
	}
 
	//Determining if the transaction will be successful before trading
	if (CrescentSdk.getInstance().isConnected()) {
	TransactionInfo info = new TransactionInfo(from, to, value, data);
	CrescentSdk.getInstance().sendTransaction(info, 
	new  TransactionCallback() {
	                 @Override
	                 public void onSendSuccess(TransactioinResult result) {
	String hash = result.hash;
	                 }
	 
	                 @Override
	                 public void onSendFail() {
                      }
	});
	}
```

5. Disconnect wallet

```javascript
CrescentSdk.getInstance().disconnect();
```

### &#x20;iOS SDK

#### **Installation and Usage**

1. Add the `CrescentSDK.framework` to the project.
2. Initialize the SDK

Notes

The Framworks-Libraries, and Embedded Content should be swicth to"Embed\&Sign" on Xcods.

```swift
var config = CrescentConfigure()
     config.style = "" //Custom Styles
     config.paymasterUrl = "https://..." //Paymaster Address
     CrescentSDK.config(configure: config)
```

3. Connect wallet

```swift
CrescentSDK.connect(connectSuccessBlock: { userinfo in
      	 let email = userinfo.email;
         let address = userinfo.address;
     }, connectFailBlock: {
         print("connectFailBlock")
    })
```

4. Send transaction

```swift
var tx = TransactionInfo()
    tx.chainId= ""
    tx.to = ""
    tx.value = ""
    tx.data = "";
    CrescentSDK.sendTransaction(info: tx, sendSuccessBlock: { transactionResult in
    		let hash = transactionResult.hash;
    }, sendFailBlock: {
         print("sendFailBlock")
    })
```

5. Disconnect wallet

```swift
CrescentSDK.disconnect()
```

## General Usage

### Define Your Paymaster

1. Deploy your own Paymaster contract

[DeployedBytecode](https://etherscan.io/tx/0x6e7bbc06925f0f86c90b217909bdcdda4163d65c2222dd287357b3d67bcec7eb) and the corresponding [PaymasterProxy Code](https://github.com/CrescentBase/account-abstraction/blob/main/contracts/core/CrescentPaymasterProxy.sol)

2. Deploy the signature server

* Example server URL：https://...

```json
method: POST
Accept: application/json
Content-Type: application/json
Body: { op:{...}, email, public_key, chain_id }
Response: { data:{ .... }  } 
```

* Examples of op and data content

```json
"sender": "0x......6B",
"add_owner": "0x......82",
"nonce": "0x......f0",
"initCode": "0x......12",
"callData": "0x......00",
"callGas": "2000000",
"verificationGas": "1000000",
"preVerificationGas": "21000",
"maxFeePerGas": "1199460050",
"maxPriorityFeePerGas": "1000000000",
"paymaster": "0x......70",
"paymasterData": "0x......1c",
"signature": "0x"
```

* Example of signature server

<pre class="language-solidity"><code class="lang-solidity">//"ethers": "^5.4.1",
import {defaultAbiCoder, keccak256} from "ethers/lib/utils.js";
//"web3": "^1.3.4",
import Web3 from "web3";
 
export function getHash(op) {
    return keccak256(defaultAbiCoder.encode([
        'address', // sender
        'uint256', // nonce
        'bytes32', // initCode
        'bytes32', // callData
        'uint256', // callGas
        'uint', // verificationGas
        'uint', // preVerificationGas
        'uint256', // maxFeePerGas
        'uint256', // maxPriorityFeePerGas
        'address' // paymaster
    ], [
        op.sender,
        op.nonce,
        keccak256(op.initCode),
        keccak256(op.callData),
        op.callGas,
        op.verificationGas,
        op.preVerificationGas,
        op.maxFeePerGas,
        op.maxPriorityFeePerGas,
        op.paymaster
    ]));
}

export function sign(privateKey, data) {
    const signature = new Web3().eth.accounts.sign(data, privateKey);
    return signature.signature;
}
 
export function signOp(op, privateKey) {
    const data = getHash(op);
    return sign(privateKey, data);
}
 
<strong>
</strong>export function test() {
    const op = { ... };
    // privateKey is the corresponding private key of the EOA that deployed Paymaster contract
    const privateKey = "....";
    //your paymaster address
    op.paymaster = "0x...";
    const signature = signOp(op, privateKey);
    op.paymasterData = signature;
    return op;
}
</code></pre>

3. SDK call example

&#x20;refer to [Initialize the SDK](sdk-integration.md#installation-and-usage)

```
config.paymasterUrl = "https://..."; //Paymaster Address
```

### Customise  UI

Crescent provides a full-featured UI customisation ability. Please reference the following doc.

{% content-ref url="sdk-ui-doc.md" %}
[sdk-ui-doc.md](sdk-ui-doc.md)
{% endcontent-ref %}

##

&#x20;

&#x20;

&#x20;

&#x20;

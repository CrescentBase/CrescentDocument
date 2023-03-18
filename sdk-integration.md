# SDK Integration

## Crescent Wallet SDKs

Crescent Wallet supports multi-platform SDK.

&#x20;

| SDK         | Status     | Document                                    |
| ----------- | ---------- | ------------------------------------------- |
| Android SDK | Done       | [Start now](sdk-integration.md#android-sdk) |
| iOS SDK     | Done       | [Start now](sdk-integration.md#ios-sdk)     |
| Unity SDK   | Developing | Developing                                  |

### &#x20;Android SDK

#### **Installation and Usage**

1. Put crescent-sdk.aar into the libs folder of the project.
2. Initialize the SDK

```javascript
CrescentConfigure config = new CrescentConfigure();
config.style  =  “”;  //Custom Styles
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
	    private String from;
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

1. Add the CrescentSDK.framework to the project.
2. Initialize the SDK

```swift
var config = CrescentConfigure()
     config.style = "" //Custom Styles
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
    tx.from = ""
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

#### DeployedBytecode

Refer to Input Data [https://etherscan.io/tx/0x6e7bbc06925f0f86c90b217909bdcdda4163d65c2222dd287357b3d67bcec7eb](https://etherscan.io/tx/0x6e7bbc06925f0f86c90b217909bdcdda4163d65c2222dd287357b3d67bcec7eb)

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

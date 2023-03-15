# Technical Architecture

Crescent is an open-source wallet, and this chapter provides an analysis of the wallet's basic infrastructure for research and learning purposes.&#x20;

Note that it's not required to read this chapter to integrate Crescent SDK.

### Bundler

The Bundler calls EntryPoint to package user transactions, validates UserOperation (UO) off-chain, removes UOs that do not meet requirements or have issues, packages compliant UOs and submits them to the blockchain.&#x20;

The main features include:

* eth\_sendUserOperation：send transactions.
* eth\_estimateUserOperationGas：gas evaluation.
* eth\_getUserOperationReceipt：Get transaction history.

```solidity
async handleMethod(method, params) {
        let result;
        switch (method) {
            case 'eth_chainId':
                // eslint-disable-next-line no-case-declarations
                const { chainId } = await this.provider.getNetwork();
                result = chainId;
                break;
            case 'eth_supportedEntryPoints':
                result = await this.methodHandler.getSupportedEntryPoints();
                break;
            case 'eth_sendUserOperation':
                result = await this.methodHandler.sendUserOperation(params[0], params[1]);
                break;
            case 'eth_estimateUserOperationGas':
                result = await this.methodHandler.estimateUserOperationGas(params[0], params[1]);
                break;
            case 'eth_getUserOperationReceipt':
                result = await this.methodHandler.getUserOperationReceipt(params[0]);
                break;
            case 'eth_getUserOperationByHash':
                result = await this.methodHandler.getUserOperationByHash(params[0]);
                break;
            case 'web3_clientVersion':
                result = this.methodHandler.clientVersion();
                break;
            default:
                throw new utils_2.RpcError(`Method ${method} is not supported`, -32601);
        }
        return result;
    }
```

### &#x20;EntryPoint Contract

EntryPoint is the core entry point for all functionalities. Each project deploys their own EntryPoint. Bundler, Wallet, and Paymaster all need to work around EntryPoint.

The main features include:

* simulateValidation：Simulate user transactions and validate UO off-chain.

```solidity
    function simulateValidation(UserOperation calldata userOp) external returns (uint256 preOpGas, uint256 prefund) {
        uint256 preGas = gasleft();


        bytes32 requestId = getRequestId(userOp);
        (prefund,,) = _validatePrepayment(0, userOp, requestId);
        preOpGas = preGas - gasleft() + userOp.preVerificationGas;


        require(msg.sender == address(0), "must be called off-chain with from=zero-addr");
    }
```

&#x20;it will create a wallet address for users according EIP-2470 if user's wallet address has not been created while in simulateValidation.

* handleOPs：Package compliant UOs and submit them on-chain.

```solidity
    function handleOps(UserOperation[] calldata ops, address payable beneficiary) public {


        uint256 opslen = ops.length;
        UserOpInfo[] memory opInfos = new UserOpInfo[](opslen);
}
```

### &#x20;Paymaster Contract

Paymaster has the following functions and features：

* Pay gas fees to EntryPoint.
* Only respond to messages from EntryPoint.
* Confirm the intention to EntryPoint that pay for a certain UO.
* stake in EntryPoint to become a paymasterer；

Note: Only after passing the validation of validatePaymasterUserOp, Paymaster will pay for users.

```solidity
function validatePaymasterUserOp(UserOperation calldata userOp, bytes32 requestId, uint256 maxCost) external view returns (bytes memory context);
```

### &#x20;PaymasterProxy Contract

The main contract of Paymaster，responsible for upgrading Paymaster contract.

```solidity
    function upgradeDelegate(address newImplementation) public {
        require(msg.sender == _getAdmin());
        _upgradeTo(newImplementation);
    }
```

### &#x20;Wallet Contract

This is the wallet user uses. The contract address is the same as wallet address.

It has the following features and functions

* Pay gas fees to EntryPoint.
* Only respond to messages from EntryPoint.
* Execute specific transaction contents from EntryPoint.
* addOwner: add a user device after passing DKIM verification.

```solidity
function addOwner(
        address owner,
        VerifierInfo calldata info
    ) external onlyEntryPoint {
        bytes memory modulus = dkimManager.dkim(info.ds);
        require(modulus.length != 0, "Not find modulus!");
        require(IVerifier(dkimVerifier).verifier(owner, modulus, info), "Verification failed!");
        require(allOwner.length < type(uint16).max, "Too many owners");
        uint16 index = uint16(allOwner.length + 1);
        allOwner.push(owner);
        owners[owner] = index;
    }
```

* \_validateSignature：Verify if the signature is consistent, and only proceed with subsequent operations if it is consistent.

```solidity
 function _validateSignature(UserOperation calldata userOp, bytes32 requestId) internal view override {
        //0x350bddaa addOwner
        bool isAddOwner = bytes4(userOp.callData) == 0x350bddaa;
        if (userOp.initCode.length != 0 && !isAddOwner) {
            revert("wallet: not allow");
        }


        if (!isAddOwner) {
            bytes32 hash = requestId.toEthSignedMessageHash();
            address signatureAddress = hash.recover(userOp.signature);
            require(owners[signatureAddress] > 0, "wallet: wrong signature");
        }
    }
```

* Normal receiving sending function.

### &#x20;**WalletProxy** Contract

Main contract of Wallet responsible for upgrading wallet contract, supporting manual and automatic upgrades. WalletController contract is required for automatic upgrades. Note: Manual upgrade as default .

```solidity
    function upgradeDelegate(address newDelegateAddress) public {
        require(msg.sender == _getAdmin());
        _upgradeTo(newDelegateAddress);
    }


    function setAutoUpdateImplementation(bool value) public {
        require(msg.sender == _getAdmin());
        StorageSlot.getBooleanSlot(_AUTO_UPDATE_SLOT).value = value;
    }
```

### &#x20;**WalletController** Contract

Work with WalletProxy to upgrade wallet.

```solidity
    function setImplementation(address _implementation) public onlyOwner {
        implementation = _implementation;
    }
```

## &#x20;DKIM

### DKIM Detail

By clicking "View Original" or "View Source" in any email, you can directly view the .eml files which includes the DKIM-Signature field. (For more details, please refer to the [DKIM\_WIKI](https://en.wikipedia.org/wiki/DomainKeys\_Identified\_Mail).)

> DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed; d=foxmail.com;
>
> s=s201512; t=1670419235;
>
> bh=9RqYI6fxZOUZAYcxZV4SvznReZm2Mn7vMx5y5+asYAM=;
>
> h=From:To:Subject:Date;
>
> b=A3Kfzk0KcOfQhiEGJZ5KUpb3ItszuNBCSJ08hhgaGUIuglV4QaTm9BVH9pDmljKl+AIzS4nRZjYFLiRQWN8ZaYh7edwCp7BAV2l2ei27+mlP/7nsCapEFdbM1cyNBoR8lGwJMkMh3HGhCPMLH8c2GQVx5GxdOj+NLVQZGNVrHwk=

* v: DKIM version.
* a: Cryptographic algorithm used for the signature.
* c: Standardization algorithm for the email header and body, divided into simple and relaxed. The standardization algorithm is a way to handle spaces, carriage returns, line feeds, and other content.
* d: Domain of the sender's service provider.
* s: Selector, customized by the sender's service provider. Multiple selectors can correspond to a single domain. One selector corresponds to a pair of public and private keys. The public key corresponding to domain+selector can be obtained by querying the DNS server.
* t: UNIX timestamp.
* bh: Body hash, the hash of the body encoded in base64
* h: Header fields that are signed, chosen by the sender's service provider
* b: Signature encoded in base64

DKIM verification ensures that the email is from the sender and has not been tampered with.

### &#x20;DKIM Authoritative Record Contract

Used to record DKIM-related data. Among them:&#x20;

* upgradeDKIM: Records and saves DKIM data..&#x20;
* dkim: Provides interface data for the DKIM verification contract.&#x20;

```solidity
contract DKIMManager is Ownable {


    mapping (bytes => bytes) private allDkim;


    constructor() {}


    function upgradeDKIM(bytes memory name, bytes memory _dkim) public onlyOwner {
        allDkim[name] = _dkim;
    }


    function dkim(bytes memory name) public view returns (bytes memory) {
        return allDkim[name];
    }


}
```

### &#x20;DKIM Verifie Contract

All encryption and verification-related functions are in this contract. User operations such as creating and changing devices require calling the verifier function.Verification success proves that the action was initiated by the account's owner.&#x20;

Among them

* verifyProof: Responsible for zero-knowledge proof verification (ZKP).&#x20;
* equalBase64: Responsible for verifying whether the bh content in DKIM is correct.&#x20;
* containsAddress: Responsible for verifying whether it is a user-initiated behavior.&#x20;
* sha256Verify: Verifies whether the b parameter in DKIM correct or not .

```solidity
   function verifier(
        address publicKey,
        bytes memory modulus,
        VerifierInfo calldata info
    ) external view returns (bool)  {
        uint[] memory input = getInput(info.hmua, info.bh, info.base);
        //ZKP Verifier
        if (!verifyProof(info.a, info.b, info.c, input)) {
            return false;
        }
        //bh(bytes) == base64(sha1/sha256(Canon-body))
        if (!equalBase64(info.bh, info.body)) {
            return false;
        }
        //Operation ∈ Canon-body
        if (!containsAddress(publicKey, info.body)) {
            return false;
        }
        //b == RSA(base)
        if (!sha256Verify(info.base, info.rb, info.e, modulus)) {
            return false;
        }
        return true;
    }
```

# Features

#### Keyless. No seed phrase and private key.

Users log in with email accounts. Seed phrases and private keys are hidden from users' workflow.

#### DKIM validation on chain

Domainkeys Identified Mail (DKIM) is an email authentication method designed to detect whether the sender address, subject, body, etc. of an email has been modified (email spoofing) .The mail will be signed by the sender's mail service provider when outrebounding. After the mail receiving server receives the DKIM-signed mail, it can verify whether the mail is indeed from the sender and the content has not been tampered with.&#x20;

Crescent implements dkim digital signature verification through smart contracts. realizes verification of email content and transactions on chain, and then manages contract accounts through email.

To avoid exposing user information while verifying ownership of email address, Crescent use ZKP technology to ensure user privacy and security in a decentralised way.

<figure><img src=".gitbook/assets/DKIM验证 (1).jpg" alt=""><figcaption><p>DKIM verify on internet vs on blockchain</p></figcaption></figure>

#### More ways to log in

Crescent will developing more ways to login such as OAuth，etc.&#x20;

#### Gasless

Gasless is crucial to the popularity of Web3.

As a smart contract wallet that supports EIP 4337 protocol, Paymaster will pay gas for users to achieve a gas free experience when users start a transaction.&#x20;

Paymaster not only supports payment in native tokens for gas, but also support other tokens or payments such as NFT.&#x20;

We provide a part of the basic paymaster template, which developers can deploy and customize the following parameters:

* The type of tokens to pay.
* Set up a contract whitelist, when the user interacts with the contract in whitelist, the gas will be paid for the user automatically.
* Customize gas policy, provide gas discounts for new users, etc.

#### Multi-chain support

**Now support the following chain**

Ethereum

Arbitrum

BSC

Polygon

&#x20;

## The differences between EOA wallet?

The core advantages are improving user conversion rate and asset security.&#x20;

1.Crescent contract wallet is created without seed phrase and can be login directly via email. For Web2 users, it greatly simplifies the operation process and lowers the cognitive threshold, but also prevents irreversible losses caused by improper management of seed phrase.&#x20;

2.Crescent can realize gasless experience that avoiding loss of users before they go deep into the game.

Gasless experience is particularly improved on games support multi chains, users no need to purchase corresponding Gas tokens for each chain.

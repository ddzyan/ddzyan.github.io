---
title: ethersjs安装和Wallet-API介绍
date: 2018-09-20 00:57:37
categories: 
- ETH
tags:
- ETH
- 区块链
---
## 简介
ethers.js库是以太坊的一个完整的JavaScript库。

参考网址：https://docs.ethers.io/ethers.js/html/getting-started.html
### 安装在Nodejs
```bash
npm install --save ethers
```

导入
```js
var ethers = require('ethers');
```
---

参考网址：https://docs.ethers.io/ethers.js/html/api-wallet.html
### wallet
一个钱包管理一个私有/公共密钥对，用于对事务进行加密签名并在Ethereum网络上证明所有权。

#### 创建wallet对象
##### 私钥创建
```js
var privateKey = "0x0123456789012345678901234567890123456789012345678901234567890123";
var wallet = new Wallet(privateKey);

console.log("Address: " + wallet.address);
// "Address: 0x14791697260E4c9A71f18484C9f997B308e59325"
```

##### 随机创建
```js
var wallet = Wallet.createRandom();
console.log("Address: " + wallet.address);
// "Address: ... this will be different every time ..."
```

<!--more-->

##### 加密钱包
```js
var data = {
    id: "fb1280c0-d646-4e40-9550-7026b1be504a",
    address: "88a5c2d9919e46f883eb62f7b8dd9d0cc45bc290",
    Crypto: {
        kdfparams: {
            dklen: 32,
            p: 1,
            salt: "bbfa53547e3e3bfcc9786a2cbef8504a5031d82734ecef02153e29daeed658fd",
            r: 8,
            n: 262144
        },
        kdf: "scrypt",
        ciphertext: "10adcc8bcaf49474c6710460e0dc974331f71ee4c7baa7314b4a23d25fd6c406",
        mac: "1cf53b5ae8d75f8c037b453e7c3c61b010225d916768a6b145adf5cf9cb3a703",
        cipher: "aes-128-ctr",
        cipherparams: {
            iv: "1dcdf13e49cea706994ed38804f6d171"
         }
    },
    "version" : 3
};

var json = JSON.stringify(data);
var password = "foo";
Wallet.fromEncryptedWallet(json, password).then(function(wallet) {
    console.log("Address: " + wallet.address);
    // "Address: 0x88a5C2d9919e46F883EB62F7b8Dd9d0CC45bc290"
});
```

##### 助记词创建
```js
var mnemonic = "radar blur cabbage chef fix engine embark joy scheme fiction master release";
var wallet = Wallet.fromMnemonic(mnemonic);

console.log("Address: " + wallet.address);
// "Address: 0xaC39b311DCEb2A4b2f5d8461c1cdaF756F4F7Ae9"
```

##### 账号密码创建
```js
var username = "support@ethers.io";
var password = "password123";
Wallet.fromBrainWallet(username, password).then(function(wallet) {
    console.log("Address: " + wallet.address);
    // "Address: 0x7Ee9AE2a2eAF3F0df8D323d555479be562ac4905"
});
```

#### wallet属性
- address ：地址
- privateKey ：私钥
- provider ：连接的提供商，允许钱包连接到以太坊网络以查询其状态并发送事务
- getAddress()：返回address
- sign(transaction)：对事务进行签名并以十六进制字符串的形式返回已签名的事务
- signMessage(message) ：签名消息并以十六进制字符串返回签名。
- encrypt(password[ , options ] [ , progressCallback]) :返回Promise对象，将钱包加密为Secret Storage JSON Wallet;

##### 例子
###### 签名事务，并且发送网络
```js
var ethers = require('ethers');
var Wallet = ethers.Wallet;
var utils = ethers.utils;
var providers = ethers.providers;

var privateKey = "0x0123456789012345678901234567890123456789012345678901234567890123";
var wallet = new Wallet(privateKey);

console.log('Address: ' + wallet.address);
// "Address: 0x14791697260E4c9A71f18484C9f997B308e59325".

var transaction = {
    nonce: 0,
    gasLimit: 21000,
    gasPrice: utils.bigNumberify("20000000000"),

    to: "0x88a5C2d9919e46F883EB62F7b8Dd9d0CC45bc290",

    value: utils.parseEther("1.0"),
    data: "0x",

    //这确保了事务不能在不同的网络上重播
    chainId: providers.networks.homestead.chainId

};

var signedTransaction = wallet.sign(transaction);

console.log(signedTransaction);
// "0xf86c808504a817c8008252089488a5c2d9919e46f883eb62f7b8dd9d0cc45bc2" +
//   "90880de0b6b3a7640000801ca0d7b10eee694f7fd9acaa0baf51e91da5c3d324" +
//   "f67ad827fbe4410a32967cbc32a06ffb0b4ac0855f146ff82bef010f6f2729b4" +
//   "24c57b3be967e2074220fca13e79"

// 这现在可以发送到Ethereum网络
var provider = providers.getDefaultProvider();
provider.sendTransaction(signedTransaction).then(function(hash) {
    console.log('Hash: ' + hash);
    // Hash:
});
```

###### 加密
```js
var password = "password123";

function callback(percent) {
    console.log("Encrypting: " + parseInt(percent * 100) + "% complete");
}

var encryptPromise = wallet.encrypt(password, callback);

encryptPromise.then(function(json) {
    console.log(json);
});
```

#### 区块链操作
这些操作要求钱包附加一个网络提供者。

##### 属性
- getBalance([ blockTag ]) ：在blockTag中返回一个带有钱包余额（作为一个BigNumber，在wei中）的Promise。默认值：blockTag =“最新”
- getTransactionCount( [blockTag]) ：返回一个Promise，其中包含此帐户在blockTag上发送的事务数（也称为nonce）。默认值：blockTag =“最新”
- estimateGas(transaction ) ：返回一个Promise，其中包含事务的估计成本（in gas, as a BigNumber）
- sendTransaction(transaction ) ：将事务发送到网络并返回带有事务详细信息的Promise。 强烈建议省略transaction.chainId，它将由提供者填写。
- send ( addressOrName, amountWei [ , options ] ) ：将金额发送到网络上的地址或名称，并返回带有交易详细信息的承诺。

##### 例子
###### 查询网络
```js
var privateKey = '0x0123456789012345678901234567890123456789012345678901234567890123';
var wallet = new ethers.Wallet(privateKey);
wallet.provider = ethers.providers.getDefaultProvider();

var balancePromise = wallet.getBalance();

balancePromise.then(function(balance) {
    console.log(balance);
});

var transactionCountPromise = wallet.getTransactionCount();

transactionCountPromise.then(function(transactionCount) {
    console.log(transactionCount);
});
```

###### 交易
```js
var privateKey = '0x0123456789012345678901234567890123456789012345678901234567890123';
var wallet = new ethers.Wallet(privateKey);
wallet.provider = ethers.providers.getDefaultProvider();

// 必须先将金额转换为最小单位 wei
// parseEther 可以将 ether 转换成 wei
var amount = ethers.utils.parseEther('1.0');

var address = '0x88a5c2d9919e46f883eb62f7b8dd9d0cc45bc290';
var sendPromise = wallet.send(address, amount);

sendPromise.then(function(transactionHash) {
    console.log(transactionHash);
});


// These will query the network for appropriate values
var options = {
    //gasLimit: 21000
    //gasPrice: utils.bigNumberify("20000000000")
};

var promiseSend = wallet.send(address, amount, options);

promiseSend.then(function(transaction) {
    console.log(transaction);
});
```

###### 发送复杂交易
```js
var privateKey = '0x0123456789012345678901234567890123456789012345678901234567890123';
var wallet = new ethers.Wallet(privateKey);
wallet.provider = ethers.providers.getDefaultProvider('ropsten');

var transaction = {
    // Recommendation: omit nonce; the provider will query the network
    // nonce: 0,

    // Gas Limit; 21000 will send ether to another use, but to execute contracts
    // larger limits are required. The provider.estimateGas can be used for this.
    gasLimit: 1000000,

    // Recommendations: omit gasPrice; the provider will query the network
    //gasPrice: utils.bigNumberify("20000000000"),

    // Required; unless deploying a contract (in which case omit)
    to: "0x88a5C2d9919e46F883EB62F7b8Dd9d0CC45bc290",

    // Optional
    data: "0x",

    // Optional
    value: ethers.utils.parseEther("1.0"),

    // Recommendation: omit chainId; the provider will populate this
    // chaindId: providers.networks.homestead.chainId
};

// Estimate the gas cost for the transaction
//var estimateGasPromise = wallet.estimateGas(transaction);

//estimateGasPromise.then(function(gasEstimate) {
//    console.log(gasEstimate);
//});

// Send the transaction
var sendTransactionPromise = wallet.sendTransaction(transaction);

sendTransactionPromise.then(function(transactionHash) {
    console.log(transactionHash);
});
```

##### 解析签名后的交易
- parseTransaction ( hexStringOrArrayish )： 将原始hexStringOrArrayish解析为Transaction。

```js
// Mainnet:
var ethers = require('ethers');
var Wallet = ethers.Wallet;
var utils = ethers.utils;
var privateKey = '0x0123456789012345678901234567890123456789012345678901234567890123';
var wallet = new ethers.Wallet(privateKey);

var raw = "0xf87083154262850500cf6e0083015f9094c149be1bcdfa69a94384b46a1f913" +
            "50e5f81c1ab880de6c75de74c236c8025a05b13ef45ce3faf69d1f40f9d15b007" +
            "0cc9e2c92f"

var transaction = {
    nonce: 1393250,
    gasLimit: 21000,
    gasPrice: utils.bigNumberify("20000000000"),

    to: "0xc149Be1bcDFa69a94384b46A1F91350E5f81c1AB",

    value: utils.parseEther("1.0"),
    data: "0x",

    // This ensures the transaction cannot be replayed on different networks
    chainId: ethers.providers.networks.homestead.chainId
};

var signedTransaction = wallet.sign(transaction);
var transaction = Wallet.parseTransaction(signedTransaction);

console.log(transaction);
// { nonce: 1393250,
//   gasPrice: BigNumber { _bn: <BN: 4a817c800> },
//   gasLimit: BigNumber { _bn: <BN: 5208> },
//   to: '0xc149Be1bcDFa69a94384b46A1F91350E5f81c1AB',
//   value: BigNumber { _bn: <BN: de0b6b3a7640000> },
//   data: '0x',
//   v: 38,
//   r: '0x3cf1f5af8bd11963193451096d86635aed589572c184ac8696dd99c9c044ded3',
//   s: '0x08c52dbf1383492c72598511bb135179ec93b062032d2a0d002214644ba39a2c',
//   chainId: 1,
//   from: '0x14791697260E4c9A71f18484C9f997B308e59325' }
```

##### 验证消息
- verifyMessage ( message , signature )：返回带有签名的消息的地址。
```js
var signature = "0xddd0a7290af9526056b4e35a077b9a11b513aa0028ec6c9880948544508f3c63" +
                  "265e99e47ad31bb2cab9646c504576b3abc6939a1710afc08cbf3034d73214b8" +
                  "1c";
var address = Wallet.verifyMessage('hello world', signature);
console.log(address);
// '0x14791697260E4c9A71f18484C9f997B308e59325'
```
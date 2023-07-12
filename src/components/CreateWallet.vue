<template>
    <div id="box">
        <h1>助记词钱包</h1>
        <p>助记词：{{ mnemonic }}</p>
        <p>路径：m/44'/60'/0'/0/0</p>
        <p>地址：0xe0e94d8f8f183a6078bb267adf8bec1eced03d48</p>
        <p>私钥：0x79ee94f9cdca87b19a4b77a01ecc689f318957e25098e6a94f5b83a161e8b2a7</p>
        <p>keyStore：{"version":3,"id":"fbf78b94-39af-4ea3-88b0-37708b95fc3f","address":"e0e94d8f8f183a6078bb267adf8bec1eced03d48","crypto":{"ciphertext":"b5a13fafe647e22ad093005c4b3ecebc069c58c783c9ecf06fbeb616c4af619b","cipherparams":{"iv":"a34f6deae85ace3b51449ce6c9cd665f"},"cipher":"aes-128-ctr","kdf":"scrypt","kdfparams":{"n":8192,"r":8,"p":1,"dklen":32,"salt":"3638d15e28cac69ef113f61b13ef5adaa88d779c01336b5f48f8cbca27e2803e"},"mac":"f857448e75c8e556a579b98a6e92f9d0f923acb7cc0718abffd52a79765e5d79"}}
        </p>
    </div>
</template>

<script setup>
import { ref } from "vue";
import * as bip39 from "bip39";
import ethWallet, { hdkey } from "ethereumjs-wallet";
import Web3 from "web3";

// 创建助记词
// let mnemonic = bip39.generateMnemonic();
let mnemonic = ref("maximum eyebrow mandate tag matrix fossil trick crazy choose phone focus cherry");
console.log(mnemonic.value);
// 初始化数据
const init = async () => {
    // 生成秘钥对 ketPair
    let seed = await bip39.mnemonicToSeed(mnemonic.value)
    // console.log(seed)

    const hdWallet = hdkey.fromMasterSeed(seed);
    const ketPair = hdWallet.derivePath("m/44'/60'/0'/0/0")
    // console.log(ketPair)

    // 获取私钥
    // 1.获取钱包对象
    const wallet = ketPair.getWallet(ketPair)
    // console.log(wallet)
    // 2.获取钱包地址
    const lowerCaseAddress = wallet.getAddressString()
    console.log(`钱包地址：${lowerCaseAddress}`)
    // 3.获取钱包的校验地址
    const checkAddress = wallet.getChecksumAddressString()
    console.log(`钱包校验地址：${checkAddress}`)
    // 4.获取私钥
    const privateKey = wallet.getPrivateKeyString()
    const privateKeyString = wallet.getPrivateKey().toString("hex")
    console.log(`钱包私钥：${privateKey}`)
    // console.log(privateKeyString)

    // 导出keyStore
    // 1.web3js
    // var web3 = new Web3(Web3.giveProvider || 'wss://goerli.infura.io/ws/v3/6c6f9d00b3114190a8c586a9a9ca9d3b')
    // const keyStore = await web3.eth.accounts.encrypt("0x79ee94f9cdca87b19a4b77a01ecc689f318957e25098e6a94f5b83a161e8b2a7", '111111')
    // console.log(`keyStoreWeb3：`, keyStore)

    // 2.wallet对象方法(不会出现：Cannot read properties of undefined (reading 'importKey')错误)
    const keyStore2 = await wallet.toV3("111111")
    console.log(`keyStoreWallet：`, keyStore2)


    // 通过keyStore获取私钥
    // 1.web3js
    // const res = await web3.eth.accounts.decrypt('{"version":3,"id":"fbf78b94-39af-4ea3-88b0-37708b95fc3f","address":"e0e94d8f8f183a6078bb267adf8bec1eced03d48","crypto":{"ciphertext":"b5a13fafe647e22ad093005c4b3ecebc069c58c783c9ecf06fbeb616c4af619b","cipherparams":{"iv":"a34f6deae85ace3b51449ce6c9cd665f"},"cipher":"aes-128-ctr","kdf":"scrypt","kdfparams":{"n":8192,"r":8,"p":1,"dklen":32,"salt":"3638d15e28cac69ef113f61b13ef5adaa88d779c01336b5f48f8cbca27e2803e"},"mac":"f857448e75c8e556a579b98a6e92f9d0f923acb7cc0718abffd52a79765e5d79"}}', '111111')
    // console.log(`通过keyStore获取私钥Web3：`, res)

    // 2.wallet对象方法
    const wallet2 = await ethWallet.fromV3(keyStore2, '111111')
    const key = wallet2.getPrivateKeyString()
    console.log(`通过keyStore获取私钥wallet：`,key)

    // 通过私钥获取地址
    const privateKey2 = Buffer(privateKeyString,'hex');
    console.log(`privateKey2：`,privateKey2)
    const wallet3 = await ethWallet.fromPrivateKey(privateKey2);
    console.log(wallet3)
    const lowerCaseAddress2 = wallet3.getAddressString();
    console.log(lowerCaseAddress2)
}
init();
</script>

<style lang="less" scoped>
#box {
    width: 98%;
    margin: 10px auto;
    border: 1px grey solid;
    padding: 1%;
    box-sizing: border-box;
}
</style>
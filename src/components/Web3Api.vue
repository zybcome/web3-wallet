<template>
  <div id="box">
    <div id="left">
      <!-- 原生转账 -->
      <h1>账户信息</h1>
      <van-divider></van-divider>
      <p>地址：{{ address }}</p>
      <!-- <p>私钥：{{ privateKey }}</p> -->
      <p>余额：{{ mountEth }} Eth</p>
      <p>余额：{{ mountWei }} Wei</p>
      <h1>转账操作</h1>
      <van-divider></van-divider>
      <van-field v-model="value" label="转账金额：" placeholder="请输入转账金额（Eth）" />
      <van-field v-model="toInput" label="转账地址：" placeholder="请输入钱包地址或Ens域名" />
      <div class="btn">
        <van-button type="primary" @click="transaction()">立即转账</van-button>
      </div>
      <div v-if="transactionFlage">
        <p>转账次数：{{ nonce }}</p>
        <p>预估Gas费用：{{ gasPrice }} Wei</p>
        <p>转账金额：{{ valueWei }} Wei</p>
        <p>转账地址：{{ toInput }}</p>
        <p>转账地址：{{ to }}</p>
        <p>Gas费用：{{ gas }} Wei</p>
        <p>交易明细：<span v-if="pushHerf == '加载中'">加载中</span><a target="_blank" v-if="pushHerf != '加载中'"
            :href="pushHerf">在区块链浏览器上查看</a></p>
      </div>
    </div>
    <!-- 调用小狐狸进行转账 -->

    <div id="right">
      <h1>MetaMask钱包</h1>
      <van-divider></van-divider>
      <div class="btn"><van-button v-if="addressMateMask == ''" type="primary" @click="MetaMaskLogin()">登录</van-button>
      </div>
      <div v-if="addressMateMask != ''">
        <p>MetaMask：{{ addressMateMask }}</p>
        <p>余额：{{ mountEthMateMask }} Eth</p>
        <p>余额：{{ mountWeiMateMask }} Wei</p>
        <h1>转账操作</h1>
        <van-divider></van-divider>
        <van-field v-model="valueMateMask" label="转账金额：" placeholder="请输入转账金额（Eth）" />
        <van-field v-model="toMateMaskInput" label="转账地址：" placeholder="请输入钱包地址或Ens域名" />
        <div class="btn">
          <van-button type="primary" @click="transactionMetaMask()">立即转账</van-button>
          <van-button style="margin-left: 5px;" type="danger" @click="MetaMaskLoginOut()">退出登录</van-button>
        </div>
        <div v-if="transactionFlageMateMask">
          <p>转账次数：{{ nonceMateMask }}</p>
          <p>预估Gas费用：{{ gasPriceMateMask }} Wei</p>
          <p>转账金额：{{ valueWeiMateMask }} Wei</p>
          <p>转账地址：{{ toMateMaskInput }}</p>
          <p>转账地址：{{ toMateMask }}</p>
          <p>Gas费用：{{ gasMateMask }} Wei</p>
          <p>交易明细：<span v-if="pushHerfMateMask == '加载中'">加载中</span><a target="_blank" v-if="pushHerfMateMask != '加载中'"
              :href="pushHerfMateMask">在区块链浏览器上查看</a></p>
        </div>
      </div>
    </div>


  </div>
</template>


<script setup>
import { ref } from "vue";
import Web3 from "web3";
import Tx from "ethereumjs-tx"
import { showDialog } from 'vant';
var web3 = new Web3(Web3.giveProvider || 'wss://goerli.infura.io/ws/v3/6c6f9d00b3114190a8c586a9a9ca9d3b')

// MetaMask钱包

const transactionFlageMateMask = ref(false);//控制交易明细显示隐藏
const addressMateMask = ref("")
const mountEthMateMask = ref("加载中");
const mountWeiMateMask = ref("加载中");
const nonceMateMask = ref("加载中");
const gasPriceMateMask = ref("加载中");
const valueMateMask = ref("");
const valueWeiMateMask = ref("加载中");
const gasMateMask = ref("加载中");
const toMateMaskInput = ref("");
// const toMateMaskInput = ref("0x8574081cdc140ecDB9Cc6860Ece5A2563A8A4384");
const toMateMask = ref("");
const pushHerfMateMask = ref("加载中");
let web3MateMask;
console.log(window.ethereum.selectedAddress)

const MetaMaskLogin = async () => {
  if (window.ethereum) {
    web3MateMask = new Web3(window.ethereum);
    window.ethereum.enable().then(res => {
      console.log(res)
      getBalanceFun();
    }).catch(err => {
      showDialog({
        message: err.message,
        theme: 'round-button',
      }).then(() => {
        // on close

      });
    })
  }
}

const getBalanceFun = async () => {
  const accounts = await web3MateMask.eth.getAccounts();
  addressMateMask.value = accounts[0]

  web3MateMask.eth.getBalance(addressMateMask.value).then(res => {
    mountWeiMateMask.value = res;
    // 单位转换
    // wei=>eth
    mountEthMateMask.value = web3MateMask.utils.fromWei(res, "ether");
  })

}

const MetaMaskInit = async () => {
  if (window.ethereum && window.ethereum.selectedAddress) {
    web3MateMask = new Web3(window.ethereum);
    getBalanceFun();
  }
}

MetaMaskInit()

const MetaMaskLoginOut = async () => {
  addressMateMask.value = "";
  transactionFlageMateMask.value = false;
}
const transactionMetaMask = async () => {
  transactionFlageMateMask.value = true;
  // 获取账户交易次数
  nonceMateMask.value = await web3MateMask.eth.getTransactionCount(addressMateMask.value)
  // 获取预估燃料费
  gasPriceMateMask.value = await web3MateMask.eth.getGasPrice();
  // 转账金额
  valueWeiMateMask.value = web3MateMask.utils.toWei(valueMateMask.value, "ether")
  if (!web3MateMask.utils.isHexStrict(toMateMaskInput.value)) {
    toMateMask.value = await web3MateMask.eth.ens.getAddress(toMateMaskInput.value).catch(err => {
      showDialog({
        message: err.message,
        theme: 'round-button',
      }).then(() => {
        // on close

      });

    })
  } else {
    toMateMask.value = toMateMaskInput.value
  }
  console.log(toMateMask.value)
  const transaction = {
    from: addressMateMask.value,
    to: toMateMask.value,
    nonce: parseInt(nonceMateMask.value),
    gasPrice: parseInt(gasPriceMateMask.value),
    value: parseInt(valueWeiMateMask.value),
    data: "0x0000",
  };
  gasMateMask.value = await web3MateMask.eth.estimateGas(transaction)
    .catch(err => {
      transactionFlageMateMask.value = false;
      showDialog({
        message: err.message,
        theme: 'round-button',
      }).then(() => {
        // on close

      });
    });
  if (!gasMateMask.value) {
    return
  }
  transaction.gas = parseInt(gasMateMask.value);
  console.log(transaction)
  web3MateMask.eth.sendTransaction(transaction)
    .on('transactionHash', function (hash) {
      console.log('交易哈希：', hash);
      MetaMaskInit()
      pushHerfMateMask.value = `https://goerli.etherscan.io/tx/${hash}`
    })
    .on('confirmation', function (confirmationNumber, receipt) {
      console.log('确认次数：', confirmationNumber);
      console.log('交易回执：', receipt);
    })
    .on('error', function (error) {
      transactionFlageMateMask.value = false;
      showDialog({
        message: error.message,
        theme: 'round-button',
      }).then(() => {
        // on close

      });
      // console.error('转账错误：', error);
    })
    .catch(err => {
      transactionFlageMateMask.value = false;
      showDialog({
        message: err.message,
        theme: 'round-button',
      }).then(() => {
        // on close

      });
    });
}


// 原生钱包

// 创建账号
// const account = web3.eth.accounts.create('123456');
// console.log(account.address)
// console.log(account.privateKey)

const address = ref('0x8574081cdc140ecDB9Cc6860Ece5A2563A8A4384')
// const address = ref('0x44710f1dE5c62F2732AA65B290a869FA6370a170')
const privateKeyWallet = ref('0x4dbb1316b75aa738828d38bb84cb6c2a5c6241544eb86703ee6250b280439ef2')
// const privateKey = ref('')
const mountWei = ref("加载中");
const mountEth = ref("加载中");
const init = () => {
  web3.eth.getBalance(address.value).then(res => {
    mountWei.value = res;
    // 单位转换
    // wei=>eth
    mountEth.value = web3.utils.fromWei(res, "ether");
  })
}
init();

// eth=>wei
// const num1 = web3.utils.toWei("0.3", "ether")



// 转账
const transactionFlage = ref(false);//控制交易明细显示隐藏
const nonce = ref("加载中");
const gasPrice = ref("加载中");
const value = ref("");
const valueWei = ref("加载中");
const gas = ref("加载中");
// const toInput = ref("0x44710f1dE5c62F2732AA65B290a869FA6370a170");
const toInput = ref("");
const to = ref("");
const pushHerf = ref("加载中");
const transaction = async () => {
  // 显示交易明细
  transactionFlage.value = true;
  // 1.构建转账参数
  // 获取账户交易次数
  nonce.value = await web3.eth.getTransactionCount(address.value)
  // 获取预估燃料费
  gasPrice.value = await web3.eth.getGasPrice()
  // 转账金额
  valueWei.value = web3.utils.toWei(value.value, "ether")
  if (!web3.utils.isHexStrict(toInput.value)) {
    to.value = await web3.eth.ens.getAddress(toInput.value)
  } else {
    to.value = toInput.value
  }
  const rawTx = {
    from: address.value,
    to: to.value,
    nonce: parseInt(nonce.value),
    gasPrice: parseInt(gasPrice.value),
    value: parseInt(valueWei.value),
    data: "0x0000",
  }
  console.log(rawTx)

  // 2.生成serializedTx
  // 转化私钥
  const privateKey = Buffer(privateKeyWallet.value.slice(2), "hex");
  console.log(privateKey)
  // Gas估算
  gas.value = await web3.eth.estimateGas(rawTx)
    .catch(err => {
      transactionFlageMateMask.value = false;
      showDialog({
        message: err.message,
        theme: 'round-button',
      }).then(() => {
        // on close

      });
    });
  if (!gas.value) {
    return
  }
  rawTx.gas = parseInt(gas.value);
  console.log(rawTx)
  // 私钥加密
  const tx = new Tx(rawTx)
  tx.sign(privateKey)
  // 生成serializedTx
  const serializedTx = `0x${tx.serialize().toString("hex")}`;
  console.log(serializedTx);

  // 3.开始转账
  const trans = web3.eth.sendSignedTransaction(serializedTx);
  trans.on('error', function (error) {
    transactionFlage.value = false;
    showDialog({
      message: error.message,
      theme: 'round-button',
    }).then(() => {
      // on close

    });
    // console.error('转账错误：', error);
  })
  trans.catch(err => {
    transactionFlage.value = false;
    showDialog({
      message: err.message,
      theme: 'round-button',
    }).then(() => {
      // on close

    });
  });
  // 提交转账
  trans.on("transactionHash", (txHash) => {
    console.log(`交易哈希值: ${txHash}`);
    init();
    pushHerf.value = `https://goerli.etherscan.io/tx/${txHash}`;

    // 第n个节点确认
    trans.on("confirmation", (confirmationNumber, receipt) => {
      console.log(`第${confirmationNumber.confirmations}个节点确认`)
      if (confirmationNumber.confirmations >= 1) {
        // 交易已经被至少1个确认，可以尝试获取交易回执信息
        web3.eth.getTransactionReceipt(txHash)
          .then((receipt) => {
            console.log("交易回执信息：", receipt);
            // 可以进一步处理交易回执信息
          })
          .catch((error) => {
            console.error("获取交易回执信息时出错：", error);
          });
      }
    });
  });

  // 第一个节点确认
  trans.on("receipt", (ret) => {
    console.log(`确认节点:${ret.confirmations}`)
  })
}
</script>

<style lang="less">
#box {
  width: 100%;
  margin: 0 auto;
  // padding: 10px;
  overflow: hidden;

  h1 {
    text-align: center;
  }

  #left {
    border: 1px grey solid;
    padding: 5px;
    float: left;
    width: 48%;
    margin-left: 1%;
  }

  #right {
    border: 1px grey solid;
    padding: 5px;
    float: right;
    width: 48%;
    margin-right: 1%;
  }

  .btn {
    text-align: center;
    margin-top: 10px;
  }
}
</style>
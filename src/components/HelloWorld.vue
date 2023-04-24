<script setup lang="ts">
import Web3 from "web3"
import { AbiItem } from 'web3-utils'
import{TransactionConfig} from "web3-core"
import { ref ,reactive} from 'vue'
import MyCoinAbi from "../static/MyCoin.json"
// defineProps<{ msg: string }>()

const readInput= ref("0xc3c14e8Cd7cD3fba816cf2ce4EBD10b6dDeB3D3a")
const toAddressInput = ref("0xcaD75EADAf24F41d6274E129d7aE54764d7cd8E7")
const toAmountInput =ref("0.0001")
let web3:Web3 ;
const blockInfo = reactive({
  chainId:{
    key:"CHainId:",
    value:"-"
  },
  blockNumber:{
    key:"BlockNumber:",
    value:"-"
  },
  blockTimeStamp:{
    key:"BlockTimeStamp:",
    value:"-"
  },
  currentAccount:{
    key:"CurrentAccount:",
    value:"-"
  },
  currentBalances:{
    key:"CurrentBalances:",
    value:"-"
  },
})

const readInfo = reactive({
  symbol:{
    key:"Symbol:",
    value:"-"
  },
  totalSupply:{
    key:"TotalSupply:",
    value:"-"
  },
  balance:{
    key:"Balance:",
    value:"-"
  },
})
const transferInfo = reactive({
  estimateGas:{
    key:"EstimateGas:",
    value:"-"
  },
  gasPrice:{
    key:"GasPrice:",
    value:"-"
  },
  tx:{
    key:"Tx:",
    value:"-"
  },
})

async function connetWallet(){    
      //获取web3的实例
      if (window.ethereum) {
        try {
            //唤起用户同意
         await window.ethereum.enable(); 
         web3 = new Web3(Web3.givenProvider || "ws://localhost:8545")
        //  console.log(web3);
         } catch (error) {
           //用户取消链接  
           console.log("取消链接");
           
         }
    //旧浏览器可以通过这种方式拿到 
    }else if(window.web3){
      web3 = new Web3(window.web3)
    }else{
        alert("please install wallet")
    }
    
    blockInfo.chainId.value = (await web3.eth.getChainId()).toString();
    blockInfo.blockNumber.value = (await web3.eth.getBlockNumber()).toString();
    let block = await web3.eth.getBlock(blockInfo.blockNumber.value);
    blockInfo.blockTimeStamp.value = block.timestamp.toString();
    blockInfo.currentAccount.value = (await web3.eth.getAccounts())[0];
    blockInfo.currentBalances.value = web3.utils.fromWei(await web3.eth.getBalance(blockInfo.currentAccount.value)) 
}

async function read(){
  let instance = new web3.eth.Contract(MyCoinAbi.abi  as AbiItem[], readInput.value);
  //调用合约中的symbol
  readInfo.symbol.value  = await instance.methods.symbol().call();
  readInfo.totalSupply.value  =  web3.utils.fromWei(await instance.methods.totalSupply().call());
  readInfo.balance.value = web3.utils.fromWei(await instance.methods.balanceOf(blockInfo.currentAccount.value).call());
}
async function transfer(){
  let instance = new web3.eth.Contract(MyCoinAbi.abi  as AbiItem[], readInput.value);
  //调用合约进行交易
    //1.构造data
    //web3.utils.toWei(amount) 将用户传入的数据转化成带18个零的decimal类型
    //.encodeABI() 这个包装的data要符合ABI编码规范
    let transferData = instance.methods.transfer(toAddressInput.value,web3.utils.toWei(toAmountInput.value)).encodeABI();
     //2.预执行一下看看消耗多少gas 同时也可以看到交易是否可以被成功执行 避免gas浪费
    
     transferInfo.estimateGas.value = web3.utils.fromWei((await web3.eth.estimateGas({
        to:readInput.value,//这里的to 是合约地址  与合约交互
        data:transferData,
        from:blockInfo.currentAccount.value,
        value:'0x0'//不交易eth 所以填0
    })).toString());
    //获取当前gasprice
    transferInfo.gasPrice.value = web3.utils.fromWei((await web3.eth.getGasPrice()).toString());
    //该账户的交易nonce
    let nonce:number = await web3.eth.getTransactionCount(blockInfo.currentAccount.value);
    //要交易的数据组装
    
    let rawTransaction:TransactionConfig = {
        from:blockInfo.currentAccount.value,
        to:readInput.value,
        nonce:Number(web3.utils.toHex(nonce)),
        gasPrice:web3.utils.toWei(transferInfo.gasPrice.value),
        gas:Number(web3.utils.toWei(transferInfo.estimateGas.value))*2,//预留一部分额度 担心预估的gas不够交易
        // value:'0x0',
        data:transferData,
        chainId:Number(blockInfo.chainId.value),
    }
    //进行交易签名 和 发起交易
    web3.eth.sendTransaction(rawTransaction).on("transactionHash",function (hash:string) {
        //监听交易的回调
        // console.log("txHash",hash);
        transferInfo.tx.value = hash
    })
}
</script>

<template>

  <div class="card">
    <el-button type="primary" size="default" @click="connetWallet">Connet Wallet</el-button>
    <!-- <div>{{ chainId }}</div> -->
    <el-row :gutter="12" class="info-card">
    <el-col :span="8">
      <div class="info-card-left">{{ blockInfo.chainId.key }}</div>
    </el-col>
    <el-col :span="16">
      <!-- <el-card shadow="hover"> Hover </el-card> -->
      <div class="info-car-right">{{ blockInfo.chainId.value }}</div>
    </el-col>
    <el-col :span="8">
      <div class="info-card-left">{{ blockInfo.blockNumber.key }}</div>
    </el-col>
    <el-col :span="16">
      <!-- <el-card shadow="hover"> Hover </el-card> -->
      <div class="info-car-right">{{ blockInfo.blockNumber.value }}</div>
    </el-col>
    <el-col :span="8">
      <div class="info-card-left">{{ blockInfo.blockTimeStamp.key }}</div>
    </el-col>
    <el-col :span="16">
      <!-- <el-card shadow="hover"> Hover </el-card> -->
      <div class="info-car-right">{{ blockInfo.blockTimeStamp.value }}</div>
    </el-col>
    <el-col :span="8">
      <div class="info-card-left">{{ blockInfo.currentAccount.key }}</div>
    </el-col>
    <el-col :span="16">
      <!-- <el-card shadow="hover"> Hover </el-card> -->
      <div class="info-car-right">{{ blockInfo.currentAccount.value }}</div>
    </el-col>
    <el-col :span="8">
      <div class="info-card-left">{{ blockInfo.currentBalances.key }}</div>
    </el-col>
    <el-col :span="16">
      <!-- <el-card shadow="hover"> Hover </el-card> -->
      <div class="info-car-right">{{ blockInfo.currentBalances.value }}</div>
    </el-col>
  </el-row>
  <el-row :gutter="12" class="read-input">
    <el-col :span="4" >
      <div class="prefx-info">contractAddress:</div>
    </el-col>
    <el-col :span="14" >
      <el-input v-model="readInput" placeholder="Please input" />
    </el-col>
    <el-col :span="5">
      <el-button type="primary" size="default" @click="read">Read</el-button>
    </el-col>
  </el-row>

  <el-row :gutter="12" class="info-card">
    <el-col :span="8">
      <div class="info-card-left">{{ readInfo.symbol.key }}</div>
    </el-col>
    <el-col :span="16">
      <!-- <el-card shadow="hover"> Hover </el-card> -->
      <div class="info-car-right">{{ readInfo.symbol.value }}</div>
    </el-col>
    <el-col :span="8">
      <div class="info-card-left">{{ readInfo.totalSupply.key }}</div>
    </el-col>
    <el-col :span="16">
      <!-- <el-card shadow="hover"> Hover </el-card> -->
      <div class="info-car-right">{{ readInfo.totalSupply.value }}</div>
    </el-col>
    <el-col :span="8">
      <div class="info-card-left">{{ readInfo.balance.key }}</div>
    </el-col>
    <el-col :span="16">
      <!-- <el-card shadow="hover"> Hover </el-card> -->
      <div class="info-car-right">{{ readInfo.balance.value }}</div>
    </el-col>

  </el-row>
  <el-row :gutter="12" class="read-input">
    <el-col :span="3" >
      <div class="prefx-info">toAddress:</div>
    </el-col>
    <el-col :span="12" >
      <el-input v-model="toAddressInput" placeholder="Please input" />
    </el-col>
    <el-col :span="3" >
      <div class="prefx-info">Amount:</div>
    </el-col>
    <el-col :span="3" >
      <el-input v-model="toAmountInput" placeholder="Please input" />
    </el-col>
    <el-col :span="3">
      <el-button type="primary" size="default" @click="transfer">Transfer</el-button>
    </el-col>
  </el-row>
  <el-row :gutter="12" class="info-card">
    <el-col :span="4">
      <div class="info-card-left">{{ transferInfo.estimateGas.key }}</div>
    </el-col>
    <el-col :span="20">
      <!-- <el-card shadow="hover"> Hover </el-card> -->
      <div class="info-car-right">{{ transferInfo.estimateGas.value }}</div>
    </el-col>
    <el-col :span="4">
      <div class="info-card-left">{{ transferInfo.gasPrice.key }}</div>
    </el-col>
    <el-col :span="20">
      <!-- <el-card shadow="hover"> Hover </el-card> -->
      <div class="info-car-right">{{ transferInfo.gasPrice.value }}</div>
    </el-col>
    <el-col :span="4">
      <div class="info-card-left">{{ transferInfo.tx.key }}</div>
    </el-col>
    <el-col :span="20">
      <!-- <el-card shadow="hover"> Hover </el-card> -->
      <div class="info-car-right">{{ transferInfo.tx.value }}</div>
    </el-col>

  </el-row>

  </div>


</template>

<style scoped>
.read-the-docs {
  color: #888;
}
.info-card{
border: 1px #eef0f4 solid;
border-radius: 4px;
margin: 10px 0;
padding: 10px;
padding-top: 0;
text-align: end;
}
.info-card-left{
  padding-top: 10px ;
  padding-right: 10px;
}
.info-car-right{
padding-top: 10px ;
}
.read-input{
  padding: 10px;
}
.prefx-info{
  /* vertical-align: middle; */
  height: 32px;
  line-height: 32px;
}
</style>

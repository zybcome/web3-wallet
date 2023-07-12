<template>
  <h1>{{ string }}</h1>
  <Web3Api></Web3Api>
</template>

<script setup>
// import { ref } from "vue";
// import Web3 from "web3";
import Web3Api from "./components/Web3Api.vue";
// const string = ref("hello web3")
// var web3 = new Web3(Web3.giveProvider || 'wss://mainnet.infura.io/ws/v3/6c6f9d00b3114190a8c586a9a9ca9d3b')
// console.log(web3)
</script>

<style>
h1 {
  color: red;
}
</style>
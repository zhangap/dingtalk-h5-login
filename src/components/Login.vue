<template>
  <div class="box">
    <v-card>
      <v-card-title>钉钉测试登录</v-card-title>
      <v-card-text>
        <pre>
        authCode结果：{{JSON.stringify(result, null, '\t')}}
        </pre>

      </v-card-text>
      <v-card-text>
        <pre>
          登录结果：{{JSON.stringify(loginResult, null, '\t')}}
        </pre>
      </v-card-text>
      <v-divider></v-divider>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="primary" @click="onGetCode" :loading="loading">获取code</v-btn>
      </v-card-actions>
    </v-card>
  </div>

</template>
<script setup>
import * as dd from 'dingtalk-jsapi';
import {ref} from "vue";
const loading = ref(false);
const result = ref({});
const loginResult = ref({});
import axios from "axios";

function onGetCode() {
  loading.value = true;
  dd.getAuthCode({
    corpId: 'ding291246203d94b630a39a90f97fcb1e09'
  }).then(res => {
    result.value = res;
    login();
  }).finally(() => {
    loading.value = false;
  })
}

function login() {
  axios.post(`http://192.168.0.200:8080/login?authCode=${result.value.code}`).then(res => {
    loginResult.value = res.data;
  }).catch(err => {
    loginResult.value = err;
  })

}
</script>
<style scoped lang="scss">
.box {
  padding: 10px;
}
</style>

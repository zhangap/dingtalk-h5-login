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
    <v-divider></v-divider>
    <v-card class="code-box">
      <v-card-title>二维码</v-card-title>
      <v-card-text class="code-wrap">
        <div id="code"></div>
      </v-card-text>
    </v-card>
  </div>

</template>
<script setup>
import * as dd from 'dingtalk-jsapi';
import {nextTick, onMounted, ref} from "vue";
const loading = ref(false);
const result = ref({});
const loginResult = ref({});
import axios from "axios";
import qs from 'qs'

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

onMounted(async () => {
  await nextTick();
  beforeCreate().then(console.log).catch(createQRCode)
})


//创建前钩子函数
function beforeCreate() {
  return new Promise((resolve, reject) => {
    let searchStr = location.search;
    searchStr = searchStr.startsWith('?') ? searchStr.substring(1) : searchStr;
    const searchParams =  qs.parse(searchStr);
    if(searchParams['code']) {
      result.value = {code : searchParams['code']};
      login();
      resolve('已经登录')
    } else {
      reject();
    }
  })

}

const appId = 'dingywbnoqy9xakbtot4';
const currentUrl = encodeURIComponent(location.href);
const goto = encodeURIComponent(`https://oapi.dingtalk.com/connect/oauth2/sns_authorize?appid=${appId}&response_type=code&scope=snsapi_login&state=STATE&redirect_uri=`+ currentUrl)
function createQRCode() {
  addEvent();
  const obj = window.DDLogin({
    id:"code",
    goto, //请参考注释里的方式
    style: "border:none;background-color:#ffffff;",
    width : "365",
    height: "400"
  });
}

function addEvent() {
  const handleMessage = function (event) {
    const origin = event.origin;
    console.log("origin", event.origin);
    if( origin === "https://login.dingtalk.com" ) { //判断是否来自ddLogin扫码事件。
      debugger;
      const loginTmpCode = event.data;
      //获取到loginTmpCode后就可以在这里构造跳转链接进行跳转了
      console.log("loginTmpCode", loginTmpCode);
      location.href = `https://oapi.dingtalk.com/connect/oauth2/sns_authorize?appid=${appId}&response_type=code&scope=snsapi_login&state=STATE&redirect_uri=REDIRECT_URI&loginTmpCode=${loginTmpCode}`;
    }
  };
  if (typeof window.addEventListener != 'undefined') {
    window.addEventListener('message', handleMessage, false);
  } else if (typeof window.attachEvent != 'undefined') {
    window.attachEvent('onmessage', handleMessage);
  }
}


</script>
<style scoped lang="scss">
.box {
  padding: 10px;
}
.code-box {
  margin-top: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

.code-wrap {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 800px;
  height: 600px;
  border: 1px solid red;
}

</style>

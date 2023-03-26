---
title: ToolKit
date: 2023-03-26 11:32:32
---

<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/4.0.0/crypto-js.min.js"></script>

<!-- 网易云音乐 -->
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=440357173&auto=1&height=66"></iframe>



### Crypto Tool

<div id="app">
  <input type="text" id="input" v-model="message" placeholder="Enter text to encrypt or decrypt"><br>
  <input type="text" id="key" v-model="key" placeholder="Enter encryption key">
  <button @click="encrypt()">Encrypt</button>
  <button @click="decrypt()">Decrypt</button>
  <p>cryptoMsg: {{ cryptoMsg }}</p>
</div>

### Help Link
- [cmd5](https://www.cmd5.org/): This site provides online MD5 / sha1/ mysql / sha256 encryption and decryption services. We have a super huge database with more than 90T data records.

<script type="text/javascript" language="JavaScript">
var app = new Vue({
  el: '#app',
  data: {
    message: 'MySpringCloud Project',
    key: '',
    cryptoMsg: ''
  },
  watch: {},
  computed: {},
  created: function () {
    console.log("Vue created");

  },
  mounted: function () {
    console.log("Vue mounted");
      
  },
  methods: {
    getTitle: function (title) {
      if (title == null) {
        return 'Null';
      }
      return title == '' ? 'Blank' : title;
    },
    encrypt: function () {
      if (this.message && this.key) {
        let encrypted = CryptoJS.AES.encrypt(this.message, this.key);
        this.cryptoMsg = encrypted.toString();
      } else {
        alert('Error: message or key is empty: {}, {}', this.message, this.key);
      }
    },
    decrypt: function () {
      if (this.message && this.key) {
        let decrypted = CryptoJS.AES.decrypt(this.message, this.key);
        this.cryptoMsg = decrypted.toString();
      } else {
        alert('Error: message or key is empty: {}, {}', this.message, this.key);
      }
    }
  }
})
</script>

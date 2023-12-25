<template>
  <body>
    <div class="loginBox">
      <h2>柚子システム</h2>
      <div @submit.prevent="login">
        <div class="item">
          <input type="text" v-model="username" required />
          <label for="">ユーザー名</label>
        </div>
        <div class="item">
          <input type="password" v-model="password" required />
          <label for="">パスワード</label>
        </div>
        <div class="item">
          <input type="text" v-model="emailCode" required />
          <label>検証コード</label>
        </div>

        <div class="bigbtn">
          <button class="btn" @click="emailCodeApi()">
            メ ー ル
            <span></span>
            <span></span>
            <span></span>
            <span></span>
          </button>
          <button class="btn" @click="login()">
            ログイン
            <span></span>
            <span></span>
            <span></span>
            <span></span>
          </button>
        </div>
      </div>
    </div>
  </body>
</template>
    
<script>
export default {
  props: ['setMainFlag'],
  name: "login-in",
  components: {},
  data: function () {
    let validateName = (rule, value, callback) => {
      if (!value) {
        return callback(new Error("用户名不能为空"));
      } else {
        callback();
      }
    };
    let validatePassword = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请输入密码"));
      } else {
        callback();
      }
    };
    return {
      loginForm: {
        // 登录用户名密码
        username: "akali",
        password: "0000",
        emailCode: "",
      },
      rules: {
        username: [
          { validator: validateName, message: "请输入用户名", trigger: "blur" },
        ],
        password: [
          {
            validator: validatePassword,
            message: "请输入密码",
            trigger: "blur",
          },
        ],
      },
    };
  },
  mounted() {},
  methods: {
    // ログイン
    async login() {
      var url = "https://www.iuui.cloud/yin/api" + "/user/login/status";
      try {
        const formData = new FormData();
        formData.append("username", this.username);
        formData.append("password", this.password);
        formData.append("emailCode", this.emailCode);

        const response = await axios.post(url, formData, {
          headers: {
            "Content-Type": "multipart/form-data",
          },
        });
        if (response.data.code == 200) {
            localStorage.setItem('token',response.data.data.token);
            this.updateMainFlag();
        }
        alert(response.data.msg);
      } catch (error) {
        console.error(error);
      } finally {
      }
    },

    updateMainFlag() {
    this.setMainFlag(true);
    },

    // メ ー ル
    async emailCodeApi() {
      var url = "https://www.iuui.cloud/yin/api" + "/send/self/code";
      try {
        const response = await axios.post(url, {});

        alert(response.data.msg);
      } catch (error) {
        console.error(error);
      }
    },
  },
};
</script>
    
<style  scoped>
* {
  margin: 0;
  padding: 0;
}

a {
  text-decoration: none;
}

.bigbtn button {
}

input,
.btn {
  background: transparent;
  border: 0;
  outline: none;
}

body {
  height: 100vh;
  background: linear-gradient(#141e30, #243b55);
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 16px;
  color: #03e9f4;
}

.loginBox {
  width: 400px;
  height: 420px;
  background-color: #0c1622;
  margin: 100px auto;
  border-radius: 10px;
  box-shadow: 0 15px 25px 0 rgba(0, 0, 0, 0.6);
  padding: 40px;
  box-sizing: border-box;
}
@media screen and (max-width: 1025px) {
  .loginBox {
    width: 300px;
    height: 420px;
    background-color: #0c1622;
    margin: 100px auto;
    border-radius: 10px;
    box-shadow: 0 15px 25px 0 rgba(0, 0, 0, 0.6);
    padding: 40px;
    box-sizing: border-box;
  }
}

h2 {
  text-align: center;
  color: aliceblue;
  margin-bottom: 30px;
  font-family: "Courier New", Courier, monospace;
}

.item {
  height: 45px;
  border-bottom: 1px solid #fff;
  margin-bottom: 40px;
  position: relative;
}

.item input {
  font-size: 13pt;
  width: 100%;
  height: 100%;
  color: #fff;
  padding-top: 20px;
  box-sizing: border-box;
}

.item input:focus + label,
.item input:valid + label {
  top: 0px;
  font-size: 8pt;
}

.item label {
  position: absolute;
  left: 0;
  top: 5px;
  transition: all 0.5s linear;
}

.btn {
  padding: 10px 20px;
  color: #03e9f4;
  position: relative;
  overflow: hidden;
  text-transform: uppercase;
  letter-spacing: 2px;
  left: 18%;
}
@media screen and (max-width: 1025px) {
  .btn {
    padding: 10px 20px;
    color: #03e9f4;
    position: relative;
    overflow: hidden;
    text-transform: uppercase;
    letter-spacing: 2px;
    left: 2%;
  }
}

.btn:hover {
  border-radius: 5px;
  color: #fff;
  background: #03e9f4;
  box-shadow: 0 0 5px 0 #03e9f4, 0 0 25px 0 #03e9f4, 0 0 50px 0 #03e9f4,
    0 0 100px 0 #03e9f4;
  transition: all 1s linear;
}

.btn > span {
  position: absolute;
}

.btn > span:nth-child(1) {
  width: 100%;
  height: 2px;
  background: -webkit-linear-gradient(left, transparent, #03e9f4);
  left: -100%;
  top: 0px;
  animation: line1 2.3s linear infinite;
}

@keyframes line1 {
  50%,
  100% {
    left: 100%;
  }
}

.btn > span:nth-child(2) {
  width: 2px;
  height: 100%;
  background: -webkit-linear-gradient(top, transparent, #03e9f4);
  right: 0px;
  top: -100%;
  animation: line2 2.3s 0.25s linear infinite;
}

@keyframes line2 {
  50%,
  100% {
    top: 100%;
  }
}

.btn > span:nth-child(3) {
  width: 100%;
  height: 2px;
  background: -webkit-linear-gradient(left, #03e9f4, transparent);
  left: 100%;
  bottom: 0px;
  animation: line3 2.3s 0.75s linear infinite;
}

@keyframes line3 {
  50%,
  100% {
    left: -100%;
  }
}

.btn > span:nth-child(4) {
  width: 2px;
  height: 100%;
  background: -webkit-linear-gradient(top, transparent, #03e9f4);
  left: 0px;
  top: 100%;
  animation: line4 2.3s 1s linear infinite;
}


@keyframes line4 {
  50%,
  100% {
    top: -100%;
  }
}
</style>
    
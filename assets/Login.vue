<template>
  <body>
    <!-- 登录 -->
    <div v-show="loginPage" class="box">
      <h2>登录</h2>
      <div class="input-box">
        <input
          placeholder="输入账号"
          type="text"
          v-model="loginForm.username"
        />
      </div>
      <div class="input-box">
        <input
          placeholder="输入密码"
          type="password"
          v-model="loginForm.password"
        />
      </div>
      <div class="btn-box">
        <a href="#/forget-password">忘记密码?</a>
        <div>
          <button @click="handleleLoginIn()">登录</button>
          <button
            @click="
              registerPage = true;
              loginPage = false;
            "
          >
            注册
          </button>
        </div>
      </div>
    </div>
    <!-- 注册  -->
    <div v-show="registerPage" class="box">
      <h2>注册</h2>
      <div class="input-box">
        <input
          placeholder="输入账号"
          type="text"
          v-model="loginForm.username"
        />
      </div>
      <div class="input-box">
        <input placeholder="输入邮箱" type="email" v-model="loginForm.email" />
      </div>
      <div class="input-box">
        <input
          placeholder="输入密码"
          type="password"
          v-model="loginForm.password"
        />
      </div>
      <div class="input-box">
        <input
          placeholder="确认密码"
          type="password"
          v-model="loginForm.passwordCheck"
        />
      </div>
      <div class="btn-box">
        <div>
          <button
            @click="
              registerPage = false;
              loginPage = true;
              register();
            "
          >
            登录
          </button>
          <button>注册</button>
        </div>
      </div>
    </div>
  </body>
</template>
    
    <script>
export default {
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
      loginPage: true,
      registerPage: false,
      loginForm: {
        // 登录用户名密码
        username: "",
        password: "",
        passwordCheck: "",
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
  mounted() {
    window.app = this;
    this.changeIndex("登录");
  },
  methods: {
    changeIndex(value) {
      this.$store.commit("setActiveName", value);
    },
    handleleLoginIn() {
      let _this = this;
      let params = new URLSearchParams();
      params.append("username", this.loginForm.username);
      params.append("password", this.loginForm.password);

      HttpManager.loginIn(params)
        .then((res) => {
          // console.log('-----------获取登录信息---------------')
          document.cookie =
            "AuthSessionId=" +
            res.data.sessionId +
            ";max-age=" +
            res.data.maxInactiveInterval;
          localStorage.setItem("token", res.data.token);
          if (res.code === 200) {
            _this.$message({ message: "登录成功", type: "success" });
            _this.setUserMsg(res.data.userMsg[0]);
            _this.$store.commit("setLoginIn", true);
            setTimeout(function () {
              _this.changeIndex("首页");
              _this.$router.push({ path: "/" });
              _this.$router.go(0);
            }, 300);
          } else {
            _this.notify("用户名或密码错误", "error");
          }
        })
        .catch((failResponse) => {});
    },
    setUserMsg(item) {
      this.$store.commit("setUserId", item.id);
      this.$store.commit("setUsername", item.username);
      this.$store.commit("setAvator", item.avator);
    },
    goSignUp() {
      this.$router.push({ path: "/sign-up" });
    },
    // 注册功能，注册成功后立即登录。
    register() {},
  },
};
</script>
    
    <style scoped>
* {
  margin: 0;
  padding: 0;
}

body {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background: url(../../src/../static/img/himawari.jpg) no-repeat;
  background-size: cover;
}

@media screen and (max-width: 1025px) {
  body {
    width: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    height: 100vh;
    background: url(../../src/../static/img/himawari.jpg) no-repeat;
  }
}

.box {
  border-radius: 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 350px;
  height: 380px;
  border-top: 1px solid rgba(255, 255, 255, 0.5);
  border-left: 1px solid rgba(255, 255, 255, 0.5);
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
  border-right: 1px solid rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(10px);
  background: rgba(50, 50, 50, 0.1);
}
@media screen and (max-width: 1025px) {
  .box {
    border-radius: 30px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 330px;
    height: 330px;
    border-top: 1px solid rgba(255, 255, 255, 0.5);
    border-left: 1px solid rgba(255, 255, 255, 0.5);
    border-bottom: 1px solid rgba(255, 255, 255, 0.2);
    border-right: 1px solid rgba(255, 255, 255, 0.2);
    backdrop-filter: blur(10px);
    background: rgba(50, 50, 50, 0.1);
  }
}

.box > h2 {
  color: rgba(255, 255, 255, 0.9);
  margin-bottom: 20px;
}

.box .input-box {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: start;
  margin-bottom: 10px;
}

.box .input-box > label {
  margin-bottom: 5px;
  color: rgba(255, 255, 255, 0.9);
  font-size: 13px;
}

.box .input-box > input {
  box-sizing: border-box;
  color: rgba(255, 255, 255, 0.9);
  font-size: 14px;
  height: 35px;
  width: 250px;
  background: rgba(255, 255, 255, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.5);
  border-radius: 5px;
  transition: 0.2s;
  outline: none;
  padding: 0 10px;
  letter-spacing: 1px;
}

.box .input-box > input:focus {
  border: 1px solid rgba(255, 255, 255, 0.8);
}

.box .btn-box {
  width: 250px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: start;
}

.box .btn-box > a {
  font-size: 14px;
  text-decoration: none;
  color: rgba(255, 255, 255, 0.9);
  transition: 0.2s;
  width: 250px;
  text-align: end;
}

.box .btn-box > a:hover {
  color: rgba(255, 255, 255, 1);
}

.box .btn-box > div {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: start;
  margin-top: 20px;
}

.box .btn-box > div > button {
  width: 120px;
  height: 35px;
  border: 1px solid rgba(197, 81, 58, 0.8);
  background: rgba(197, 81, 58, 0.5);
  color: rgba(255, 255, 255, 0.9);
  border-radius: 5px;
  transition: 0.2s;
}

.box .btn-box > div > button:nth-of-type(2) {
  margin-left: 10px;
}

.box .btn-box > div > button:hover {
  border: 1px solid rgba(248, 108, 76, 0.5);
  background: rgba(248, 108, 76, 0.5);
}
</style>
    
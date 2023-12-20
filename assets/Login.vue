<template>
<body>
  <!-- 登录 -->
   <div v-show="loginPage" class="box">
        <h2>登录</h2>
        <div class="input-box">
            <input placeholder="输入账号" type="text" v-model="loginForm.username"/>
        </div>
        <div class="input-box">
            <input placeholder="输入密码" type="password" v-model="loginForm.password"/>
        </div>
        <div class="btn-box">
            <a href="#/forget-password">忘记密码?</a>
            <div>
                <button @click="handleleLoginIn()">登录</button>
                <button @click="registerPage = true; loginPage = false">注册</button>
            </div>
        </div>
    </div>
    <!-- 注册  -->
    <div v-show="registerPage" class="box">
        <h2>注册</h2>
        <div class="input-box">
            <input placeholder="输入账号" type="text" v-model="loginForm.username"/>
        </div>
        <div class="input-box">
            <input placeholder="输入邮箱" type="email" v-model="loginForm.email"/>
        </div>
        <div class="input-box">
            <input placeholder="输入密码" type="password" v-model="loginForm.password"/>
        </div>
        <div class="input-box">
            <input placeholder="确认密码" type="password" v-model="loginForm.passwordCheck"/>
        </div>
        <div class="btn-box">
            <div>
                <button @click="registerPage = false; loginPage = true; register()">登录</button>
                <button >注册</button>
            </div>
        </div>
    </div>
</body>
</template>

<script>
  import mixin from '../mixins'
  import LoginLogo from '../components/LoginLogo'
  import {HttpManager} from '../api/index'
  import { rsaEncrypt, rsaDecrypt } from '../utils/JsEncrypt'//rsa
import { stringify } from 'json5'
  export default {
    name: 'login-in',
    mixins: [mixin],
    components: {
      LoginLogo
    },
    data: function () {
      let validateName = (rule, value, callback) => {
        if (!value) {
          return callback(new Error('用户名不能为空'))
        } else {
          callback()
        }
      }
      let validatePassword = (rule, value, callback) => {
        if (value === '') {
          callback(new Error('请输入密码'))
        } else {
          callback()
        }
      }
      return {
        loginPage : true,
        registerPage : false,
        loginForm: { // 登录用户名密码
          username: '',
          password: '',
          passwordCheck: ''
        },
        rules: {
          username: [
            {validator: validateName, message: '请输入用户名', trigger: 'blur'}
          ],
          password: [
            {validator: validatePassword, message: '请输入密码', trigger: 'blur'}
          ]
        }
      }
    },
    mounted() {
      window.app = this;
      this.changeIndex('登录')
    },
    methods: {
      changeIndex(value) {
        this.$store.commit('setActiveName', value)
      },
      handleleLoginIn() {
        let _this = this
        let params = new URLSearchParams()
        params.append('username', this.loginForm.username)
        params.append('password', this.loginForm.password)
        
        HttpManager.loginIn(params)
        .then(res => {
            // console.log('-----------获取登录信息---------------')
            document.cookie = 'AuthSessionId=' + res.data.sessionId+';max-age='+res.data.maxInactiveInterval
            localStorage.setItem('token',res.data.token)
            if (res.code === 200) {
              _this.$message({message: '登录成功',type: 'success'})
              _this.setUserMsg(res.data.userMsg[0])
              _this.$store.commit('setLoginIn', true)
              setTimeout(function () {
                _this.changeIndex('首页')
                _this.$router.push({path: '/'})
                _this.$router.go(0)
              }, 300)
            } else {
              _this.notify('用户名或密码错误', 'error')
            }
          })
          .catch(failResponse => {
          })
      },
      setUserMsg(item) {
        this.$store.commit('setUserId', item.id)
        this.$store.commit('setUsername', item.username)
        this.$store.commit('setAvator', item.avator)
      },
      goSignUp() {
        this.$router.push({path: '/sign-up'})
      },
      // 注册功能，注册成功后立即登录。
      register(){
        
      }
    }
  }
</script>

<style lang="scss" scoped>
  @import './assets/login-in.scss';
</style>

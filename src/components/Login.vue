<template>
  <div class="login_container">
    this is login_container
    <div class="login_box">
      <!-- 头像区域 -->
      <div class="avatar">
        <img alt
             src="../assets/timg.jpg" />
      </div>
      <!-- 表单区域 -->
      <el-form :model="loginForm"
               :rules="loginRules"
               class="login_form"
               label-width="0px"
               ref="loginFormRef">
        <el-form-item prop="username">
          <el-input prefix-icon="iconfont icon-user"
                    v-model="loginForm.username"></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input prefix-icon="iconfont icon-3702mima"
                    type="password"
                    v-model="loginForm.password"></el-input>
        </el-form-item>
        <el-form-item class="btns">
          <el-button @click="login"
                     type="primary">登录</el-button>
          <el-button @click="resetForm"
                     type="info">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      loginRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 8, message: '长度在 3 到 8 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 6, max: 12, message: '长度在 6 到 12 位密码', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    resetForm: function () {
      this.$refs.loginFormRef.resetFields()
    },
    login: function () {
      this.$refs.loginFormRef.validate(async flag => {
        if (!flag) return
        const { data: res } = await this.$http.post('login', this.loginForm)
        if (res.meta.status === 400) {
          this.$message.error('登陆失败，用户名或密码错误！')
        } else if (res.meta.status === 200) {
          this.$message.success('恭喜您，登陆成功！')
          // 保存用户的token到sessionStorage中
          window.sessionStorage.setItem('token', res.data.token)
          // 跳转到首页
          this.$router.push('/home')
        }
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
  background: #2b4b6b;
  height: 100%;
}
.login_box {
  width: 450px;
  height: 300px;
  background: #fff;
  border-radius: 3px;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  .avatar {
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 130px;
    height: 130px;
    border-radius: 50%;
    padding: 10px;
    border: 1px solid #fff;
    box-shadow: 0 0 10px;
    background: #eee;
    img {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background: #eee;
    }
  }
  .login_form {
    position: absolute;
    bottom: 0;
    padding: 0 20px;
    width: 100%;
    box-sizing: border-box;
    .btns {
      display: flex;
      justify-content: flex-end;
    }
  }
}
</style>

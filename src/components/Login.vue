<template>
  <div class="login_container">
    <div class="login_box">
      <div class="avater_box">
        <img src="../assets/logo.png" alt="" />
      </div>

      <el-form ref="formRef" :model="loginForm" class="element-form" :rules="loginFormRules">
        <el-form-item prop="username">
          <el-input
            prefix-icon="iconfont icon-user"
            v-model="loginForm.username"
          ></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input
            prefix-icon="iconfont icon-3702mima"
            type="password"
            v-model="loginForm.password"
          ></el-input>
        </el-form-item>
        <el-form-item class="btns">
          <el-button type="primary" @click="loginFormF">登录</el-button>
          <el-button type="info" @click="resetForm">重置</el-button>
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
      loginFormRules: {
        username: [{ required: true, message: '请输入名称', trigger: 'blur' }, { min: 3, max: 10, message: '长度在3 到 10 个字符', trigger: 'blur' }],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' }, { min: 6, max: 15, message: '长度在6 到 15 个字符', trigger: 'blur' }]
      }
    }
  },
  methods: {
    resetForm () {
      this.$refs.formRef.resetFields()
    },
    loginFormF () {
      this.$refs.formRef.validate(async vaild => {
        if(!vaild) return
        const {data:res} = await this.$http.post('login', this.loginForm)
        if(res.meta.status!==200) return this.$message.error('登录失败');
        this.$message.success('登录成功')
        sessionStorage.setItem('token', res.data.token)
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
  position: relative;
  background-color: #2b4b6b;
  height: 100%;
}
.login_box {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;
}
.avater_box {
  position: absolute;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 140px;
  height: 140px;
  border-radius: 50%;
  border: 1px solid #eee;
  box-shadow: 0 0 5px #ddd;
  padding: 10px;
  img {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    background-color: #eee;
  }
}
.element-form {
  position: absolute;
  box-sizing: border-box;
  padding: 10px;
  width: 100%;
  bottom: 0;
}
.btns {
  display: flex;
  justify-content: flex-end;
}
</style>

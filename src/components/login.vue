<template>
    <div class="login_container">
       <div class="login_box">
         <!-- 头像区 -->
         <div class="avater_box">
           <img src="../assets/logo.png" alt="">
         </div>
          <!-- 登录表单 -->
          <el-form ref="loginFromRef" :model="loginFrom" :rules="loginFromRules" label-width="0px" class="login_form">
            <!-- 登录框 -->
            <el-form-item prop="username" >
              <el-input v-model="loginFrom.username" prefix-icon="iconfont icon-user-fill" ></el-input>
            </el-form-item>
            <!-- 密码框 -->
            <el-form-item prop="password">
              <el-input v-model="loginFrom.password" prefix-icon="iconfont icon-suo" type="password"></el-input>
            </el-form-item>
            <!-- 按钮 -->
            <el-form-item class="btns">
                <el-button type="primary" @click="login">登录</el-button>
                <el-button type="info" @click="resetLoginFrom">重置</el-button>
            </el-form-item>
          </el-form>
       </div>
    </div>
</template>

<script>
// 行为区
export default {
  data () {
    return {
      // 这是登录表单的数据绑定
      loginFrom: {
        username: 'admin',
        password: '123456'
      },
      // 这是表单的验证规则
      loginFromRules: {
        // 验证用户名
        username: [
          { required: true, message: '请输入登录名称', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        // 验证登录密码
        password: [
          { required: true, message: '请输入登录密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ]

      }
    }
  },
  methods: {
    // 点击重置按钮，重置登录表单
    resetLoginFrom () {
      // console.log(this)
      this.$refs.loginFromRef.resetFields()
    },
    login () {
      this.$refs.loginFromRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        const { data: res } = await this.$http.post('login', this.loginFrom)
        if (res.meta.status !== 200) return this.$message.error('登录失败')
        this.$message.success('登录成功')
        // 1.将登录成功之后的token，保存到客户端的sessionStorage中
        // 1.1项目中的其他API只能在登录后才能访问
        // 1.2 token只能在当前网站打开期间生效
        window.sessionStorage.setItem('token', res.data.token)
        // 2.通过编程式导航跳转到后台主页，路由为/home
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="less" scoped>
  .login_container{
    background-color: #2b4b6b;
    height: 100%;
  }
  .login_box{
    width: 450px;
    height: 300px;
    background-color: #fff;
    border-radius: 3px;
    position: absolute;
    top:50%;
    left: 50%;
    transform: translate(-50%,-50%);
    .avater_box{
      width: 130px;
      height: 130px;
      border: 1px solid #eee;
      border-radius: 50%;
      // overflow: hidden;
      padding: 10px;
      box-shadow: 0 0 10px #ddd;
      position: absolute;
      left: 50%;
      transform: translate(-50%,-50%);
      background-color: #fff;
      img{
        width: 100%;
        height: 100%;
        border-radius: 50%;
        background-color: #eee;
      }
    }

    .login_form{
      position: absolute;
      bottom: 0;
      width: 100%;
      box-sizing: border-box;
      padding:0 20px;
    }

    .btns{
      display: flex;
      justify-content: flex-end;
    }
  }

</style>

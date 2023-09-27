<template>
  <div class="register_container">
    <div class="register_box">
      <!-- 头像区域 -->
      <div class="avater_box">
        <img src="../assets/sysLogo.png" alt="" />
      </div>
      <!-- 登陆表单 -->
      <el-form
        ref="registerFormRef"
        :model="registerForm"
        :rules="registerFormRules"
        label-width="0px"
        class="register_form"
      >
        <!-- 用户名 -->
        <el-form-item prop="user_id">
          <el-input
            v-model="registerForm.user_id"
            prefix-icon="el-icon-user-solid"
          ></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input
            v-model="registerForm.password"
            prefix-icon="el-icon-s-goods"
            type="password"
          ></el-input>
        </el-form-item>
        <!-- 按钮 -->
        <el-form-item class="btns">
          <el-radio-group v-model="identity">
            <el-radio label="student">student</el-radio>
            <el-radio label="teacher" class="teacherChoice">teacher</el-radio>
          </el-radio-group>
          <el-button type="primary" @click="register">提交</el-button>
          <el-button type="primary" @click="back">返回</el-button>
          <el-button type="info" @click="resetRegisterForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
import Students from './admin/Students.vue'

export default {
  data() {
    return {    
      // 登录表单的数据
      identity:"student",
      // 教师
      registerForm: {
        user_id: '1001',
        password: '12345678',
      },
      // 表单验证规则对象
      registerFormRules: {
        // 验证用户名是否合法
        user_id: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '长度在 3 到 10 个字符',
            trigger: 'blur',
          },
        ],
        // 验证密码是否合法
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 6,
            max: 15,
            message: '长度在 6 到 15 个字符',
            trigger: 'blur',
          },
        ],
      },
    }
  },
  methods: {
    // 点击重置按钮
    resetRegisterForm() {
      // console.log(this);
      this.$refs.registerFormRef.resetFields()
    },
    // 点击提交按钮
    register() {
      this.$refs.registerFormRef.validate(async (valid) => {
        // console.log(valid);
        if (!valid) return
        const { data: res } = await this.$http.post('register?identity='+this.identity,this.registerForm)

        console.log(res)
        if (res.code !== 200) return this.$message.error('注册失败')
        this.$message.success('注册成功')
        // 如果注册成功，跳转回登录页面重新登录
        if (res.code==200) this.$router.push('/login');

        // // 1.将登录成功之后的 token，保存到客户端的 sessionStorage
        // window.sessionStorage.setItem('token', res.data.token)
        // window.sessionStorage.setItem('user', res.data.user_id)

        // // 2.通过编程式导航跳转到后台主页，路由地址
        // // 跳转到管理员页面
        // if (res.data.user_type == 'A') this.$router.push('/admin/home')
        // // 跳转到学生页面
        // if (res.data.user_type == 'S') this.$router.push('/student/home')
        // // 跳转到教师页面
        // if (res.data.user_type == 'T') this.$router.push('/teacher/home')
      })
    },
    back(){
      this.$router.push('/login')
    }
  },
}
</script>

<style lang="less" scoped>
.register_container {
  background-color: #2b4b6b;
  background-image: url('../assets/background.png');
  background-size: 200% 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.register_box {
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;
  position: absolute;
}

.avater_box {
  height: 130px;
  width: 130px;
  border: 1px solid #eee;
  border-radius: 50%;
  padding: 10px;
  box-shadow: 0 0 10px #eee;
  position: absolute;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #fff;
  img {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    background-color: #eee;
  }
}
.register_form {
  position: absolute;
  bottom: 0%;
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;
}
.teacherChoice {
  position: relative;
  right: 10%;
}

.btns {
  display: flex;
  justify-content: flex-end;
}
</style>
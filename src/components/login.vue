<template>
  <div class="login_container">
    <div class="login_box">
        <!-- 头像区 -->
        <div class="avatar_box">
            <img src="../assets/logo.png" alt="">
        </div>
        <!-- 表单区 -->
        <el-form class="formLogin" :model="form" :rules="rules" ref="formRef">
            <!-- 用户名 -->
            <el-form-item prop="username">
                <el-input  prefix-icon="iconfont icon-user" v-model="form.username"></el-input>
            </el-form-item>
            <!-- 密码框 -->
            <el-form-item prop="password">
                <el-input  prefix-icon="iconfont icon-3702mima" v-model="form.password" type="password"></el-input>
            </el-form-item>
            <!-- 按钮区 -->
            <el-form-item class="btns">
                <el-button type="primary" @click="login">登录</el-button>
                <el-button type="info" @click="resetForm">重置</el-button>
            </el-form-item>
            
        </el-form>
    </div>
  </div>

</template>

<script>

export default {
    data() {
        return {
            form: {
                username: 'admin',
                password: '123456',
            },
            rules: {
                username: [
                    { required: true, message: '请输入登录名称', trigger: 'blur' },
                    { min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }
                ],
                password: [
                    { required: true, message: '请输入密码', trigger: 'blur' },
                    { min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }
                ]
            }
        }
    },
    methods: {
        resetForm() {
            this.$refs.formRef.resetFields()
        },
        login() {
            this.$refs.formRef.validate( async valid => {
                if (!valid) return;
                const {data: res} = await this.$http.post('login', this.form)
                if(res.meta.status !== 200) return this.$message.error('登陆失败')
                this.$message.success('登录成功')
                window.sessionStorage.setItem('token', res.data.token)
                this.$router.push('/home')
            })
        }
    }
}
</script>

<style lang="less" scoped>
.login_container {
    background-color: #2b4b6b;
    height: 100%;
    .login_box {
        width: 450px;
        height: 300px;
        background-color: #fff;
        border-radius: 3px;
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%,-50%);
    }
    .avatar_box {
        height: 130px;
        width: 130px;
        border: 10px solid #fff;
        border-radius: 50%;
        overflow: hidden;
        box-sizing: border-box;
        box-shadow: 0 0 10px #ddd;
        position: absolute;
        left: 50%;
        transform: translate(-50%,-50%);
        img {
            width: 100%;
            height: 100%;
            background-color: #eee;
        }
    }
    .btns {
        float: right;
    }
    .formLogin {
        margin: 110px 30px 10px;
    }
}
</style>
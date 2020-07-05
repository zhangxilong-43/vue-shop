<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card>

            <el-row :gutter="20">
                <el-col :span="10">
                    <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable  @change="getUsersList">
                        <el-button slot="append" icon="el-icon-search" @click="getUsersList" ></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addVisible = true">添加用户</el-button>
                </el-col>
            </el-row>

            <el-table :data="userdList" style="width: 100%" border stripe>
                <el-table-column type="index"></el-table-column>
                <el-table-column prop="username" label="姓名" width="180"> </el-table-column>
                <el-table-column prop="email" label="邮箱" width="180"> </el-table-column>
                <el-table-column prop="mobile" label="电话"> </el-table-column>
                <el-table-column prop="role_name" label="角色"> </el-table-column>
                <el-table-column prop="mg_state" label="状态">
                    <template slot-scope="scope">
                        <el-switch  v-model="scope.row.mg_state" @change="userChange(scope.row)"></el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="180">
                    <template slot-scope="scope" >
                        <el-button type="primary" icon="el-icon-edit"  size="mini" @click="showEdit(scope.row.id)"></el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUser(scope.row.id)"></el-button>
                        <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false" >
                            <el-button type="warning" icon="el-icon-setting" size="mini" @click="setRole(scope.row)" ></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
                
            </el-table>

            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[1, 2, 3, 4]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total"> </el-pagination>    

            <el-dialog title="新增用户" :visible.sync="addVisible" width="50%" @close="addClosed">
                
                <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">

                    <el-form-item label="用户名" required prop="username">
                        <el-input auto-complete="off" v-model="addForm.username" ></el-input>
                    </el-form-item>
                    <el-form-item label="密码" required prop="password">
                        <el-input auto-complete="off" v-model="addForm.password"></el-input>
                    </el-form-item>
                    <el-form-item label="邮箱" required prop="eamil">
                        <el-input auto-complete="off" v-model="addForm.eamil"></el-input>
                    </el-form-item>
                    <el-form-item label="手机" required prop="mobile">
                        <el-input auto-complete="off" v-model="addForm.mobile"></el-input>
                    </el-form-item>

                </el-form>

                <div slot="footer" class="dialog-footer">
                    <el-button @click="addVisible = false">取 消</el-button>
                    <el-button type="primary" @click="addUser">确 定</el-button>
                </div>

            </el-dialog>

            <el-dialog title="编辑用户" :visible.sync="editVisible" width="50%" >
                
                <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">

                    <el-form-item label="用户名" required >
                        <el-input auto-complete="off" v-model="editForm.username" disabled></el-input>
                    </el-form-item>

                    <el-form-item label="邮箱" required prop="email">
                        <el-input auto-complete="off" v-model="editForm.email"></el-input>
                    </el-form-item>

                    <el-form-item label="手机" required prop="mobile">
                        <el-input auto-complete="off" v-model="editForm.mobile"></el-input>
                    </el-form-item>

                </el-form>

                <div slot="footer" class="dialog-footer">
                    <el-button @click="editVisible = false">取 消</el-button>
                    <el-button type="primary" @click="editUser">确 定</el-button>
                </div>

            </el-dialog>

            <el-dialog title="分配角色" :visible.sync="setRoleDialogVisible" width="50%" >
                <div>
                    <p>当前的用户：{{userInfo.username}}</p>
                    <p>当前的角色：{{userInfo.role_name}}</p>
                    <p>分配新的角色：
                        <el-select v-model="selectedRoleId" placeholder="请选择">
                            <el-option v-for="item in roleList" :key="item.id" :label="item.roleName" :value="item.id"></el-option>
                        </el-select>
                    </p>
                </div>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="setRoleDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
                </span>
            </el-dialog>

        </el-card>
    </div>
</template>

<script>
export default {
    data() {
        let checkEamil = (rule, value, cb) => {
            const regEmail = /^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/
            if(regEmail.test(value)) {
                return cb()
            }
            cb(new Error('请输入合法的邮箱'))
        }
        let checkMobile = (rule, value, cb) => {
            const regMobile = /^1(?:3\d|4[4-9]|5[0-35-9]|6[67]|7[013-8]|8\d|9\d)\d{8}$/
            if(regMobile.test(value)) {
                return cb()
            }
            cb(new Error('请输入合法的手机号'))
        }
        return{
            queryInfo: {
                query: '',
                pagenum: 1,
                pagesize: 2,
            },
            userdList: [],
            total: 0,
            addVisible: false,
            editVisible: false,
            addForm: {
                username: '',
                password: '',
                eamil: '',
                mobile: '',
            },
            addFormRules: {
                username: [
                    { required: true, message: '请输入用户名', trigger: 'blur' },
                    { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
                ],
                password: [
                    { required: true, message: '请输入密码', trigger: 'blur' },
                    { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
                ],
                eamil: [
                    { required: true, message: '请输入邮箱地址', trigger: 'blur' },
                    { validator: checkEamil, trigger: 'blur' }
                ],
                mobile: [
                    { required: true, message: '请输入手机', trigger: 'blur' },
                    { validator: checkMobile, trigger: 'blur' }
                ],
                
            },
            editForm: {},
            editFormRules: {
                eamil: [
                    { required: true, message: '请输入邮箱地址', trigger: 'blur' },
                    { validator: checkEamil, trigger: 'blur' }
                ],
                mobile: [
                    { required: true, message: '请输入手机', trigger: 'blur' },
                    { validator: checkMobile, trigger: 'blur' }
                ],
                
            },
            setRoleDialogVisible: false,
            userInfo: '',
            roleList: [],
            selectedRoleId: '',
        }
    },
    created() {
        this.getUsersList()
    },
    methods: {
        async getUsersList() {
            const {data : res} = await this.$http.get('users', {params: this.queryInfo})
            if (res.meta.status !== 200)  return this.$message.error('获取用户列表信息失败！');
            this.userdList = res.data.users 
            this.total = res.data.total
        },
        handleSizeChange(newSize) {
            this.queryInfo.pagesize = newSize
            this.getUsersList()
        },
        handleCurrentChange(newPage) {
            this.queryInfo.pagenum = newPage
            this.getUsersList()
        },
        async userChange(val) {
            const {data: res} = await this.$http.put(`users/${val.id}/state/${val.mg_state}`)
            if(res.meta.status !== 200) {
                val.mg_state = !val.mg_state
                return this.$message.error('更新用户状态失败！')
            }
            this.$message.success('更新用户状态成功！')
        },
        addClosed() {
            this.$refs.addFormRef.resetFields()
        },
        addUser() {
            this.$refs.addFormRef.validate(async valid => {
                if(!valid) return 
                const {data: res} = await this.$http.post('users/', this.addForm)
                if(res.meta.status!==201) {
                    this.$message.error('添加用户失败！')
                }
                this.$message.success('添加用户成功！')
                this.addVisible = false
                this.getUsersList()
            })
        },
        async showEdit(id) {
            this.editVisible = true
            const {data: res} = await this.$http.get('users/' + id)
            if(res.meta.status!==200) {
                this.$message.error('查询用户失败！')
            }
            this.editForm = res.data;
            this.$message.success('查询用户成功！')

        },
        editUser() {
            this.$refs.editFormRef.validate(async valid => {
                if(!valid) return 
                
                const {data: res} = await this.$http.put('users/' + this.editForm.id, {
                    email: this.editForm.email, 
                    mobile: this.editForm.mobile
                })
                if(res.meta.status!==200) {
                    this.$message.error('编辑用户失败！')
                }
                
                this.editVisible = false
                this.getUsersList()
                this.$message.success('编辑用户成功！')
            })
        },
        async removeUser(id) {
            const res = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', { confirmButtonText: '确定', cancelButtonText: '取消', type: 'warning' })
            .catch(err => err)
            if (res !== 'confirm') {
                return this.$message.info('已取消删除！')
            }
            const {data: Res} = await this.$http.delete('users/' + id)
            
            if(Res.meta.status !== 200) {
                return this.$message.error('删除失败！')
            }
            this.$message.success('删除成功！')
            this.getUsersList()
            
            
        },
        async setRole(userInfo) {
            this.userInfo = userInfo

            const {data: res} = await this.$http.get('roles')
            if(res.meta.status !== 200) {
                return this.$message.error('角色列表获取失败！')
            }
            this.roleList = res.data

            this.setRoleDialogVisible = true
        },
        async saveRoleInfo() {
            if(!this.selectedRoleId) {
                return this.$message.error('请选择分配的角色')
            }
            const {data: res} = await this.$http.put(`users/${this.userInfo.id}/role`, {
                rid: this.selectedRoleId
            })
            if(res.meta.status !== 200) {
                console.log(res);
                return this.$message.error('分配角色失败！')
            }
            
            this.$message.success('分配角色失败！')
            this.getUsersList()
            this.setRoleDialogVisible = false
        }
    }
    
}
</script>

<style lang="less" scoped>
.el-table {
    margin-top: 20px;
}
.el-pagination {
    margin-top: 15px;
}
</style>
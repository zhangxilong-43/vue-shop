<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>权限列表</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card>
            <el-button type="primary" @click="addRoleVisible = true">添加角色</el-button>
            <el-table :data="rolesList" style="width: 100%" border stripe  row-key="id">

                <el-table-column type="expand"> 
                    <template slot-scope="scope">
                        <el-row v-for="(item1, i1) in scope.row.children" :key="item1.id" :class="['bdbottom', i1 === 0? 'bdtop':'']" class="center">

                            <el-col :span='5'>
                                <el-tag closable @close="removeRightById(item1.id)">{{item1.authName}} </el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>

                            <el-col :span='19'>
                                <el-row v-for="(item2, i2) in item1.children" :key="item2.id" :class="[i2 === 0? '':'bdtop']" class="center">
                                    <el-col :span='6'>
                                        <el-tag closable type='success' @close="removeRightById(item2.id)">{{item2.authName}}</el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span='18'>
                                        <el-tag v-for="(item3) in item2.children" :key="item3.id" type='warning' closable @close="removeRightById(scope.row, item3.id)">
                                            {{item3.authName}}
                                        </el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>

                        </el-row>
                    </template>
                </el-table-column>

                <el-table-column type="index"> </el-table-column>
                <el-table-column prop="roleName" label="角色名称" width="300"> </el-table-column>
                <el-table-column prop="roleDesc" label="角色描述"  width="300"> </el-table-column>
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEdit(scope.row)">编辑</el-button>
                        <el-button size="mini" type="danger" icon="el-icon-edit" @click="removeRole(scope.row.id)">删除</el-button>
                        <el-button size="mini" type="warning" icon="el-icon-edit" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>

        <el-dialog title="添加角色" :visible.sync="addRoleVisible" width="500px" @close="addRoleClosed">
                
            <el-form :model="addRoleForm" :rules="addRoleFormRules" ref="addRoleFormRef" label-width="80px">

                <el-form-item label="角色名称" required prop="roleName">
                    <el-input auto-complete="off" v-model="addRoleForm.roleName" ></el-input>
                </el-form-item>
                <el-form-item label="角色描述"  prop="roleDesc">
                    <el-input auto-complete="off" v-model="addRoleForm.roleDesc" ></el-input>
                </el-form-item>

            </el-form>

            <div slot="footer" class="dialog-footer">
                <el-button @click="addRoleVisible = false">取 消</el-button>
                <el-button type="primary" @click="addRole">确 定</el-button>
            </div>

        </el-dialog>

        <el-dialog title="编辑角色" :visible.sync="editRoleVisible" width="500px" @close="editRoleClosed">
                
            <el-form :model="editRoleForm" :rules="editRoleFormRules" ref="editRoleFormRef" label-width="80px">

                <el-form-item label="角色名称" required prop="roleName">
                    <el-input auto-complete="off" v-model="editRoleForm.roleName" ></el-input>
                </el-form-item>
                <el-form-item label="角色描述"  prop="roleDesc">
                    <el-input auto-complete="off" v-model="editRoleForm.roleDesc" ></el-input>
                </el-form-item>

            </el-form>

            <div slot="footer" class="dialog-footer">
                <el-button @click="editRoleVisible = false">取 消</el-button>
                <el-button type="primary" @click="editRole">提 交</el-button>
            </div>

        </el-dialog>

        <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="30%" @close="setRightDialogClosed">

            <el-tree :data="rightsList" :props="defaultProps"  show-checkbox  node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"> </el-tree>

            <span slot="footer" class="dialog-footer">
                <el-button @click="setRightDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="setRight">确 定</el-button>
            </span>

        </el-dialog>

    </div>
</template>

<script>
export default {
    data() {
        return {
            rolesList: [],
            rightsList: [],
            addRoleVisible: false,
            addRoleForm: {
                roleName: '',
                roleDesc: '',
            },
            editRoleForm: {
                id: '',
                roleName: '',
                roleDesc: '',
            },
            addRoleFormRules: {
                roleName: [
                    { required: true, message: '请输入角色名称', trigger: 'blur' },
                    { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
                ],
                roleDesc: [
                    { required: false, message: '请输入角色描述（可省略）', trigger: 'blur' },
                    { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
                ] 
            },
            editRoleVisible: false,
            editRoleFormRules: {
                roleName: [
                    { required: true, message: '请输入角色名称', trigger: 'blur' },
                    { min: 1, max: 10, message: '长度在 1 到 10 个字符', trigger: 'blur' }
                ],
                roleDesc: [
                    { required: false, message: '请输入角色描述（可省略）', trigger: 'blur' },
                    { min: 1, max: 20, message: '长度在 1 到 20 个字符', trigger: 'blur' }
                ] 
            },
            setRightDialogVisible: false,
            defaultProps: {
                children: 'children',
                label: 'authName'
            },
            defKeys: [],
            roleId: '',
        }
    },
    created() {
        this.getRolesList()
    },
    methods: {
        async getRolesList() {
            const {data: res} = await this.$http.get('roles')
            if(res.meta.status !== 200) {
                return this.$message.error('获取权限列表失败！')
            }
            this.rolesList = res.data
            console.log(this.rolesList);
            
            
        },
        addRoleClosed() {
            this.$refs.addRoleFormRef.resetFields()
        },
        editRoleClosed() {
            this.$refs.editRoleFormRef.resetFields()
        },
        async addRole() {
            const {data: res} = await this.$http.post('roles',this.addRoleForm);
            if(res.meta.status !== 201) {
                return this.$message.error('添加角色失败！')
            }
            this.$message.success('添加角色成功！')
            this.getRolesList()
            this.addRoleVisible = false
        },
        async removeRole(id) {
            const Res = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {confirmButtonText: '确定', cancelButtonText: '取消',type: 'warning'})
            .catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                });          
            });
            if(Res == 'confirm') {
                const {data: res} = await this.$http.delete('roles/' + id)
                if(res.meta.status !== 200) {
                    return this.$message.error('删除角色失败！')
                }
                this.$message.success('删除角色成功！')
            }
            this.getRolesList()
        },
        showEdit(val) {
            this.editRoleForm.id = val.id
            this.editRoleForm.roleName = val.roleName
            this.editRoleForm.roleDesc = val.roleDesc
            this.editRoleVisible = true
        },
        async editRole() {
            const {data: res} = await this.$http.put('roles/' + this.editRoleForm.id, this.editRoleForm)
            if(res.meta.status !== 200) {
                return this.$message.error('编辑角色失败！')
            }
            this.$message.success('编辑角色成功！')
            this.getRolesList()
            this.editRoleVisible = false
        },
        async removeRightById(role, id2) {
            const Res = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {confirmButtonText: '确定', cancelButtonText: '取消',type: 'warning'})
            .catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                });          
            });
            if(Res == 'confirm') {
                const {data: res} = await this.$http.delete(`roles/${role.id}/rights/${id2}`)
                if(res.meta.status !== 200) {
                    return this.$message.error('删除权限失败！')
                }
                this.$message.success('删除权限成功！')
                // this.getRolesList()不建议
                role.children = res.data
            }
        },
        async showSetRightDialog(role) {
            this.roleId = role.id
            const {data: res} = await this.$http.get('rights/tree')
            if(res.meta.status !== 200) {
                return this.$message.error('获取权限数据失败！')
            }
            this.rightsList = res.data
            
            this.getAllDefKeys(role, this.defKeys)
            
            this.setRightDialogVisible = true
        },
        getAllDefKeys(node, arr) {

            if(!node.children) {
                return arr.push(node.id)
            }
            node.children.forEach(item =>  this.getAllDefKeys(item, arr) )
        },
        setRightDialogClosed() {
            this.defKeys = []
        },
        async setRight() {
            const keys = [
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ]
            const idStr = keys.join(',')
            
            const {data: res} = await this.$http.post(`roles/${this.roleId}/rights`, {rids: idStr})
            if(res.meta.status !== 200) {
                return this.$message.error('授权角色失败！')
            }
            this.$message.success('分配权限成功！')
            
            this.getRolesList()
            this.setRightDialogVisible = false
        }
    }
}
</script>

<style lang="less" scoped>
.el-button {
    margin-bottom: 15px;
}
.el-tag {
    margin: 7px;
}
.bdtop {
    border-top: 1px solid #eee;
}
.bdbottom {
    border-bottom: 1px solid #eee;
}
.center {
    display: flex;
    align-items: center;
}

</style>
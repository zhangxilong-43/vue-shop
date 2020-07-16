<template>
    <div>

        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>参数列表</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card>

            <el-alert title="注意：只允许为第三级分类设置相关参数！" type="warning" class="el-icon-warning" :closable="false"></el-alert>

            <el-row class="cat_opt">
                <el-col>
                    <span>选择商品分类：</span>
                    <el-cascader
                        expand-trigger="hover"
                        :options="cateList"
                        :props="cateProps"
                        v-model="selectedCateKeys"
                        @change="handleChange">
                    </el-cascader>
                </el-col>
            </el-row>

            <el-tabs v-model="activeName" @tab-click="handleTabClick">
                <el-tab-pane label="动态参数" name="many" >
                    <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible = true" class="addBtn">添加属性</el-button>
                    <el-table :data="manyTableData" border stripe>
                        <el-table-column type="expand">
                            
                            <template slot-scope="scope">

                                <el-tag @close="handleClose(i,scope.row)" class="paramsTag" v-for="(item, i) in scope.row.attr_vals" :key="i" closable>{{item}}</el-tag>
                                <el-input
                                    class="input-new-tag"
                                    v-if="scope.row.inputVisible"
                                    v-model="scope.row.inputValue"
                                    ref="saveTagInput"
                                    autofocus 
                                    size="small"
                                    @keyup.enter.native="handleInputConfirm(scope.row)"
                                    @blur="handleInputConfirm(scope.row)"
                                    >
                                </el-input>
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>

                            </template>

                        </el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>

                <el-tab-pane label="静态属性" name="only" >
                    <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible = true">添加属性</el-button>
                    <el-table :data="onlyTableData" border stripe>
                        <el-table-column type="expand">

                            <template slot-scope="scope">

                                <el-tag @close="handleClose(i,scope.row)" class="paramsTag" v-for="(item, i) in scope.row.attr_vals" :key="i" closable>{{item}}</el-tag>
                                <el-input
                                    class="input-new-tag"
                                    v-if="scope.row.inputVisible"
                                    v-model="scope.row.inputValue"
                                    ref="saveTagInput"
                                    autofocus 
                                    size="small"
                                    @keyup.enter.native="handleInputConfirm(scope.row)"
                                    @blur="handleInputConfirm(scope.row)"
                                    >
                                </el-input>
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>

                            </template>

                        </el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
            </el-tabs>

        </el-card>
 
        <el-dialog :title="'添加'+titleText"
            :visible.sync="addDialogVisible"
            width="50%" @close="addDialogClosed">

            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="addForm.attr_name"></el-input>
                </el-form-item>
            </el-form>

            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addParams">确 定</el-button>
            </span>
        </el-dialog>

        <el-dialog :title="'修改'+titleText"
            :visible.sync="editDialogVisible"
            width="50%" @close="editDialogClosed">

            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="editForm.attr_name"></el-input>
                </el-form-item>
            </el-form>

            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editParams">确 定</el-button>
            </span>
        </el-dialog>

    </div>
</template>

<script>
export default {
    data() {
        return {
            cateList: [],
            cateProps: {
                value: 'cat_id',
                label: 'cat_name',
                children: 'children',
            },
            selectedCateKeys: [],
            activeName: 'many',
            manyTableData: [],
            onlyTableData: [],
            addDialogVisible: false,
            addForm: {
                attr_name: ''
            },
            addFormRules: {
                attr_name: { required: true, message: '请输入参数名称', trigger: 'blur' },
            },
            editDialogVisible: false,
            editForm: {
                attr_name: ''
            },
            editFormRules: {
                attr_name: { required: true, message: '请输入参数名称', trigger: 'blur' },
            },
        }
    },
    created() {
        this.getCateList()
    },
    methods: {
        async getCateList() {
            const {data: res} = await this.$http.get('categories')
            if(res.meta.status !== 200) {
                return this.$message.error("获取商品分类失败！");
            }
            this.cateList = res.data
        },
        async getParamsData() {
            if (this.selectedCateKeys.length !== 3) {
                this.selectedCateKeys = []
                this.manyTableData = []
                this.onlyTableData = []
                return
            }
            const {data: res} = await this.$http.get(`categories/${this.cateId}/attributes`, {
                params: {sel: this.activeName}
            })
            console.log(res);
            if(res.meta.status !== 200) {
                return this.$message.error("获取参数列表失败！");
            }
            res.data.forEach(item => {
                item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
                item.inputVisible = false
                item.inputValue = ''
            });
            console.log(res.data);
            if (this.activeName === 'many') {
                this.manyTableData = res.data
            } else {
                this.onlyTableData = res.data
            }
        },
        handleChange() {
            this.getParamsData()
        },
        handleTabClick() {
            this.getParamsData()
        },
        addDialogClosed() {
            this.$refs.addFormRef.resetFields()
        },
        addParams() {
            this.$refs.addFormRef.validate( async valid => {
                if(!valid) return
                const {data: res} = await this.$http.post(`categories/${this.cateId}/attributes`, {
                    attr_name: this.addForm.attr_name,
                    attr_sel: this.activeName
                })
                if(res.meta.status !== 201) {
                    return this.$message.error("添加参数失败！");
                }
                this.$message.success('添加参数成功！')
                this.addDialogVisible = false
                this.getParamsData()
            })
        },
        async showEditDialog(id) {
            const {data: res} = await this.$http.get(`categories/${this.cateId}/attributes/${id}`, {
                params: {attr_sel: this.activeName}
            })
            if(res.meta.status !== 200) {
                return this.$message.error("获取参数信息失败！");
            }
            this.editForm = res.data
            this.editDialogVisible = true
        },
        editDialogClosed() {
            this.$refs.editFormRef.resetFields()
        },
        async editParams() {
            const Res = await this.$refs.editFormRef.validate(async valid => {
                if (!valid) return
                if (!this.editForm.attr_name.trim()) return this.$message.error("请输入名称！")
                const {data: res} = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
                    attr_name: this.editForm.attr_name,
                    attr_sel: this.activeName
                })
                if(res.meta.status !== 200) {
                    return this.$message.error("修改失败！");
                }
                this.$message.success("修改成功！")
                this.getParamsData()
                this.editDialogVisible = false
            })
        },
        async removeParams(id) {
            const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {confirmButtonText: '确定', cancelButtonText: '取消', type: 'warning'})
            .catch(err => err)
            if(confirmResult !== 'confirm') {
                return this.$message.info('已取消删除！')
            }
            const {data: res} = await this.$http.delete(`categories/${this.cateId}/attributes/${id}`)
            if(res.meta.status !== 200) {
                return this.$message.error("删除参数信息失败！");
            }
            this.$message.success("删除参数信息成功！")
            this.getParamsData()
        },
        async handleInputConfirm(row) {
            if(row.inputValue.trim().length === 0) {
                row.inputValue = ''
                row.inputVisible = false
                return
            }
            // this.$nextTick(_ => {
            //     this.$refs.saveTagInput.$refs.input.focus();
            // });
            row.attr_vals.push(row.inputValue.trim())
            row.inputVisible = false
            row.inputValue = ''
            this.putParamsVals(row)
        },
        showInput(row) {
            row.inputVisible = true
        },
        async putParamsVals(row) {
            const {data: res} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
                attr_name: row.attr_name,
                attr_sel: row.attr_sel,
                attr_vals: row.attr_vals.join(' ')
            })
            console.log(res);
            if(res.meta.status !== 200) {
                return this.$message.error("修改参数信息失败！");
            }
            this.$message.success("修改参数信息成功！")
        },
        handleClose(i, row) {
            row.attr_vals.splice(i, 1)
            this.putParamsVals(row)
        }
    },
    computed: {
        isBtnDisabled() {
            if(this.selectedCateKeys.length !== 3) {
                return true
            }
            return false
        },
        cateId() {
            if (this.selectedCateKeys.length === 3) {
                return this.selectedCateKeys[2]
            }
            return null
        },
        titleText() {
            if (this.activeName === 'many') {
                return '动态参数'
            } else {
                return '静态属性'
            }
        }
    }
}
</script>

<style lang="less" scoped>
.cat_opt {
    margin: 15px 0;
}
.addBtn {
    margin-bottom: 15px;
}
.input-new-tag {
    width: 120px;
}
.paramsTag {
    margin : 0 10px;
}
</style>
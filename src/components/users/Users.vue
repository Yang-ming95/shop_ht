<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card>
            <!-- 搜索与添加区域 -->
            <el-row :gutter="20">
                <el-col :span="7">
                    <el-input placeholder="请输入内容" v-model="queryInfo.query">
                        <el-button slot="append" icon="el-icon-search"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="dialogVisible=true">添加用户</el-button>
                </el-col>
            </el-row>
            <!-- 用户列表区域 -->
            <el-table :data="userlist" border stripe>
                <el-table-column type="index"></el-table-column>
                <el-table-column label="姓名" prop="name"></el-table-column>
                <el-table-column label="邮箱" prop="email"></el-table-column>
                <el-table-column label="电话" prop="mobile"></el-table-column>
                <el-table-column label="角色" prop="role_name"></el-table-column>
                <el-table-column label="状态">
                    <template slot-scope="scope">
                        <!-- {{ scope.row }} -->
                        <el-switch @change="userStateChanged(scope.row)" v-model="scope.row.mg_state">
                        </el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <!-- 修改按钮 -->
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">
                        </el-button>
                        <!-- 删除按钮  -->
                        <el-button type="danger" icon="el-icon-delete" size="mini"
                            @click="removeUserById(scope.row.id)"></el-button>
                        <!-- 角色按钮 -->
                        <el-tooltip :enterable="false" effect="dark" content="分配角色" placement="top">
                            <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页区域 -->
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum" :page-sizes="[1, 2, 5, 10]" :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper" :total="total">
            </el-pagination>
        </el-card>
        <!-- 添加用户的对话框 -->
        <el-dialog title="添加用户" :visible.sync="dialogVisible" width="50%" @close="addDialogClosed">
            <!-- 表单 -->
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="addForm.username"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input v-model="addForm.password" type="password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="addForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="addForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <!-- 内容主体区 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addUser">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 修改用户的对话框 -->
        <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%">
            <span>这是一段信息</span>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editDialogVisible = false">确 定</el-button>
            </span>
        </el-dialog>

    </div>
</template>

<script>
    export default {
        data() {
            // 验证邮箱规则
            var checkEmail = (rule, value, cb) => {
                // 验证邮箱的正则表达式
                const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
                if (regEmail.test(value)) {
                    // 合法的邮箱
                    return cb()
                }
                cb(new Error('请输入合法的邮箱'))
            }
            // 验证手机号码规则
            var checkMobile = () => {
                // 验证手机号正则
                const regMobile = /^(0|86|17951)?(13[0-9]|15[12356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
                if (regMobile.test(value)) {

                    return cb()
                }
                cb(new Error('请输入合法的手机号'))
            }
            return {
                userlist: [
                    {
                        "name": "admin",
                        "id": 1,
                        "email": "1234456@qq.com",
                        "mobile": "12568653589",
                        "role_name": "员工",
                        "mg_state": true
                    },
                    {
                        "name": "admin",
                        "id": 2,
                        "email": "1234456@qq.com",
                        "mobile": "12568653589",
                        "role_name": "员工",
                        "mg_state": false
                    },
                    {
                        "name": "admin",
                        "id": 3,
                        "email": "1234456@qq.com",
                        "mobile": "12568653589",
                        "role_name": "员工",
                        "mg_state": false
                    },
                    {
                        "name": "admin",
                        "id": 4,
                        "email": "1234456@qq.com",
                        "mobile": "12568653589",
                        "role_name": "员工",
                        "userlist": true
                    },
                ],
                // 获取用户列表参数对象
                queryInfo: {
                    query: '',
                    // 当前的页数
                    pagenum: 1,
                    // 当前每页显示多少条数据
                    pagesize: 4
                },
                total: 5,
                // 控制添加用户对话框的显示于隐藏
                dialogVisible: false,
                // 添加用户的表单数据
                addForm: {
                    username: '',
                    password: '',
                    email: '',
                    mobile: ''
                },
                // 添加表单的验证规则对象
                addFormRules: {
                    username: [
                        { required: true, message: '请输入用户名', trigger: 'blur' }, { min: 3, max: 10, message: '用户名的长度在3-10个字符之间', trigger: 'blur' }
                    ],
                    password: [
                        { required: true, message: '请输入密码', trigger: 'blur' }, { min: 6, max: 15, message: '密码的长度在6-15个字符之间', trigger: 'blur' }
                    ],
                    email: [
                        { required: true, message: '请输入邮箱', trigger: 'blur' }, { min: 6, max: 20, message: '请输入正确的邮箱', trigger: 'blur' }, { validator: checkEmail, trigger: 'blur' }
                    ],
                    mobile: [
                        { required: true, message: '请输入手机号码', trigger: 'blur' }, { min: 11, max: 11, message: '请输入正确的手机号码', trigger: 'blur' }, { validator: checkMobile, trigger: 'blur' }
                    ]

                },
                // 控制修改对话框的显示隐藏
                editDialogVisible: false

            }
        },
        methods: {
            // 监听 pagesize 改变事件
            handleSizeChange(newSize) {
                console.log(newSize)
                this.queryInfo.pagesize = newSize


            },
            // 监听页码值改变的事件 未完成 需要数据接口
            handleCurrentChange(newPage) {
                console.log(newPage)
            },
            // 监听 switch 开关的状态改变 未完成 需要数据接口
            userStateChanged(userinfo) {
                console.log(userinfo)
            },
            // 监听对话框关闭的事件 
            addDialogClosed() {
                this.$refs.addFormRef.resetFields()
            },
            // 点击按钮添加新用户 未完成 需要数据接口
            addUser() {

                this.dialogVisible = false
            },
            // 修改按钮 未完成
            showEditDialog(id) {
                console.log(id)
                this.editDialogVisible = true
            },
            // 删除按钮 
            async removeUserById(id) {
                console.log(id)
                // 弹框询问用户是否删除数据
                const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).catch(err => err)
                // 如果用户确认删除，则返回值为字符串 confirm
                // 如果用户取消删除，则返回值为字符串 cancel
                // console.log(confirmResult)
                if(confirmResult != 'confirm'){
                    return this.$message.info('已经取消删除')
                }
                console.log('确认了删除')

                const { data: res } = await this.userlist.splice(id)
                this.$message.success('删除用户成功')
            }
        }
    }
</script>

<style lang="less" scoped>

</style>
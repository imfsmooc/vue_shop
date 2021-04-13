<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card>
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="10">
          <!-- 搜索导航栏 -->
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getUserList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUserList"
            ></el-button>
          </el-input>
          <!-- eslint-disable-next-line vue/no-parsing-error -->
        </el-col>
        <el-col :span="4">
          <el-button
            type="primary"
            @click="dialogVisible = true"
          >添加用户</el-button>
          <!-- eslint-disable-next-line vue/no-parsing-error -->
        </el-col>
        <!-- eslint-disable-next-line vue/no-parsing-error -->
      </el-row>
      <!-- 用户列表区 -->
      <el-table
        :data="userlist"
        border
        stripe
      >
        <el-table-column type="index"></el-table-column>
        <el-table-column
          label="姓名"
          prop="username"
        ></el-table-column>
        <el-table-column
          label="邮箱"
          prop="email"
        ></el-table-column>
        <el-table-column
          label="电话"
          prop="mobile"
        ></el-table-column>
        <el-table-column
          label="角色"
          prop="role_name"
        ></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              @change="userStateChange(scope.row)"
            >
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column
          label="操作"
          width="180px"
        >
          <!-- eslint-disable-next-line vue/no-unused-vars -->
          <template slot-scope="scope">
            <!-- 修改按钮 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.id)"
            ></el-button>
            <!-- 删除按钮 -->
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeUserById(scope.row.id)"
            ></el-button>
            <!-- 分配角色按钮 -->
            <el-tooltip
              effect="dark"
              content="分配角色"
              placement="top-start"
              :enterable="false"
            >
              <el-button
                type="warning"
                icon="el-icon-setting"
                size="mini"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页视图区 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
      <!-- 添加对话框 -->
      <el-dialog
        title="添加用户"
        :visible.sync="dialogVisible"
        width="50%"
        @close="addDialogClose"
      >
        <!-- 主体区 -->
        <el-form
          :model="addForm"
          :rules="addFormRules"
          ref="addFormRef"
          label-width="70px"
        >
          <el-form-item
            label="用户名"
            prop="username"
          >
            <el-input v-model="addForm.username"></el-input>
          </el-form-item>
          <el-form-item
            label="密码"
            prop="password"
          >
            <el-input v-model="addForm.password"></el-input>
          </el-form-item>
          <el-form-item
            label="邮箱"
            prop="email"
          >
            <el-input v-model="addForm.email"></el-input>
          </el-form-item>
          <el-form-item
            label="手机"
            prop="mobile"
          >
            <el-input v-model="addForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部按钮区 -->
        <span
          slot="footer"
          class="dialog-footer"
        >
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button
            type="primary"
            @click="addUser"
          >确 定</el-button>
        </span>
      </el-dialog>
      <!-- 修改用户对话框 -->
      <el-dialog
        title="修改用户"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClose"
      >
        <el-form
          :model="editForm"
          :rules="editFormRules"
          ref="editFormRef"
          label-width="70px"
        >
          <el-form-item label="用户名">
            <el-input
              v-model="editForm.username"
              disabled
            ></el-input>
          </el-form-item>
          <el-form-item
            label="邮箱"
            prop="email"
          >
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item
            label="手机"
            prop="mobile"
          >
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span
          slot="footer"
          class="dialog-footer"
        >
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button
            type="primary"
            @click="editUserInfo"
          >确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    // 验证邮箱的规则
    // eslint-disable-next-line no-unused-vars
    var checkEmail = (rule, value, callback) => {
      // eslint-disable-next-line no-unused-vars
      const regEmail = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
      if (regEmail.test(value)) {
        // 合法邮箱
        return callback()
      }
      callback(new Error('请输入合法的邮箱'))
    }
    // eslint-disable-next-line no-unused-vars
    var checkMobile = (rule, value, callback) => {
      // eslint-disable-next-line no-unused-vars
      const regMobile = /^(?:(?:\+|00)86)?1(?:(?:3[\d])|(?:4[5-79])|(?:5[0-35-9])|(?:6[5-7])|(?:7[0-8])|(?:8[\d])|(?:9[189]))\d{8}$/
      if (regMobile.test(value)) {
        return callback()
      }
      callback(new Error('请输入合法的手机号'))
    }
    return {
      // 获取用户列表的参数对象
      queryInfo: {
        query: '',
        // 当前的页数
        pagenum: 1,
        // 当前每页显示多少条
        pagesize: 2
      },
      userlist: [],
      total: 0,
      // 控制添加用户的显示和隐藏对话框
      dialogVisible: false,
      // 添加用户的表单数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加表单验证规则的对象
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', triggler: 'blur' },
          {
            min: 3,
            max: 10,
            message: '用户名的长度在3~10个字符',
            triggle: 'blur'
          }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 6,
            max: 15,
            message: '用户名的长度在6~15个字符',
            triggle: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱', triggler: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入号码', triggler: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 控制修改用户对话框的显示与隐藏
      editDialogVisible: false,
      // 查询到的用户信息
      editForm: {

      },
      // 修改表单验证规则的对象
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', triggler: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入号码', triggler: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      }

    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    async getUserList () {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      }) // eslint-disable-line no-unused-vars
      if (res.meta.status !== 200) {
        return this.$message.console.error('获取用户列表失败')
      }
      this.userlist = res.data.users
      this.total = res.data.total
      console.log(res)
    },
    // 监听pagesize改变的事件
    handleSizeChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    handleCurrentChange (newPage) {
      // console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    // 监听swith开关状态的改变
    async userStateChange (userinfo) {
      console.log(userinfo)
      const { data: res } = await this.$http.put(
        `users/${userinfo.id}/state/${userinfo.mg_state}`
      ) // eslint-disable-line no-unused-vars
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('更新用户状态失败')
      }
      this.$message.success('更新用户状态成功')
    },
    // 监听添加用户对话框关闭事件
    addDialogClose () {
      // 重置表单，先找到ref的引用，通过引用调用resetFields
      this.$refs.addFormRef.resetFields()
    },
    addUser () {
      this.$refs.addFormRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        // 通过网络请求添加用户
        // eslint-disable-next-line no-unused-vars
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加用户失败')
        }
        this.$message.success('添加用户成功')
        // 将对话框隐藏
        this.dialogVisible = false
        // 重新获取用户列表数据
        this.getUserList()
      })
    },
    async showEditDialog (id) {
      // eslint-disable-next-line no-unused-vars
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('修改用户列表失败')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 监听修改对话框关闭事件
    editDialogClose () {
      this.$refs.editFormRef.resetFields()
    },
    // 修改用户信息并提交
    editUserInfo () {
      this.$refs.editFormRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        // 发起修改用户信息的请求
        // eslint-disable-next-line no-unused-vars
        const { data: res } = await this.$http.put('users/' + this.editForm.id, { email: this.editForm.email, mobile: this.editForm.mobile })

        if (res.meta.status !== 200) {
          return this.$message.error('用户信息更新失败')
        }

        // 关闭对话框
        this.editDialogVisible = false
        // 刷新数据
        this.getUserList()
        // 提示修改成功
        this.$message.success('用户信息更新成功')
      })
    },
    // 根据id删除对应得数据
    async removeUserById (id) {
      // 弹框询问用户是否删除
      // eslint-disable-next-line no-unused-vars
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户确认删除，则返回值为confirm
      // 如果用户取消删除，则返回值为concle
      // console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }

      // eslint-disable-next-line no-unused-vars
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) { return this.$message.error('删除用户失败') }
      this.$message.success('删除用户成功')
      this.getUserList()
    }
  }
}
</script>

<style lang="less" scoped>
</style>

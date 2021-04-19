/* eslint-disable vue/no-unused-vars */
<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区 -->
      <el-table
        :data="rolesList"
        border
        stripe
      >
        <!-- 展开列 -->
        <el-table-column type="expand">
          <!-- eslint-disable-next-line vue/no-unused-vars -->
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom',i1===0 ?'bdtop':'','vcenter']"
              v-for="(item1,i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightById(scope.row,item1.id)"
                >{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级三级权限 -->
              <el-col :span="19">
                <!-- 通过for循环来嵌套渲染二级权限 -->
                <el-row
                  :class="[i2===0?'':'bdtop','vcenter']"
                  v-for="(item2,i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row,item2.id)"
                    >{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级权限 -->
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="(item3) in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row,item3.id)"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <pre>

            </pre>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column
          label="角色名称"
          prop="roleName"
        ></el-table-column>
        <el-table-column
          label="角色描述"
          prop="roleDesc"
        ></el-table-column>
        <el-table-column
          label="操作"
          width="300px"
        >
          <!-- eslint-disable-next-line vue/no-unused-vars -->
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
            >编辑</el-button>
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
            >删除</el-button>
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限的对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed()"
    >
      <!-- 树形控件 -->
      <el-tree
        :data="rightList"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="allotRights"
        >确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: [],
      // 控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      rightList: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的id值数组
      defKeys: [],
      roleId: []
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      // eslint-disable-next-line no-unused-vars
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色失败')
      }

      this.rolesList = res.data
    },
    // 根据ID删除对应的权限
    async removeRightById (role, _rightId) {
      // 弹框提示用户是否需要删除
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') { return this.$message.info('取消了删除') }

      // eslint-disable-next-line no-template-curly-in-string

      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${_rightId}`)
      console.log(res)
      if (res.meta.status !== 200) { return this.$message.error('取消删除权限失败') }
      // 重新获取所有的权限列表
      // this.getRolesList()

      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog (role) {
      this.roleId = role.id
      // 获取所有权限列表数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限失败')
      // 把获取到的数据保存到data中
      this.rightList = res.data
      // 递归显示三级权限的id
      this.getLeefKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },

    // 通过递归的形式，获取角色下所有三级权限的id，并保存到defKeys中
    getLeefKeys (node, arr) {
      // 如果当前node节点不包含children属性，则为三级权限
      if (!node.children) {
        return arr.push(node.id)
      }

      node.children.forEach(item => {
        this.getLeefKeys(item, arr)
      })
    },
    // 监听对话框的关闭事件
    setRightDialogClosed () {
      this.defKeys = []
    },
    // 点击为权限分配角色
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        // 展开运算符...
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]

      const idStr = keys.join(',')

      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })

      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }

      this.$message.success('分配权限成功')
      // 刷新列表
      this.getRolesList()
      // 隐藏对话框
      this.setRightDialogVisible = false
    }
  }
}
</script>

 <style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>

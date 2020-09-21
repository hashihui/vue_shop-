<template>
  <div>
    <!-- 面包屑视图区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card class="box-card">
      <!-- 搜索与添加组件 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="dialogVisibleChange">添加用户</el-button>
        </el-col>
      </el-row>
      <el-table :data="userList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="username" label="姓名"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="mobile" label="电话"></el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column prop="mg_state" label="状态">
          <template v-slot:default="scope">
            <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template v-slot:default="scope">
            <el-row>
              <!-- 修改按钮 -->
              <el-button
                type="primary"
                icon="el-icon-edit"
                size="mini"
                @click="editDialogVisibleChange(scope.row)"
              ></el-button>
              <!-- 删除按钮 -->
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
                @click="deleteUser(scope.row.id)"
              ></el-button>
              <!-- 分配角色按钮 -->
              <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                <el-button
                  type="warning"
                  icon="el-icon-setting"
                  size="mini"
                  @click="setRoleDialogChange(scope.row)"
                ></el-button>
              </el-tooltip>
            </el-row>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>

      <!-- 添加用户的弹出框 -->
      <el-dialog title="添加用户" :visible.sync="dialogVisible" width="50%" @close="addDialogClosed">
        <!-- 内容主题区域 -->
        <el-form ref="addUserFormRef" :model="addUserForm" :rules="addUserRules" label-width="70px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addUserForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addUserForm.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addUserForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机" prop="mobile">
            <el-input v-model="addUserForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区域 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 修改用户弹出框 -->
      <el-dialog
        title="修改用户"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClose"
      >
        <el-form
          ref="editUserFormRef"
          :model="editUserForm"
          :rules="addUserRules"
          label-width="70px"
        >
          <el-form-item label="用户名">
            <el-input disabled v-model="editUserForm.username"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editUserForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机" prop="mobile">
            <el-input v-model="editUserForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="editUser">确定</el-button>
        </span>
      </el-dialog>

      <!-- 删除角色 -->
      <el-dialog title="删除用户" :visible.sync="deleteDialogVisible">
        <h1>是否删除用户</h1>
        <el-row>
          <el-button @click="deleteDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="deleteUser">确定</el-button>
        </el-row>
      </el-dialog>
    </el-card>

    <!-- 分配角色 -->
    <el-dialog :visible.sync="setRoleDialogVisible">
      <p>当前用户：{{userInfo.username}}</p>
      <p>当前的角色：{{userInfo.role_name}}</p>
      <p>
        分配新角色：
        <el-select v-model="selectedRole" placeholder="请选择">
          <el-option
            v-for="item in roleList"
            :key="item.id"
            :value="item.id"
            :label="item.roleName"
          ></el-option>
        </el-select>
      </p>
      <span slot="footer" class="dialog-footer">
        <template v-solt="scope">
          <el-button @click="setRoleDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="editRole">确定</el-button>
        </template>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    // 验证邮箱规则
    var checkEmail = (rule, value, callback) => {
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) {
        return callback()
      }
      callback(new Error('请输入合法的邮箱'))
    }

    var checkMobile = (rule, value, callback) => {
      const regMobile = /^(0|86|17951)?(13[0-9]|15[0123456789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (regMobile.test(value)) {
        return callback()
      }
      callback(new Error('请输入合法的电话号码'))
    }
    return {
      // 获取用户列表的参数对象
      queryInfo: {
        query: '',
        // 当前的页数
        pagenum: 1,
        // 当前每页显示多少条数据
        pagesize: 5
      },
      userList: [],
      total: 0,
      dialogVisible: false,
      addUserForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      addUserRules: {
        username: [
          { required: true, message: '用户名不能为空', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在3到10个字符之间', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '密码不能为空', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在6到15个字符之间', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '邮箱不能为空', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '手机不能为空', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      editDialogVisible: false,
      editUserForm: {
        id: '',
        username: '',
        email: '',
        mobile: ''
      },
      deleteDialogVisible: false,
      setRoleDialogVisible: false,
      // 需要被分配角色的用户信息
      userInfo: {},
      // 选择的角色
      selectedRole: '',
      // 所有的角色列表
      roleList: []
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户列表失败')
      }
      this.total = res.data.total
      this.userList = res.data.users
      // console.log(res)
    },
    // 监听 pagesize 改变的事件
    handleSizeChange(newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 监听 页码值 改变的事件
    handleCurrentChange(newPage) {
      // console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    async userStateChanged(userInfo) {
      // console.log(userInfo)
      const { data: res } = await this.$http.put(
        `users/${userInfo.id}/state/${userInfo.mg_state}`
      )
      // console.log(res)
      if (res.meta.status !== 200) {
        userInfo.mg_state = !userInfo.mg_state
        return this.$message.error('更新用户状态失败')
      }
      this.$message.success('更新用户状态成功')
    },
    dialogVisibleChange() {
      this.dialogVisible = !this.dialogVisible
    },
    addUser() {
      this.$refs.addUserFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.addUserForm)
        // console.log(res)
        if (res.meta.status !== 201) return this.$message.error('添加用户失败')
        this.$message.success('添加用户成功')
      })
      // 隐藏添加用户的对话框
      this.dialogVisible = false
      // 重新获取列表
      this.getUserList()
    },
    addDialogClosed() {
      this.$refs.addUserFormRef.resetFields()
    },
    editDialogVisibleChange(info) {
      this.editDialogVisible = true
      this.editUserForm.id = info.id
      this.editUserForm.username = info.username
      this.editUserForm.email = info.email
      this.editUserForm.mobile = info.mobile
    },
    // 修改用户信息并提交
    editUser() {
      // console.log(this.editUserForm)
      this.$refs.editUserFormRef.validate(async (valid) => {
        if (!valid) return
        // console.log(this.editUserForm)
        const { data: res } = await this.$http.put(
          `users/${this.editUserForm.id}`,
          this.editUserForm
        )
        // console.log(res)
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
        this.$message.success('更新成功')
        this.editDialogVisible = false
        this.getUserList()
      })
    },
    editDialogClose() {
      this.$refs.editUserFormRef.resetFields()
    },
    // deleteDialogVisibleChange(id) {
    //   this.deleteDialogVisible = true
    //   this.deleteId = id
    // },
    // async deleteUser(deleteId) {
    //   // console.log(this.deleteId)
    //   const { data: res } = await this.$http.delete(`users/${Number(deleteId)}`)
    //   if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
    //   this.$message.success('删除成功')
    //   this.deleteDialogVisible = false
    //   this.getUserList()
    // },
    async deleteUser(deleteId) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该项目，是否继续？',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch((err) => err)
      // 删除操作
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除成功')
      } else {
        // 执行删除操作
        const { data: res } = await this.$http.delete(
          `users/${Number(deleteId)}`
        )
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
        this.$message.success('删除成功')
        this.deleteDialogVisible = false
        this.getUserList()
      }
    },
    setRoleDialogChange(user) {
      this.setRoleDialogVisible = true
      this.userInfo = user

      this.getRoleList()
    },
    // 获取角色列表
    async getRoleList() {
      const { data: res } = await this.$http.get('roles')
      this.roleList = res.data
    },
    async editRole() {
      // console.log(this.userInfo)
      if (!this.selectedRole) {
        this.$message.error('请选择要分配的角色')
      } else {
        const { data: res } = await this.$http.put(
          `users/${this.userInfo.id}/role`,
          {
            rid: this.selectedRole
          }
        )

        if (res.meta.status !== 200) {
          return this.$message.error(res.met.message)
        }

        this.$message.success('更新成功')
        this.getUserList()
        this.setRoleDialogVisible = false
        this.selectedRole = ''
        this.userInfo = {}
      }
    }
  }
}
</script>

<style lang="less" scoped>
</style>

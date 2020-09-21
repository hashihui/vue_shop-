<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item to="/home">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片布局 -->
    <el-card>
      <!-- 添加角色按钮 -->
      <el-button type="primary">添加角色</el-button>
      <!-- table -->
      <el-table :data="roleList" border stripe>
        <el-table-column type="expand">
          <template v-slot="scope">
            <el-row
              :class="['bdbottom', index1 === 0?'bdtop':'', 'vcenter']"
              v-for="(item1, index1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二, 三级权限 -->
              <el-col :span="19">
                <!-- 渲染二级权限 -->
                <el-row
                  :class="[index2 === 0?'':'bdtop', 'vcenter']"
                  v-for="(item2, index2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                    >{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      v-for="(item3) in item2.children"
                      :key="item3.id"
                      type="warning"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="290px">
          <template v-slot:default="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showSetRightDialog(scope.row)"
            >分配角色</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 分配权限的对话框 -->
    <el-dialog :visible.sync="rightDialogVisible" width="50%" @close="clearTree">
      <!-- 树形控件 -->
      <el-tree
        :data="roleData"
        show-checkbox
        node-key="id"
        :props="defaultProps"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="rightsTree"
      ></el-tree>

      <span slot="footer" class="dialog-footer">
        <el-button @click="rightDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="editRights()">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 所有角色的数据
      roleList: [],
      rightDialogVisible: false,
      // 所有权限的数据
      roleData: {},
      // 树形控件的属性绑定对象
      defaultProps: {
        children: 'children',
        label: 'authName'
      },
      // 默认选中的节点id
      defKeys: [],
      roleId: 0
    }
  },
  created() {
    this.getRoleList()
  },
  methods: {
    async getRoleList() {
      const { data: res } = await this.$http.get('roles')
      this.roleList = res.data

      // 获取所有的权限的数据
      const { data: res1 } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error(res1.meta.message)
      }
      // 保存数据
      this.roleData = res1.data
    },
    async removeRightById(role, rightId) {
      // 删除权限
      const confirmVaule = await this.$confirm('是否删除该权限', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch((err) => err)

      if (confirmVaule !== 'confirm') {
        return this.$message.info('取消删除成功')
      }
      const { data: res } = await this.$http.delete(
        `roles/${Number(role.id)}/rights/${Number(rightId)}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success('删除成功')
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      // 角色有的权限的id
      this.getLeafKeys(role, this.defKeys)
      // 获取该角色的id
      this.roleId = role.id

      /**
       * 将回调延迟到下次 DOM 更新循环之后执行。
       * 在修改数据之后立即使用它，然后等待 DOM 更新。
       * 它跟全局方法 Vue.nextTick 一样，
       * 不同的是回调的 this 自动绑定到调用它的实例上。
       */
      this.$nextTick(() => {
        this.$refs.rightsTree.setCheckedKeys(this.defKeys)
      })

      this.rightDialogVisible = true
    },
    // 通过递归的形式获取末层节点
    getLeafKeys(role, arr) {
      if (role.children) {
        for (const a of role.children) {
          this.getLeafKeys(a, this.defKeys)
        }
      } else {
        arr.push(role.id)
      }
    },
    clearTree() {
      // console.log(this.defKeys)
      this.defKeys = []
    },
    async editRights() {
      // console.log(this.roleId)
      const keys = this.$refs.rightsTree
        .getHalfCheckedKeys()
        .concat(this.$refs.rightsTree.getCheckedKeys())
        .join(',')
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        {
          rids: keys
        }
      )

      if (res.meta.status !== 200) {
        return this.$message.error('更新失败')
      }

      this.$message.success('更新成功')
      this.getRoleList()
      this.rightDialogVisible = false
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
  border-bottom: 1px solid #eeeeee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>

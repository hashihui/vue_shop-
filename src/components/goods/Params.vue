<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item to="/home">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片试图 -->
    <el-card>
      <!-- 警告框 -->
      <el-alert :closable="false" title="注意：只允许为三级分类设置相关参数！" type="warning" show-icon></el-alert>
      <!-- 选择商品分类的级联选择器 -->
      <p>
        选择商品分类：
        <el-cascader
          v-model="selectKeys"
          :options="softTypeList"
          :props="{expandTrigger: 'hover', label: 'cat_name', value: 'cat_id'}"
          clearable
          @change="handleCasChange"
        ></el-cascader>
      </p>

      <!-- Tabs -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 添加动态参数 -->
        <el-tab-pane label="动态参数" name="many">
          <!-- 添加参数按钮 -->
          <el-button
            size="mini"
            type="primary"
            :disabled="isBtnDissable"
            @click="handleAddDialogVisible"
          >添加参数</el-button>
          <!-- 表格 -->
          <el-table border :data="manyTabsData" stripe>
            <el-table-column type="expand">
              <template v-slot="scope">
                <!-- 显示每个属性的下属值 -->
                <el-row>
                  <el-tag
                    closable
                    v-for="item in scope.row.attr_vals?scope.row.attr_vals.split(','):[]"
                    :key="item"
                    @close="elTagClose(scope.row, item)"
                  >{{item}}</el-tag>
                  <el-input
                    v-if="scope.row.inputVisible"
                    v-model="scope.row.inputValue"
                    class="input-new-tag"
                    size="small"
                    ref="saveTagInput"
                    @blur="addAttrVal(scope.row)"
                    @keyup.enter.native="addAttrVal(scope.row)"
                  ></el-input>
                  <el-button
                    v-else
                    class="button-new-tag"
                    size="small"
                    @click="showInput(scope.row)"
                  >+ New Tag</el-button>
                </el-row>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template v-slot="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="editDialogChange(scope.row)"
                >修改</el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="deleteAttr(scope.row)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态属性 -->
        <el-tab-pane label="静态属性" name="only">
          <!-- 添加属性按钮 -->
          <el-button
            size="mini"
            type="primary"
            :disabled="isBtnDissable"
            @click="handleAddDialogVisible"
          >添加参数</el-button>
          <!-- 表格 -->
          <el-table border :data="onlyTabsData" stripe>
            <el-table-column type="expand">
              <template v-slot="scope">
                <!-- 显示每个属性的下属值 -->
                <el-row>
                  <el-tag
                    closable
                    v-for="item in scope.row.attr_vals?scope.row.attr_vals.split(','):[]"
                    :key="item"
                    @close="elTagClose(scope.row, item)"
                  >{{item}}</el-tag>
                  <el-input
                    v-if="scope.row.inputVisible"
                    v-model="scope.row.inputValue"
                    class="input-new-tag"
                    size="small"
                    ref="saveTagInput"
                    @blur="addAttrVal(scope.row)"
                    @keyup.enter.native="addAttrVal(scope.row)"
                  ></el-input>
                  <el-button
                    v-else
                    class="button-new-tag"
                    size="small"
                    @click="showInput(scope.row)"
                  >+ New Tag</el-button>
                </el-row>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template v-slot="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="editDialogChange(scope.row)"
                >修改</el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="deleteAttr(scope.row)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加动态参数和静态属性的弹出框 -->
    <el-dialog
      :title="'添加'+ titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogColse"
    >
      <el-form :model="addForm" ref="addFormRef" :rules="addFormRules">
        <el-form-item :label="titleText" label-width="80px" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <template>
          <el-button @click="addDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="addAttr">确定</el-button>
        </template>
      </span>
    </el-dialog>

    <!-- 修改的弹窗 -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClose"
    >
      <el-form ref="editFormRef" :model="editForm" :rules="addFormRules">
        <el-form-item :label="titleText" label-width="80px" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <template>
          <el-button @click="editDialogVisible = false">取消</el-button>
          <el-button @click="editAttr" type="primary">确定</el-button>
        </template>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 选中的商品分类编号
      selectKeys: [],
      // 商品分类的列表
      softTypeList: [],
      // 被激活的页签的名称
      activeName: 'many',
      manyTabsData: [],
      onlyTabsData: [],
      // 添加属性弹出框
      addDialogVisible: false,
      // 添加属性的表单数据
      addForm: {
        attr_name: ''
      },
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      // 修改表单的出现
      editDialogVisible: false,
      // 修改表单的数据
      editForm: {
        attr_name: ''
      },
      // 要修改的属性的信息
      wantEditInfo: {}
    }
  },
  created() {
    // 获取商品列表
    this.getTypeList()
  },
  methods: {
    // 获取商品分类的列表
    async getTypeList() {
      const { data: res } = await this.$http.get('categories', {
        params: {
          type: 3
        }
      })

      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }

      this.softTypeList = res.data
    },
    async handleCasChange() {
      // console.log(this.selectKeys)
      if (this.selectKeys.length !== 3) {
        this.selectKeys = []
        this.onlyTabsData = []
        this.manyTabsData = []
      } else {
        // console.log(this.selectKeys)
        // 发起数据请求
        this.getTabsData()
      }
    },
    handleTabClick() {
      this.getTabsData()
    },
    async getTabsData() {
      if (!this.isBtnDissable) {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: {
              sel: this.activeName
            }
          }
        )
        // console.log(res.data)
        if (res.meta.status !== 200) {
          return this.$message.error('请求失败')
        }

        res.data.forEach((item) => {
          // 控制文本框的显示和隐藏
          item.inputVisible = false
          item.inputValue = ''
        })
        if (this.activeName === 'many') {
          this.manyTabsData = res.data
        } else {
          this.onlyTabsData = res.data
        }
      }
    },
    // 控制添加弹出框出现
    handleAddDialogVisible() {
      this.addDialogVisible = true
    },
    // 当弹出框关闭
    addDialogColse() {
      this.$refs.addFormRef.resetFields()
    },
    // 添加属性
    addAttr() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        // 添加
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          {
            ...this.addForm,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 201) {
          return this.$message.error('修改失败')
        }

        this.$message.success('修改成功')
        this.addDialogVisible = false
        this.getTabsData()
      })
    },
    // 点击修改按钮
    editDialogChange(type) {
      //   console.log(type)
      this.editDialogVisible = true
      this.wantEditInfo = type
      this.editForm.attr_name = type.attr_name
    },
    // 关闭修改弹框触发
    editDialogClose() {
      // console.log('关闭')
      this.$refs.editFormRef.resetFields()
    },
    // 修改属性
    async editAttr() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.wantEditInfo.attr_id}`,
          {
            ...this.editForm,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('更新失败')
        }

        this.$message.success('更新成功')
        this.editDialogVisible = false
        this.getTabsData()
      })
    },
    // 删除属性
    async deleteAttr(deleteInfo) {
      const confirmResult = await this.$confirm(
        '将删除该项数据，是否继续？',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch((err) => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除成功')
      } else {
        // 进行删除
        const { data: res } = await this.$http.delete(
          `categories/${this.cateId}/attributes/${deleteInfo.attr_id}`
        )
        console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('删除失败')
        }

        this.$message.success('删除成功')
        this.getTabsData()
      }
    },
    showInput(info) {
      info.inputVisible = true
      this.$nextTick((_) => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // input失去焦点时触发添加事件
    async addAttrVal(info) {
      //   console.log(this.inputValue)
      if (info.inputValue.trim().length === 0) {
        // 当输入的值为空时
        info.inputVisible = false
        info.inputValue = ''
      } else {
        // 当输入的值不为空
        const arr = info.attr_vals ? info.attr_vals.split(',') : []
        arr.push(info.inputValue)
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${info.attr_id}`,
          {
            ...info,
            attr_vals: arr.join(',')
          }
        )

        if (res.meta.status !== 200) {
          return this.$message.error('更新失败')
        }

        info.inputVisible = false
        info.inputValue = ''
        // console.log(res.data)
        info.attr_vals = res.data.attr_vals
      }
    },
    async elTagClose(info, name) {
      const arr = info.attr_vals.split(',')
      const index = arr.findIndex((v) => v === name)
      // 删除
      arr.splice(index, 1)
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${info.attr_id}`,
        {
          ...info,
          attr_vals: arr.join(',')
        }
      )

      if (res.meta.status !== 200) {
        return this.$message.error('更新失败')
      }

      info.inputVisible = false
      info.inputValue = ''
      // console.log(res.data)
      info.attr_vals = res.data.attr_vals
    }
  },
  computed: {
    // 如果按钮长度为空则禁用，不为空则启用
    isBtnDissable() {
      if (this.selectKeys.length !== 3) {
        return true
      }
      return false
    },
    // 当前选择的三级分类的id
    cateId() {
      if (this.selectKeys.length === 3) {
        return this.selectKeys[2]
      }
      return null
    },
    // 动态计算标题的文本
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
.el-cascader {
  width: 250px;
}
.el-tag {
  margin-left: 10px;
  margin-bottom: 10px;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
}
</style>

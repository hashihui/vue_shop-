<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item to="/home">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片区域 -->
    <el-card>
      <!-- 添加按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="handleAddDialogChange()">添加分类</el-button>
        </el-col>
      </el-row>

      <!-- 表格区域 -->
      <!-- <el-table
        border
        stripe
        :data="goodsList"
        row-key="cat_id"
        :tree-props="{children: 'children', hasChildren: 'hasChildren'}"
      >
        <el-table-column label="分类名称" prop="cat_name"></el-table-column>
        <el-table-column label="是否有效" prop></el-table-column>
        <el-table-column label="排序"></el-table-column>
        <el-table-column label="操作"></el-table-column>
      </el-table>-->
      <tree-table
        class="tree-table"
        border
        :show-row-hover="false"
        :data="goodsList"
        :columns="columns"
        :show-index="true"
        :expand-type="false"
        :selection-type="false"
        index-text="#"
      >
        <!-- 模板列：是否有用 -->
        <template v-slot:isok="scope">
          <i
            style="color: lightgreen;"
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
          ></i>
          <i style="color: red;" class="el-icon-error" v-else></i>
        </template>

        <!-- 等级模板 -->
        <template v-slot:order="scope">
          <el-tag type="primary" size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" size="mini" v-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag type="warning" size="mini" v-if="scope.row.cat_level === 2">三级</el-tag>
        </template>

        <!-- 操作模板 -->
        <template v-slot:option="scope">
          <el-button icon="el-icon-edit" type="primary" size="mini">编辑</el-button>
          <el-button
            icon="el-icon-delete"
            type="danger"
            size="mini"
            @click="deleteSoft(scope.row)"
          >删除</el-button>
        </template>
      </tree-table>

      <!-- 分页区域 -->
      <el-pagination
        :total="total"
        :page-size="5"
        layout="total, prev, pager, next, jumper"
        @current-change="handleCurrentChange"
      ></el-pagination>
    </el-card>

    <!-- 添加分类弹出框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addSoftDialogVisible"
      width="50%"
      @close="addSoftDialogClose"
    >
      <el-form ref="addSoftFormRef" :model="addSoftForm" :rules="addSoftFormRules">
        <el-form-item label="分类名称" prop="cat_name" label-width="80px">
          <el-input v-model="addSoftForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类" label-width="80px">
          <el-cascader
            v-model="selectedKeys"
            :options="selectForm"
            :props="{ expandTrigger: 'hover', label: 'cat_name', value: 'cat_id', checkStrictly: true }"
            @change="parentCateChange"
            clearable
            ref="cascaderRef"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addSoftDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="addCate">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 請求的參數
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分類的數據列表
      goodsList: [],
      //  总数据条数
      total: 0,
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 表示，当前列定义为模板列
          type: 'template',
          // 表示当前使用的模板名称
          template: 'isok'
        },
        {
          label: '等级',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'option'
        }
      ],
      addSoftDialogVisible: false,
      addSoftForm: {
        // 添加的分类名称
        cat_name: '',
        // 父级id
        cat_pid: 0,
        // 分类等级 默认1级分类
        cat_level: 0
      },
      addSoftFormRules: {
        cat_name: [
          { required: true, message: '分类名称不能为空', trigger: 'blur' }
        ]
      },
      selectForm: [],
      // 选中的父级分类的id数组
      selectedKeys: []
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })

      if (res.meta.status !== 200) {
        return this.$message.error('獲取商品列表失敗')
      }
      // console.log(res)
      this.total = res.data.total
      this.goodsList = res.data.result
    },
    handleCurrentChange(currentPage) {
      this.queryInfo.pagenum = currentPage
      this.getGoodsList()
    },
    async handleAddDialogChange() {
      // 获取父级分类，也就是把所有的1，2级别分类列出
      const { data: res } = await this.$http.get('categories', {
        params: {
          type: 2
        }
      })

      if (res.meta.status !== 200) {
        return this.$message.error('獲取商品列表失敗')
      }
      this.selectForm = res.data

      // 展示弹出层
      this.addSoftDialogVisible = true
    },
    // 选择项发生变化调用
    parentCateChange() {
      // console.log(this.selectedKeys)
      // 如果 selectedKey里面的length>0 证明选择了父级分类
      // 反之没有选择父级分类
      if (this.selectedKeys.length > 0) {
        // 选中了父级分类
        this.addSoftForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
        this.addSoftForm.cat_level = this.selectedKeys.length
      } else {
        this.addSoftForm.cat_pid = 0
        this.addSoftForm.cat_level = 0
      }

      // 收回级联选择器弹出框
      this.$nextTick(() => {
        this.$refs.cascaderRef.dropDownVisible = false
      })
    },
    // 存在问题
    addCate() {
      // console.log(this.addSoftForm)
      // 表单预验证
      this.$refs.addSoftFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          'categories',
          this.addSoftForm
        )
        // console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加失败')
        }

        this.$message.success('添加成功')
        this.getGoodsList()
        this.addSoftDialogVisible = false
      })
    },
    addSoftDialogClose() {
      // console.log(this.$refs.addSoftDialogRef)
      // 重置
      this.$refs.addSoftFormRef.resetFields()
      this.selectedKeys = []
      this.addSoftForm.cat_level = 0
      this.addSoftForm.cat_pid = 0
    },
    // 删除弹框
    async deleteSoft(softInfo) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该分类，是否继续',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch((err) => err)
      // console.log(confirmResult)

      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      // console.log(softInfo)

      // 请求接口删除
      const { data: res } = await this.$http.delete(
        `categories/${softInfo.cat_id}`
      )

      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }

      this.$message.success('删除成功')
      this.getGoodsList()
    }
  }
}
</script>

<style lang="less" scoped>
.tree-table {
  margin-top: 15px;
  margin-bottom: 15px;
  font-size: 12px;
}
.el-cascader {
  width: 100%;
}
</style>

<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item to="/home">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-alert title="添加商品信息" type="info" show-icon center :closable="false"></el-alert>
      <!-- 步骤条 -->
      <el-steps :active="activeIndex - 0" align-center finish-status="success">
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>

      <!-- 添加商品的表单 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-position="top">
        <el-tabs
          :tab-position="'left'"
          v-model="activeIndex"
          :before-leave="beforeTabLeave"
          @tab-click="tabClicked"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-model="addForm.goods_cat"
                :options="cateList"
                :props="{expandTrigger: 'hover', label: 'cat_name', value: 'cat_id'}"
                clearable
                @change="handleChange"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item v-for="item in manyTableData" :key="item.attr_id" :label="item.attr_name">
              <el-checkbox-group v-model="item.checkList">
                <el-checkbox
                  border
                  :label="item1"
                  v-for="(item1, index1) in item.attr_vals"
                  :key="index1"
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item :label="item.attr_name" v-for="item in onlyTabData" :key="item.attr_id">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <!-- action 表示图片上传的地址 -->
            <el-upload
              list-type="picture"
              :action="uploadUrl"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              :headers="headerObj"
              :on-success="handleSuccess"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <el-button type="primary" class="sub_btn" @click="add">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>

    <!-- 图片的预览窗口 -->
    <el-dialog title="图片预览" :visible.sync="showImgDialogVisible">
      <img :src="previewPath" />
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  data() {
    return {
      activeIndex: '0',
      // 添加表单的数据
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        // 商品的详情描述
        goods_introduce: ''
      },
      addFormRules: {
        goods_name: [
          { required: true, message: '商品名称不能为空', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '商品价格不能为空', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '商品重量不能为空', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '商品数量不能为空', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '商品分类不能为空', trigger: 'blur' }
        ]
      },
      // 商品的分类列表
      cateList: [],
      // 商品的参数
      manyTableData: [],
      // 商品的静态属性
      onlyTabData: [],
      // 上传图片的地址
      uploadUrl: 'http://localhost:8888/api/private/v1/upload',
      // 上传图片请求的header
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      showImgDialogVisible: false,
      // 预览图片的地址
      previewPath: ''
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: {
          type: 3
        }
      })

      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }

      this.cateList = res.data
    },
    handleChange() {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
      // console.log(this.addForm.goods_cat)
    },
    beforeTabLeave(activeName, oldActiveName) {
      if (this.addForm.goods_cat.length !== 3) {
        this.$message.error('请先选择商品分类')
        return false
      }
    },
    async tabClicked() {
      // console.log(this.activeIndex)
      if (this.activeIndex === '1') {
        // 获取商品的参数
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: {
              sel: 'many'
            }
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取商品参数失败')
        }

        // console.log(res.data)
        res.data.forEach((v) => {
          v.attr_vals = v.attr_vals.length === 0 ? [] : v.attr_vals.split(',')
          v.checkList = v.attr_vals
        })
        // console.log(res.data)
        this.manyTableData = res.data
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: {
              sel: 'only'
            }
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取静态属性失败')
        }

        // console.log(res.data)
        this.onlyTabData = res.data
      }
    },
    // 处理图片预览效果
    handlePreview(file) {
      // console.log(file)
      this.showImgDialogVisible = true
      this.previewPath = file.response.data.url
    },
    // 处理图片删除效果
    handleRemove(file) {
      const filePath = file.response.data.tmp_path
      const index = this.addForm.pics.findIndex((x) => x.pic === filePath)
      this.addForm.pics.splice(index, 1)
      // console.log(this.addForm)
    },
    // 监听图片上传成功
    handleSuccess(response) {
      // this.addForm.pics.push(response.data)
      // 1.拼接得到一个图片信息对象
      // 2.将图片信息对象，push到pics数组中
      const picInfo = { pic: response.data.tmp_path }
      this.addForm.pics.push(picInfo)
      // console.log(this.addForm)
    },
    // 点击添加按钮
    async add() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return this.$message.error('请填写必要的表单项')
        // 执行添加逻辑
        // 深拷贝addForm对象
        // loadsh cloneDeep(obj)
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        form.attrs = []
        // 处理many 和 only 的属性，处理动态参数和静态属性
        this.manyTableData.forEach((item) => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.checkList.join(',')
          }
          form.attrs.push(newInfo)
        })
        this.onlyTabData.forEach((item) => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attrs_vals
          }
          form.attrs.push(newInfo)
        })
        // console.log(form)
        // 完成添加商品的操作
        const { data: res } = await this.$http.post('goods', form)

        if (res.meta.status !== 201) {
          return this.$message.error(res.meta.msg)
        }

        this.$message.success('创建成功')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    cateId() {
      if (this.addForm.goods_cat.length === 3) return this.addForm.goods_cat[2]
      else return null
    }
  }
}
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 10px 0 0 !important;
}
img {
  width: 100%;
}
.sub_btn {
  margin-top: 15px;
}
</style>

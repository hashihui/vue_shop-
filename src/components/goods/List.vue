<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item to="/home">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片布局 -->
    <el-card>
      <!-- 搜索框和添加按钮 -->
      <el-row :gutter="20">
        <!-- 搜索输入框 -->
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="searchValue" clearable>
            <el-button slot="append" icon="el-icon-search" @click="searchGoodsInfo"></el-button>
          </el-input>
        </el-col>
        <!-- 添加按钮 -->
        <el-col :span="4">
          <el-button type="primary" @click="goAddPage">添加商品</el-button>
        </el-col>
      </el-row>

      <!-- 商品列表 -->
      <el-table border stripe :data="goodsList">
        <el-table-column type="index"></el-table-column>
        <el-table-column label="商品名称" prop="goods_name"></el-table-column>
        <el-table-column label="商品价格（元）" prop="goods_price" width="110px"></el-table-column>
        <el-table-column label="商品重量" prop="goods_weight" width="70px"></el-table-column>
        <el-table-column label="创建时间" prop="add_time" width="140px">
          <template v-slot="scope">{{scope.row.add_time | dateFormat}}</template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template v-slot="scope">
            <el-button icon="el-icon-edit" type="primary" size="mini"></el-button>
            <el-button
              icon="el-icon-delete"
              type="danger"
              size="mini"
              @click="deleteGoods(scope.row)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination
        @current-change="handleCurrentChange"
        :current-page.sync="currentPage"
        :total="goodsTotal"
        :page-size="10"
        layout="total, prev, pager, next, jumper"
      ></el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 请求的参数
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      //   商品列表
      goodsList: [],
      //   商品总数量
      goodsTotal: 0,
      //   现在的页数
      currentPage: 1,
      //   搜索框中的值
      searchValue: ''
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList() {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryInfo
      })

      if (res.meta.status !== 200) {
        return this.$message.error('请求商品列表失败')
      }
      this.goodsList = res.data.goods
      this.goodsTotal = res.data.total
    },
    handleCurrentChange() {
      this.queryInfo.pagenum = this.currentPage
      this.getGoodsList()
    },
    // 模糊搜索搜索商品
    searchGoodsInfo() {
      this.queryInfo.query = this.searchValue
      this.getGoodsList()
    },
    // 根据id删除商品
    async deleteGoods(goodInfo) {
      console.log(goodInfo)
      const confirmResult = await this.$confirm(
        '此操作将永久删除该商品，是否继续？',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch((err) => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除成功')
      }

      //   开始删除操作
      const { data: res } = await this.$http.delete(
        `goods/${goodInfo.goods_id}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }

      this.$message.success('删除成功')
      this.getGoodsList()
    },
    // 跳转至添加商品界面
    goAddPage() {
      this.$router.push('/goods/add')
    }
  },
  watch: {
    searchValue: function (newVal, oldVal) {
      if (newVal === '') {
        this.queryInfo.query = ''
        this.getGoodsList()
      }
    }
  }
}
</script>

<style lang="less" scoped>
</style>

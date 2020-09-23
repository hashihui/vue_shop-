<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input>
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary">添加</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <el-table :data="ordersList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="订单编号" prop="order_number"></el-table-column>
        <el-table-column label="订单价格" prop="order_price"></el-table-column>
        <el-table-column label="是否付款" prop="pay_status">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status === '1'">已付款</el-tag>
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send"></el-table-column>
        <el-table-column label="下单时间">
          <template slot-scope="scope">{{scope.row.create_time | dateFormat}}</template>
        </el-table-column>
        <el-table-column label="操作">
          <template>
            <el-button icon="el-icon-edit" type="primary" size="mini" @click="showBox"></el-button>
            <el-button icon="el-icon-location" type="success" size="mini" @click="showProgress"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页功能 -->
      <el-pagination
        :total="total"
        :page-size="queryInfo.pagesize"
        layout="total, prev, pager, next, jumper"
        @current-change="handleCurrentChange"
      ></el-pagination>
    </el-card>

    <el-dialog title="修改地址" :visible.sync="dialogVisible" width="50%" @close="addDialogClose">
      <el-form :model="addressForm" :rules="addressFormRules" ref="addressFormRef">
        <el-form-item label="省市区/县" label-width="100px" prop="address1">
          <el-cascader v-model="addressForm.address1" :options="cityData"></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" label-width="100px" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary">确定</el-button>
      </span>
    </el-dialog>

    <el-dialog title="订单详情" :visible.sync="timeDialogVisible" width="50%">
      <el-timeline>
        <el-timeline-item
          v-for="(item, index) in process"
          :key="index"
          :timestamp="item.time"
        >{{item.context}}</el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>

<script>
import cityData from './citydata.js'
export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      ordersList: [],
      total: 0,
      dialogVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [
          { required: true, message: '地址不能为空', trigger: 'blur' }
        ],
        address2: [{ required: true, message: '地址不能为空', trigger: 'blur' }]
      },
      cityData: cityData,
      timeDialogVisible: false,
      process: []
    }
  },
  created() {
    this.getOrderList()
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })

      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取列表失败')
      }

      //   console.log(res.data)
      this.ordersList = res.data.goods
      this.total = res.data.total
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    showBox() {
      this.dialogVisible = true
    },
    addDialogClose() {
      this.$refs.addressFormRef.resetFields()
    },
    showProgress() {
      this.timeDialogVisible = true
      this.getProcess()
    },
    async getProcess() {
      const { data: res } = await this.$http.get('kuaidi/1106975712662')
      if (res.meta.status !== 200) {
        return this.$message.error('获取物流信息失败')
      }

      // console.log(res.data)
      this.process = res.data
    }
  }
}
</script>

<style lang="less" scoped>
</style>

<template>
  <div>
    <!-- 导航栏 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 内容区 -->
    <el-card class="box-card">
      <el-row :gutter="25">
        <el-col :span="10">
          <el-input
            placeholder="请输入内容"
            v-model="inputValue"
            class="input-with-select"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="searchValue"
            ></el-button
          ></el-input>
        </el-col>
      </el-row>

      <el-table :data="ordersList">
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="订单编号" prop="order_id"></el-table-column>
        <el-table-column label="订单价格" prop="order_price"></el-table-column>
        <el-table-column label="是否付款">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.pay_status == 0">未付款</el-tag>
            <el-tag v-else>已付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货">
          <template slot-scope="scpoe">
            {{ scpoe.row.is_send | isSend }}
          </template>
        </el-table-column>
        <el-table-column label="下单时间">
          <template slot-scope="scpoe">{{
            scpoe.row.create_time | getTimeStamp
          }}</template></el-table-column
        >
        <el-table-column label="操作">
          <el-button
            icon="el-icon-search"
            type="primary"
            @click="dialogVisible = true"
          ></el-button>
          <el-button icon="el-icon-location" type="success"></el-button>
        </el-table-column>
      </el-table>

      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="400"
      >
      </el-pagination>

      <el-dialog title="修改地址" :visible.sync="dialogVisible" width="30%">
        <el-form
          ref="addFormRef"
          :model="addForm"
          class="element-form"
          :rules="addFormRules"
        >
          <el-form-item label="省市区/县"  prop="region">
            <el-select v-model="addForm.region" placeholder="请选择活动区域">
              <el-option label="区域一" value="shanghai"></el-option>
              <el-option label="区域二" value="beijing"></el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="详细地址" label-width="80px" prop="address">
            <el-input v-model="addForm.address"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="dialogVisible = false"
            >确 定</el-button
          >
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      inputValue: '',
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10,
      },
      total: 0,
      ordersList: [],
      dialogVisible: false,
      addForm: {
        region: {},
        address: ""
      },
      addFormRules: {
        region:[
          { required: true, message: '请输入', trigger: 'blur' },
        ],
        address:[
          { required: true, message: '请输入', trigger: 'blur' },
        ],
      },
    }
  },
  created() {
    this.getOrdersList()
  },
  methods: {
    async getOrdersList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo,
      })
      if (res.meta.status != 200) {
        return this.$message.error(res.meta.msg)
      }
      this.total = res.data.total
      this.ordersList = res.data.goods
    },
    searchValue() {
      this.queryInfo.query = this.inputValue
      this.getOrdersList()
    },
    handleSizeChange(val) {
      this.queryInfo.pagesize = val
      this.getOrdersList()
    },
    handleCurrentChange(val) {
      this.queryInfo.pagenum = val
      this.getOrdersList()
    },
  },
  filters: {
    isSend(data) {
      return data == 1 ? '是' : '否'
    },
    getTimeStamp(timeStamp) {
      const date = new Date(timeStamp * 1000)
      const Y = date.getFullYear() + '-'
      const M =
        (date.getMonth() + 1 < 10
          ? '0' + (date.getMonth() + 1)
          : date.getMonth() + 1) + '-'
      const D = date.getDate() + ' '
      const h = date.getHours() + ':'
      const m = date.getMinutes() + ':'
      const s = date.getSeconds()
      return Y + M + D + h + m + s
    },
  },
  computed: {},
}
</script>
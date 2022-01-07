<template>
  <div>
    <!-- 导航栏 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <!-- 输入框 -->
      <el-row :gutter="25">
        <el-col :span="10"
          ><el-input
            placeholder="请输入内容"
            v-model="inputValue"
            class="input-with-select"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="searchValue"
            ></el-button> </el-input
        ></el-col>
        <el-col :span="6"
          ><el-button type="primary" @click="addGood">添加商品</el-button></el-col
        >
      </el-row>
      <!-- 内容区域 -->
      <el-table :data="shopList" border>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="goods_name" label="商品名称"></el-table-column>
        <el-table-column
          prop="goods_price"
          label="商品价格(元)"
        ></el-table-column>
        <el-table-column prop="goods_number" label="商品数量"></el-table-column>
        <el-table-column label="创建时间">
          <template slot-scope="scope">
            {{ getTime(scope.row.add_time) }}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit"></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              @click="deleteGood(scope.row)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页功能 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      inputValue: '',
      shopList: [],
      total: 0,

      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10,
      },
    }
  },
  created() {
    this.getShopList()
  },
  methods: {
    async getShopList() {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryInfo,
      })
      if (res.meta.status != 200) {
        return this.$message.error(res.meta.msg)
      }
      this.shopList = res.data.goods
      this.total = res.data.total
    },
    getTime(timeStamp) {
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
    // 页码大小变动
    handleSizeChange(val) {
      this.queryInfo.pagesize = val
      this.getShopList()
    },
    // 当前页变动
    handleCurrentChange(val) {
      this.queryInfo.pagenum = val
      this.getShopList()
    },
    searchValue() {
      this.queryInfo.query = this.inputValue
      this.getShopList()
    },
    async deleteGood(data) {
      const confirmRes = await this.$confirm(
        '此操作将永久删除该商品, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }
      ).catch((err) => err)

      if (confirmRes != 'confirm')
        return this.$message({
          type: 'info',
          message: '已取消删除',
        })
      const { data: res } = await this.$http.delete(`goods/${data.goods_id}`)
      if (res.meta.status != 200) return this.$message.error(res.meta.msg)
      this.$message({
        type: 'success',
        message: '删除成功!',
      })
      this.getShopList()
    },
    addGood() {
        this.$router.push('/goods/add')     
    }
  },
}
</script>
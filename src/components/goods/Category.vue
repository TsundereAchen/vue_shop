<template>
  <div>
    <!-- 导航栏 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card class="box-card">
      <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
      <!-- 商品分类列表 -->
      <zk-table
        class="shopTable"
        index-text="#"
        :data="cateList"
        :columns="columns"
        border
        :selection-type="false"
        :expand-type="false"
        :show-index="true"
        :show-row-hover="false"
      >
        <template slot="isOk" slot-scope="scope">
          <i class="el-icon-success" v-if="!scope.row.cat_deleted"></i>
          <i class="el-icon-error" v-else></i>
        </template>
        <template slot="level" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level == 0">一级</el-tag>
          <el-tag type="success" v-else-if="scope.row.cat_level == 1"
            >二级</el-tag
          >
          <el-tag type="info" v-else>三级</el-tag>
        </template>
        <template slot="operate" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit">编辑</el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            @click="deleteCate(scope.row.cat_id)"
            >删除</el-button
          >
        </template>
      </zk-table>
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
    <!-- 添加分类弹窗显示 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
    >
      <el-form
        ref="addCateFormRef"
        :model="newCateInfo"
        :rules="addCateFormRules"
        label-width="100px"
      >
        <el-form-item prop="cat_name" label="分类名称：">
          <el-input v-model="newCateInfo.cat_name"></el-input>
        </el-form-item>
        <el-form-item label=" 父类分类：">
          <!-- props 用来配置指定对象 -->
          <!-- options 用来指定数据源 -->
          <!-- v-model 用来指定选择的值 -->
          <el-cascader
            clearable
            :props="parentCateOptions"
            v-model="selectCateKeys"
            :options="parentCateList"
            @change="parentCateChange"
          ></el-cascader>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="confirmCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  props: {},
  data() {
    return {
      // 查询条件
      queryInfo: {
        type: 3,
        // 当前页码值
        pagenum: 1,
        // 每页显示多少条数据
        pagesize: 5,
      },
      // 分类列表
      cateList: [],
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name',
          width: '400px',
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isOk',
        },
        {
          label: '排序',
          type: 'template',
          template: 'level',
        },
        {
          label: '操作',
          type: 'template',
          template: 'operate',
        },
      ],
      // 新添加的分类信息
      newCateInfo: {
        cat_pid: 0,
        cat_name: '',
        cat_level: 0,
      },
      // 新添加分类名称规则
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
        ],
      },
      // 总的商品分类列表数目
      total: 0,
      // 是否显示添加分类弹窗
      addCateDialogVisible: false,
      // 父级分类列表
      parentCateList: [],
      // 父级分类结构
      parentCateOptions: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover',
        checkStrictly: 'true',
      },
      // 选中的父级分类id数组
      selectCateKeys: [],
    }
  },
  created() {
    this.getGoodCateList()
  },
  methods: {
    // 获取商品分类列表
    async getGoodCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo,
      })
      if (res.meta.status != 200) {
        return this.$message.error('获取商品分类失败')
      }
      this.cateList = res.data.result
      this.total = res.data.total
    },
    //  每页显示条数改变
    handleSizeChange(val) {
      this.queryInfo.pagesize = val
      this.getGoodCateList()
    },
    // 当前页数改变
    handleCurrentChange(val) {
      this.queryInfo.pagenum = val
      this.getGoodCateList()
    },
    // 展示添加分类弹窗
    showAddCateDialog() {
      this.addCateDialogVisible = true
      this.getParentCateList()
    },
    // 获取父级分类列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 },
      })
      if (res.meta.status != 200) {
        return this.$message.error('父级分类获取失败')
      }
      this.parentCateList = res.data
    },
    parentCateChange() {
      if (this.selectCateKeys.length > 0) {
        const arr = this.selectCateKeys
        this.newCateInfo.cat_pid = arr[arr.length - 1]
        this.newCateInfo.cat_level = arr.length
      } else {
        this.newCateInfo.cat_pid = 0
        this.newCateInfo.cat_level = 0
      }
    },
    async confirmCate() {
      const { data: res } = await this.$http.post(
        'categories',
        this.newCateInfo
      )
      if (res.meta.status != 200) {
        this.$message.error(res.meta.msg)
      } else {
        this.$message.success(res.meta.msg)
        this.getGoodCateList()
      }
      this.addCateDialogVisible = false
    },
    async deleteCate(id) {
      const { data: res } = await this.$http.delete(`categories/${id}`)
      if (res.meta.status != 200) {
        this.$message.error(res.meta.msg)
      } else {
        this.$message.success(res.meta.msg)
        this.getGoodCateList()
      }
    },
  },
}
</script>

<style lang="less" scoped>
.shopTable {
  margin-top: 10px;
}
.cascader {
  width: 100%;
}
</style>
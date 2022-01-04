<template>
  <div>
    <!-- 导航栏 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 内容区 -->
    <el-card class="box-card">
      <el-alert
        title="注意：只允许为第三级分类设置相关参数！"
        type="warning"
        show-icon
      >
      </el-alert>
      <el-row type="flex" align="middle" class="row-cate">
        <el-col :span="3"><span>选择商品分类:</span></el-col>
        <el-col :span="10">
          <el-cascader
            v-model="selectCateKeys"
            :options="cateList"
            :props="cateOptions"
            @change="handleChange"
          ></el-cascader>
        </el-col>
      </el-row>

      <!-- 参数标签页 -->
      <el-tabs v-model="activeName" @tab-click="handleClick">
        <el-tab-pane label="动态参数" name="only"></el-tab-pane>
        <el-tab-pane label="静态属性" name="many"></el-tab-pane>
      </el-tabs>
      <ParamComp
        :btnName="btnName"
        :showBtn="showBtn"
        :tableData="attrList"
        :tableName="tableName"
        @addParam="paramFuc"
        @deleteAttr="deleteAttr"
        @addTag="addTag"
      ></ParamComp>
    </el-card>
  </div>
</template>

<script>
import ParamComp from './ParamComp.vue'
export default {
  components: {
    ParamComp,
  },
  data() {
    return {
      // 商品分类列表
      cateList: [],
      // 分类配置
      cateOptions: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover',
      },
      // 选择的keys
      selectCateKeys: [],
      // 参数类型
      activeName: 'only',
      // 按钮名称
      btnName: '添加参数',
      // 表单列名称
      tableName: '参数名称',
      // 参数列表
      attrList: [],
    }
  },
  created() {
    this.getShopCate()
  },
  methods: {
    //   获取商品分类列表
    async getShopCate() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status != 200) {
        return this.$message.error(res.meta.msg)
      }
      this.cateList = res.data
    },
    // 切换分类触发
    handleChange() {
      this.getCurAttrList()
    },
    // 切换参数列表
    handleClick() {
      this.btnName = this.activeName == 'only' ? '添加参数' : '添加属性'
      this.tableName = this.activeName == 'only' ? '参数名称' : '属性名称'
      if (this.selectCateKeys.length >= 3) this.getCurAttrList()
    },
    // 获取最新的参数数据
    async getCurAttrList() {
      const { data: res } = await this.$http.get(
        `categories/${this.catId}/attributes`,
        { params: { sel: this.activeName } }
      )
      if (res.meta.status != 200) {
        return this.$message.error(res.meta.msg)
      }
       res.data.forEach(item => {
          item.attr_vals = item.attr_vals? item.attr_vals.split(' '):[]
          // 添加每一行文本框的显示隐藏数据
          item.inputVisible = false;
          // 文本框输入的值
          item.inputValue = ''
      });
      console.log(res.data)
      this.attrList = res.data
    },
    // 添加参数/添加属性
    paramFuc() {},
    // 删除参数/删除属性
    deleteAttr() {},
    addTag(tagParams) {
        this.attrList.forEach(item=>{
            if(item.attr_id == tagParams.id) {
                item.attr_vals.push(tagParams.val)
            }
        })
    }
  },
  computed: {
    showBtn() {
      return this.selectCateKeys.length >= 3
    },
    catId() {
      const arr = this.selectCateKeys
      if (arr.length >= 3) {
        return arr[arr.length - 1]
      }
      return null
    },
  },
}
</script>

<style lang="less">
.row-cate {
  margin-top: 20px;
}
</style>
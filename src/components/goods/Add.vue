<template>
  <div>
    <!-- 导航栏 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <el-alert title="添加商品信息" type="info" show-icon center></el-alert>
      <!-- 步骤条 -->
      <el-steps
        :active="activeIndex - '0'"
        finish-status="success"
        class="box-step"
      >
        <el-step :title="item" v-for="item in shopInfo" :key="item"></el-step>
      </el-steps>
      <!-- 标签页 -->
      <el-form
        :model="addFrom"
        :rules="addFormRules"
        ref="addFormRef"
        label-position="top"
      >
        <el-tabs
          v-model="activeIndex"
          tab-position="left"
          class="box-tab"
          :before-leave="tabChange"
          @tab-click="tabClick"
        >
          <el-tab-pane label="基本信息">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addFrom.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addFrom.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addFrom.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addFrom.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-model="addFrom.goods_cat"
                :options="cateList"
                :props="cateProps"
                @change="handleChange"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数">
            <el-form-item
              v-for="item in manyAttrList"
              :key="item.attr_id"
              :label="item.attr_name"
            >
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  :label="cb"
                  v-for="(cb, idx) in item.attr_vals"
                  :key="idx"
                  border
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性">
            <el-form-item
              v-for="item in onlyAttrList"
              :key="item.attr_id"
              :label="item.attr_name"
            >
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片">
            <el-upload
              class="upload-demo"
              action="https://lianghj.top:8888/api/private/v1/upload"
              :headers="headerInfo"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :on-success="uploadSuccess"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容">
            <quill-editor v-model="addFrom.goods_introduce"></quill-editor>
            <el-button type="primary" class="finishBtn" @click="addGood">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  data() {
    return {
      shopInfo: [
        '基本信息',
        '商品参数',
        '商品属性',
        '商品图片',
        '商品内容',
        '完成',
      ],
      activeIndex: '0',
      addFrom: {
        goods_name: '',
        goods_price: 0,
        goods_number: 0,
        goods_weight: 0,
        goods_cat: '',
        pics: [],
        attrs: []
      },
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' },
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' },
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' },
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' },
        ],
        goods_cat: [
          { required: true, message: '请选择商品分类', trigger: 'blur' },
        ],
      },
      cateList: [],
      cateProps: {
        label: 'cat_name',
        value: 'cat_id',
        expandTrigger: 'hover',
      },
      // 商品参数
      manyAttrList: [],
      onlyAttrList: [],
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status != 200) {
        return this.$message.error(res.meta.msg)
      }
      this.cateList = res.data
    },
    handleChange() {
      if (this.addFrom.goods_cat.length < 3) {
        this.addFrom.goods_cat = []
      }
    },
    tabChange(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addFrom.goods_cat.length < 3) {
        this.$message.error('请选择商品分类')
        return false
      }
    },
    async tabClick() {
      if (this.activeIndex == 1) {
        const { data: res } = await this.$http.get(
          `categories/${this.catId}/attributes`,
          { params: { sel: 'many' } }
        )
        if (res.meta.status != 200) {
          return this.$message.error(res.meta.msg)
        }
        this.manyAttrList = res.data
        this.manyAttrList.forEach((item) => {
          item.attr_vals = item.attr_vals ? [] : item.attr_vals.split(' ')
        })
      } else if (this.activeIndex == 2) {
        const { data: res } = await this.$http.get(
          `categories/${this.catId}/attributes`,
          { params: { sel: 'only' } }
        )
        if (res.meta.status != 200) {
          return this.$message.error(res.meta.msg)
        }
        this.onlyAttrList = res.data
      }
    },
    handlePreview(data) {},
    handleRemove(file) {
      const idx = this.addFrom.pics.findIndex(
        (item) => item.pic == file.response.data.tmp_path
      )
      this.addFrom.pics.splice(idx)
      console.log(this.addFrom.pics)
    },
    uploadSuccess(response) {
      const pic = { pic: response.data.tmp_path }
      this.addFrom.pics.push(pic)
    },
    addGood() {
        this.$refs.addFormRef.validate( async ress => {
            if(!ress) {
               return this.$message.error('未输入完全')
            }
            const form = _.cloneDeep(this.addFrom)
            form.goods_cat = form.goods_cat.join(',')
            this.manyAttrList.forEach(item => {
                const info = { attr_id:item.attr_id,attr_value: item.attr_vals.join(' ') }
                this.addFrom.attrs.push(info)
            })
            this.onlyAttrList.forEach(item => {
                const info = { attr_id:item.attr_id,attr_value: item.attr_vals}
                this.addFrom.attrs.push(info)
            })
            form.attrs = this.addFrom.attrs
            const {data:res} = await this.$http.post('goods', form)
            if(res.meta.status!=201) {
                return this.$message.error(res.meta.msg)
            }
            this.$router.push('/goods')
        })
    }
  },
  computed: {
    catId() {
      if (this.addFrom.goods_cat.length >= 3) {
        return this.addFrom.goods_cat[2]
      }
      return null
    },
    headerInfo() {
      return {
        Authorization: sessionStorage.getItem('token'),
      }
    },
  },
}
</script>

<style lang="less">
.box-step,
.box-tab,
.finishBtn {
  margin-top: 20px;
}
</style>
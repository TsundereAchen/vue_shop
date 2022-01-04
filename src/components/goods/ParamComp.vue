<template>
  <div>
    <el-button type="primary" @click="addParam" :disabled="!showBtn">
      {{ btnName }}
    </el-button>

    <el-table :data="tableData" border style="width: 100%">
      <el-table-column type="expand">
        <template slot-scope="props">
          <el-tag
            :key="tag"
            v-for="tag in props.row.attr_vals"
            closable
            :disable-transitions="false"
            @close="handleClose(tag)"
          >
            {{ tag }}
          </el-tag>
          <!-- keyup.enter.native回车事件 -->
          <el-input
            class="input-new-tag"
            v-if="props.row.inputVisible"
            v-model="props.row.inputValue"
            ref="saveTagInput"
            size="small"
            @keyup.enter.native="handleInputConfirm(props.row)"
            @blur="handleInputConfirm(props.row)"
          >
          </el-input>
          <el-button
            v-else
            class="button-new-tag"
            size="small"
            @click="showInput"
            >+ New Tag</el-button
          >
        </template>
      </el-table-column>
      <el-table-column type="index" width="50" label="#"></el-table-column>
      <el-table-column prop="attr_name" :label="tableName"></el-table-column>
      <el-table-column label="操作">
        <el-button type="primary" icon="el-icon-edit">修改</el-button>
        <el-button type="danger" icon="el-icon-delete" @click="deleteAttr"
          >删除</el-button
        >
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  props: {
    // 按钮名称
    btnName: '',
    // 表单列名称
    tableName: '',
    // 按钮是否禁用
    showBtn: Boolean,
    // 表单数据
    tableData: [],
  },
  data() {
    return {
      dynamicTags: this.tableData.attr_vals
    }
  },
  created() {},
  methods: {
    addParam() {
      this.$emit('addParam')
    },
    deleteAttr() {
      this.$emit('deleteAttr')
    },
    // 删除标签
    handleClose(tag) {},
    // 添加标签按钮点击
    showInput() {
      this.inputVisible = true
      this.$nextTick((_) => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 回车事件
    async handleInputConfirm(attrData) {
      let inputValue = attrData.inputValue
      if (inputValue) {
        attrData.attr_vals.push(inputValue)
        const str = attrData.attr_vals.join(' ')
        const { data: res } = await this.$http.post(
          `categories/${attrData.attr_id}/attributes`,
          {
            attr_name: attrData.attr_name,
            attr_sel: attrData.attr_sel,
            attr_vals: str,
          }
        )
        if (res.meta.status != 201) {
          this.$message.error(res.meta.msg)
        } else {
        //   this.$emit('addTag',{id:attrData.attr_id,val:inputValue})
        }
      }
      this.inputVisible = false
      attrData.inputValue = ''
    },
  },
  computed: {},
}
</script>

<style lang="less">

</style>
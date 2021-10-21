<template>
  <div class="users_container">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>活动管理</el-breadcrumb-item>
      <el-breadcrumb-item>活动列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card class="box-card">
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="" v-model="userInfo.query">
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getTableList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="5">
          <el-button type="primary" @click="showUsers">添加用户</el-button>
        </el-col>
      </el-row>

      <el-table :data="tableData" stripe style="width: 100%" border="">
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="username" label="姓名" width="180">
        </el-table-column>
        <el-table-column prop="email" label="邮箱" width="180">
        </el-table-column>
        <el-table-column prop="mobile" label="电话"> </el-table-column>
        <el-table-column prop="role_name" label="角色"> </el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              @change="changeStatus(scope.row)"
            >
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="openUpdateUser(scope.row.id)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-share"
              size="mini"
            ></el-button>
            <el-button
              type="warning"
              icon="el-icon-delete"
              size="mini"
              @click="deleteUser(scope.row.id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="userInfo.pagenum"
        :page-sizes="[2, 5, 10, 20]"
        :page-size="userInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      @close="addHandleClose"
    >
      <el-form
        ref="addFormRef"
        :model="addForm"
        class="element-form"
        :rules="addFormRules"
        label-width="100px"
      >
        <el-form-item prop="username" label="用户名">
          <el-input
            prefix-icon="iconfont icon-user"
            v-model="addForm.username"
          ></el-input>
        </el-form-item>
        <el-form-item prop="password" label="密码">
          <el-input
            prefix-icon="iconfont icon-3702mima"
            type="password"
            v-model="addForm.password"
          ></el-input>
        </el-form-item>
        <el-form-item prop="email" label="邮箱">
          <el-input
            prefix-icon="iconfont icon-3702mima"
            type="email"
            v-model="addForm.email"
          ></el-input>
        </el-form-item>
        <el-form-item prop="mobile" label="手机">
          <el-input
            prefix-icon="iconfont icon-3702mima"
            type="tel"
            v-model="addForm.mobile"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUsers">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog title="修改信息" :visible.sync="updateDialogVisible">
      <el-form
        ref="updateFormRef"
        :model="updateForm"
        class="element-form"
        :rules="addFormRules"
        label-width="100px"
      >
        <el-form-item prop="username" label="用户名">
          <el-input
            prefix-icon="iconfont icon-user"
            v-model="updateForm.username"
            disabled
          ></el-input>
        </el-form-item>
        <el-form-item prop="email" label="邮箱">
          <el-input
            prefix-icon="iconfont icon-3702mima"
            v-model="updateForm.email"
          ></el-input>
        </el-form-item>
        <el-form-item prop="mobile" label="手机号">
          <el-input
            prefix-icon="iconfont icon-3702mima"
            v-model="updateForm.mobile"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="updateUsers">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    var checkEmail = (rule, value, callback) => {
      const reg = /^[A-Za-z0-9_-]+@([A-Za-z0-9_-])+(\.[A-Za-z0-9_-])+/

      if (reg.test(value)) return callback()
      callback(new Error('输入不合法'))
    }

    var checkMobile = (rule, value, callback) => {
      var reg = /^1(3\d|4[5-9]|5[0-35-9]|6[2567]|7[0-8]|8\d|9[0-35-9])\d{8}$/
      if (reg.test(value)) return callback()
      callback(new Error('输入不合法'))
    }

    return {
      tableData: [],
      userInfo: {
        query: '',
        // 当前页数
        pagenum: 1,
        // 当前每一页显示多少数据
        pagesize: 2,
      },
      // 一共多少条数据
      total: 0,
      // 添加用户弹窗显示
      addDialogVisible: false,
      updateDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: '',
      },
      updateForm: {
        id: '',
        email: '',
        mobile: '',
      },
      addFormRules: {
        username: [
          { required: true, message: '请输入名称', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在3 到 10 个字符', trigger: 'blur' },
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在6 到 15 个字符', trigger: 'blur' },
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' },
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' },
        ],
      },
    }
  },
  methods: {
    async getTableList() {
      const { data: res } = await this.$http.get('users', {
        params: this.userInfo,
      })
      if (res.meta.status != 200) this.$message.error(res.meta.msg)
      this.tableData = res.data.users
      this.userInfo.pagenum = res.data.pagenum
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.userInfo.pagesize = newSize
      this.getTableList()
    },
    handleCurrentChange(newPage) {
      this.userInfo.pagenum = newPage
      this.getTableList()
    },
    async changeStatus(d) {
      const { data: res } = await this.$http.put(
        `users/${parseInt(d.id)}/state/${d.mg_state}`
      )
      if (res.meta.status != 200) this.$message.error(res.meta.msg)
    },
    showUsers() {
      this.addDialogVisible = true
    },
    addHandleClose() {
      this.$refs.addFormRef.resetFields()
    },
    addUsers() {
      this.$refs.addFormRef.validate(async (vaild) => {
        if (!vaild) return
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status != 201) return this.$message.error(res.meta.msg)
        this.getTableList()
        this.addDialogVisible = false
      })
    },
    updateUsers() {
      this.$refs.updateFormRef.validate(async (vaild) => {
        if (!vaild) return
        const { data: res } = await this.$http.put(
          `users/${this.updateForm.id}`,
          this.updateForm
        )
        if (res.meta.status != 200) this.$message.error(res.meta.msg)
        else this.$message.success('修改成功')
        this.getTableList()
        this.updateDialogVisible = false
      })
    },
    async openUpdateUser(id) {
      this.updateDialogVisible = true
      const { data: res } = await this.$http.get(`users/${id}`)
      this.updateForm = res.data
    },
    deleteUser(id) {
      this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      })
        .then(async () => {
          const { data: res } = await this.$http.delete(`users/${id}`)
          if (res.meta.status == 200) {
            this.$message({
              type: 'success',
              message: '删除成功!',
            })
            this.getTableList()
          } else {
            this.$message({
              type: 'info',
              message: '删除失败',
            })
          }
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除',
          })
        })
    },
  },
  created() {
    this.getTableList()
  },
}
</script>
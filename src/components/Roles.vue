<template>
  <div class="roles_container">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <el-button type="primary" @click="addDialogVisible = true"
        >添加角色</el-button
      >

      <el-table :data="roleList" stripe border>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              v-for="(item, i1) in scope.row.children"
              :key="item.id"
              :class="['bottom-border', 'vcenter', i1 == 0 ? 'top-border' : '']"
            >
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item.id)">
                  {{ item.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row
                  :class="['vcenter', i2 == 0 ? '' : 'top-border']"
                  v-for="(item2, i2) in item.children"
                  :key="item2.id"
                >
                  <el-col :span="5">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                    >
                      {{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="14">
                    <el-tag
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                    >
                      {{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="editUser(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="deleteUser(scope.row.id)"
              >删除</el-button
            >
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="setRoleDialogFuc(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <el-dialog title="修改信息" :visible.sync="updateDialogVisible">
      <el-form
        ref="updateFormRef"
        :model="updateForm"
        class="element-form"
        :rules="addFormRules"
        label-width="100px"
      >
        <el-form-item prop="roleName" label="角色名称">
          <el-input
            prefix-icon="iconfont icon-user"
            v-model="updateForm.roleName"
          ></el-input>
        </el-form-item>
        <el-form-item prop="roleDesc" label="角色描述">
          <el-input
            prefix-icon="iconfont icon-user"
            v-model="updateForm.roleDesc"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="cancelAddUser">取 消</el-button>
        <el-button type="primary" @click="updateUsers">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog title="添加角色信息" :visible.sync="addDialogVisible">
      <el-form
        ref="addFormRef"
        :model="addForm"
        class="element-form"
        :rules="addFormRules"
        label-width="100px"
      >
        <el-form-item prop="roleName" label="角色名称">
          <el-input
            prefix-icon="iconfont icon-user"
            v-model="addForm.roleName"
          ></el-input>
        </el-form-item>
        <el-form-item prop="roleDesc" label="角色描述">
          <el-input
            prefix-icon="iconfont icon-user"
            v-model="addForm.roleDesc"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog
      title="分配权限"
      :visible.sync="setRoleDialogVisible"
      @close="defkeys = []"
    >
      <el-tree
        :data="setRolesData"
        :props="rolesProps"
        show-checkbox
        default-expand-all
        node-key="id"
        :default-checked-keys="defkeys"
        ref="treeRef"
      ></el-tree>

      <span slot="footer" class="dialog-footer">
        <el-button @click="setRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="setNewRoleDialogFuc">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      roleList: [],
      setRolesData: [],
      rolesProps: {
        label: 'authName',
        children: 'children',
      },
      defkeys: [],
      roleId: '',
      updateDialogVisible: false,
      addDialogVisible: false,
      setRoleDialogVisible: false,
      updateForm: {
        roleId: '',
        roleName: '',
        roleDesc: '',
      },
      addForm: {
        roleId: '',
        roleName: '',
        roleDesc: '',
      },
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在3 到 10 个字符', trigger: 'blur' },
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在6 到 15 个字符', trigger: 'blur' },
        ],
      },
    }
  },
  created() {
    this.getRoleList()
  },
  methods: {
    async getRoleList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status != 200) return this.$message.error(res.meta.msg)
      this.roleList = res.data
    },
    updateUsers() {
      this.$refs.updateFormRef.validate(async (vaild) => {
        if (!vaild) return
        const { data: res } = await this.$http.put(
          `roles/${this.updateForm.roleId}`,
          this.updateForm
        )
        if (res.meta.status != 200) return this.$message.error(res.meta.msg)
        this.$message.success('修改成功')
        this.getRoleList()
        this.updateDialogVisible = false
      })
    },
    async editUser(id) {
      const { data: res } = await this.$http.get(`roles/${id}`)
      if (res.meta.status != 200) return this.$message.error(res.meta.msg)
      this.updateForm = res.data
      this.updateDialogVisible = true
    },
    async deleteUser(id) {
      const confirmRes = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }
      ).catch((err) => err)
      if (confirmRes !== 'confirm') {
        return this.$message({
          type: 'info',
          message: '已取消删除',
        })
      }
      const { data: res } = await this.$http.delete(`roles/${id}`)
      if (res.meta.status != 200) return this.$message.error('删除失败')
      this.$message.success('删除成功!')
      this.getRoleList()
    },
    addUser() {
      this.$refs.addFormRef.validate(async (vaild) => {
        if (!vaild) return
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status != 201) return this.$message.error(res.meta.msg)
        this.$message.success(res.meta.msg)
        this.getRoleList()
        this.addDialogVisible = false
      })
    },
    cancelAddUser() {
      this.addDialogVisible = false
      this.$refs.addFormRef.resetFields()
    },
    async removeRightById(role, rightId) {
      const confirmRes = await this.$confirm(
        '此操作将删除该角色权限, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }
      ).catch((err) => err)
      if (confirmRes != 'confirm') {
        return this.$message.info('取消了删除')
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      if (res.meta.status != 200) return this.$message.error('删除失败')
      this.$message.success('删除成功')
      role.children = res.data
    },
    async setRoleDialogFuc(role) {
      this.defkeys = []
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status != 200) return this.$message.err(res.meta.msg)
      this.setRolesData = res.data
      this.getLeafKeys(role, this.defkeys)
      this.roleId = role.id
      this.setRoleDialogVisible = true
    },
    getLeafKeys(node, arr) {
      if (!node.children) return arr.push(node.id)
      node.children.forEach((element) => {
        this.getLeafKeys(element, arr)
      })
    },
    async setNewRoleDialogFuc() {
      const key = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys(),
      ]
      const idKeys = key.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, {
        rids: idKeys,
      })
      if (res.meta.status != 200) this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
      this.setRoleDialogVisible = false
    },
  },
}
</script>
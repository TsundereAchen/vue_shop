<template>
  <el-container>
    <el-header>
      <div>
        <img src="../assets/heima.png" alt="" />
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="loginOut">退出</el-button>
    </el-header>
    <el-container>
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu
          background-color="#373d41"
          text-color="#fff"
          active-text-color="#409eff"
          :collapse="isCollapse"
          :collapse-transition='false'
          unique-opened
        >
          <el-submenu
            :index="item.id + ''"
            v-for="item in dataList"
            :key="item.id"
          >
            <template slot="title">
              <i :class="formIcon[item.id]"></i>
              <span>{{ item.authName }}</span>
            </template>
            <el-menu-item
              :index="d.id + ''"
              v-for="d in item.children"
              :key="d.id"
            >
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{ d.authName }}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>

      <el-main>Main</el-main>
    </el-container>
  </el-container>
</template>
 
<script>
export default {
  data() {
    return {
      dataList: [],
      formIcon: {
        125: 'el-icon-user-solid',
        103: 'el-icon-s-goods',
        101: 'el-icon-s-order',
        102: 'el-icon-s-home',
        145: 'el-icon-s-claim',
      },
      isCollapse: false,
    }
  },
  methods: {
    loginOut() {
      sessionStorage.removeItem('token')
      this.$router.push('/login')
    },
    async requestList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status != 200) return this.$message.error(res.meta.msg)
      this.dataList = res.data
    },
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    },
  },
  created() {
    this.requestList()
  },
}
</script>

<style lang="less" scoped>
.el-container {
  height: 100%;
  padding: 0;
  margin: 0;
}
.el-header {
  margin: 0;
  padding-left: 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #fff;
  font-size: 20px;
  background-color: #373d41;
  div {
    display: flex;
    align-items: center;
    span {
      margin-left: 10px;
    }
  }
}
.toggle-button {
  background-color: #4a5064;
  word-spacing: 0.5em;
  text-align: center;
  color: #fff;
  font-size: 10px;
  line-height: 24px;
  letter-spacing: 0.3em;
  cursor: pointer;
}
.el-aside {
  background-color: #333744;
}
.el-main {
  background-color: #eaedf1;
}
</style>
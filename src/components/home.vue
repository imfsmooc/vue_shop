<template>
  <el-container class="home_container">
    <!-- 头部区域 -->
  <el-header>
    <div>
      <img src="../assets/header.jpeg" alt="">
      <span>管理平台</span>
    </div>
    <el-button type="info" @click="logout">退出</el-button>
    </el-header>
  <el-container>
    <!-- 侧边栏 -->
    <el-aside :width="isCollapse ? '64px':'200px'">
      <div class="toggle-button" @click="toggleCollpuse">|||</div>
        <!-- 侧边栏菜单区 -->
      <el-menu background-color="#545c64" text-color="#fff" active-text-color="#409fff" unique-opened :collapse="isCollapse" :collapse-transition="false" router :default-active="activePath">
        <!-- 一级菜单 -->
        <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
          <template slot="title">
            <!-- 一级菜单的图标 -->
            <i :class="iconsObj[item.id]"></i>
            <!-- 一级菜单的文字 -->
            <span>{{item.authName}}</span>
          </template>
          <!-- 二级菜单 -->
          <el-menu-item :index="'/'+ subItem.path +''" v-for="subItem in item.children" :key ="subItem.id" @click="saveNavState('/'+ subItem.path )">
            <i class="el-icon-menu"></i>
            <span>{{subItem.authName}}</span>
          </el-menu-item>
        </el-submenu>

      </el-menu>

    </el-aside>
    <!-- 主体 -->
    <el-main>
      <!-- 路由占位符 -->
       <router-view></router-view>
    </el-main>
  </el-container>
</el-container>
</template>

<script>
export default {
  data () {
    // 左侧菜单数据
    return {
      menuList: [],
      iconsObj: {
        125: 'iconfont icon-yonghuguanli_huaban',
        103: 'iconfont icon-quanxianguanli',
        101: 'iconfont icon-shangpinguanli',
        102: 'iconfont icon-dingdanguanli',
        145: 'iconfont icon-shujutongji'
      },
      isCollapse: false,
      // 被激活的链接地址
      activePath: ''
    }
  },
  // 生命周期函数
  created () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList () {
      // eslint-disable-next-line no-unused-vars
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
      console.log(res)
    },
    // 点击按钮，切换菜单的折叠与展开
    toggleCollpuse () {
      this.isCollapse = !this.isCollapse
    },
    // 保存链接的激活状态
    saveNavState (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }

  }
}
</script>

<style lang="less" scoped>
  .home_container{
    height: 100%;
  }
  .el-header{
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    img{
      width: 60px;
      height: 60px;
    }
    padding-left: 0;
    align-items: center;
    color: #fff;
    font-size: 20px;
    >div{
      display: flex;
      align-items: center;
      span{
        margin-left: 15px;
      }
    }
  }
  .el-aside{
    background-color: #333744;
    .el-menu{
      border-right: 0px;
    }
  }
  .el-main{
    background-color: #eaedf1;
  }
  .iconfont{
    margin-right: 10px;
  }
  .toggle-button{
    background-color: #4a5064;
    color: #fff;
    font-size: 10px;
    line-height: 24px;
    text-align: center;
    letter-spacing: .2em;
    cursor: pointer;
  }
</style>

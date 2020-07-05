<template>
    <el-container class="home">

        <el-header>
            <div>
                <img src="../assets/logo.svg" >
                <span>电商后台管理系统</span>
            </div>
            <el-button type="info" @click="logout" class="info"> 退出 </el-button>
        </el-header>

        <el-container>

            <el-aside :width="collapse ? '64px' : '200px'">
                <div class="toggle-button" @click="toggleCollapse" >| | |</div>
                <el-menu unique-opened style="border: 0" :collapse="collapse" :collapse-transition="false" router :default-active='activePath'>
                    <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
                        <template slot="title">
                            <i :class="iconsObj[item.id]"></i>
                            <span style="padding-left: 10px">{{item.authName}}</span>
                        </template>
                        <el-menu-item :index="'/' +subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="savePath('/' +subItem.path)">
                            <template slot="title">
                                <i class="el-icon-menu"></i>
                                <span>{{subItem.authName}}</span>
                            </template>
                        </el-menu-item>
                    </el-submenu>
                </el-menu>
            </el-aside>

            <el-main>
                <router-view></router-view>
            </el-main>

        </el-container>
    </el-container>
</template>

<script>
export default {
    data() {
        return {
            menuList:[],
            iconsObj: {
                '125': 'iconfont icon-users',
                '103': 'iconfont icon-tijikongjian',
                '101': 'iconfont icon-shangpin',
                '102': 'iconfont icon-danju',
                '145': 'iconfont icon-baobiao',
            },
            collapse: false,
            activePath: '',
        }
    },
    created() {
        this.getMenuList()
        this.activePath = window.sessionStorage.getItem('activePath')
    },
    methods: {
        logout() {
            window.sessionStorage.clear()
            this.$router.push('/login')
        },
        async getMenuList() {
            const {data: res } = await this.$http.get('menus')
            
            if(res.meta.status !== 200) return this.$message.error(res.meta.msg)
            this.menuList = res.data
        },
        toggleCollapse() {
            this.collapse = !this.collapse
        },
        savePath(val) {
            window.sessionStorage.setItem('activePath', val)
            this.activePath = val
        }
    }
}
</script>script

<style lang="less" scoped>
.home {
    height: 100%;
}
.el-header {
    background-color: rgba(84, 157, 230, 0.616);
    display: flex;
    justify-content: space-between;
    align-items: center;
    color: #fff;
    font-size: 30px;
    div {
        display: flex;
        align-items: center;
        span {
            margin-left: 15px;
        }
    }
    .info {
        background-color: rgb(3, 69, 139);
    }
}
.el-aside {
    background-color: rgb(140, 197, 255);
    .el-menu {
        background-color: #c7e0fa;
    }
    .el-menu-item-group__title {
        padding: 0!important;
    }
    .toggle-button {
        background-color: #85a6c7;
        color: #fff;
        text-align: center;
        height: 18px;
        line-height: 21px;
        cursor: pointer;
    }
}
.el-main {
    background-color: rgb(217, 236, 255);
}
</style>
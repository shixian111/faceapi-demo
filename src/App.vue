<template>
  <div id="app">
    <!-- <router-view></router-view> -->
    <bigScreen></bigScreen>
  </div>
</template>

<script>
import bigScreen from "./views/bigScreen"
export default {
  name: 'App',
  components:{
    bigScreen
}
}

</script>

<style>
body,html{
  /* background-color:#F3F3F4; */
  margin:0 auto; padding:0;
  height: 100%;
}
header,div{margin:0; padding:0;}
h1,h2,h3,h4,h5,h6,ul,li,dl,dt,dd,form,img,p{
margin:0; padding:0; border:none; list-style-type:none;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  height:100%;
}

/* 公共样式 */
.content-wrap{
  margin-top:45px;
  border-top:30px solid #F0F1F4;
  margin-bottom:15px;
  background-color: #fff;
  overflow-y:hidden;
  width:100%;
  overflow-x:hidden;
  /* margin-left:1%;   */
  padding:10px 0;
  min-height:82vh;
}
.page-middle{
  margin-top:20px;
  padding:0 40px;
}
.page-foot{
  margin-top:20px;
  text-align: center;
}
.el-table{
  border-left:1px solid #EBEEF5;
  border-right:1px solid #EBEEF5;
}
.el-table thead{
           color:#54565B;
        }
        .el-table th{
            background-color: #EEEEEE;
        }
        .el-table th>.cell{
            font-weight: normal;
        }
        .el-icon-arrow-down{
          font-size: 15px;
          color:#000;
        }
    .clearfix:after,.clearfix:before{
        content: "";
        display: table;
    }
    .clearfix:after{
        clear: both;
    }
    .clearfix{
        *zoom: 1;
    }
</style>

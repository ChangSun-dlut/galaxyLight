<template>
  <div class="dataBar" v-loading.body="loading">
    <Sreach :data = "sreachObj"  v-on:childMessage = "getprocessedTaskDataBySreach"></Sreach> <!--载入公用搜索组件，搜索后出发API请求-->
    
    <table width="100%" border="0" v-if="ProcessedTaskData.length > 0">
      <tr>
        <td>bid</td>
        <td>bapp</td>
        <td>uri</td>
        <td>edate</td>
        <td>pathParemater</td>
       
      </tr>
      <tr v-for = "item in ProcessedTaskData">
        <td>{{item.bid}}</td>
        <td>{{item.bapp}}</td>
        <td>{{item.uri}}/{{item.bid}}</td>
        <td>{{item.edate | formatDate}}</td>
        <td>{{item.pathParemater | json2string}}</td>
     
      </tr>
    </table>
    
    <el-row v-else>
      <el-col :span="24" style="width:100%;text-align:center;padding:15px 0;font-size:14px;"><div class="grid-content">暂无数据</div></el-col>
    </el-row>

    <Page v-show="ProcessedTaskData.length > pageObj.pageSize" :pageTotal="pageTotal" v-on:childMessage = "getprocessedTaskDataByPage"></Page> <!--载入公用分页组件-->
  </div>
</template>

<script>
import Sreach from './sreach' // 引入搜索组件
import Page from './page' // 引入分页组件
// import * as utils from './../utils' // utils 工具

export default {
  name: 'processedTask',
  /**
    子组件
  */
  components: {  // 注册公共组件
    Sreach, Page
  },
  /**
    model数据状态
  */
  data () {
    return {
      mock: true, // 是否为mock
      loading: true, // loading是否显示
      sreachObj: { // 用来接受子组价（搜索）的返回对象
        taskBeginDate: '', // 开始时间
        taskEndDate: '', // 结束时间
        taskSreachID: '', // 搜索任务ID
        taskSreachName: '' // 搜索任务名称
      },
      pageObj: { // 用来接受子组件（分页）的返回对象
        pageNo: 1, // 当前页
        pageSize: 10 // 每页多少
      },
      ProcessedTaskData: [], // 请求数据容器
      pageTotal: 1, // 初始页码
      etype: 1 // 任务类型
    }
  },
  /**
    ready
  */
  mounted: function () {
    this.$nextTick(function () {
      console.log(this.extendParams())
      this.getProcessedTaskData(this.extendParams()) // 页面初始化已处理的任务数据
    })
  },
  /**
    方法
  */
  methods: {
    /* 合并请求参数 */
    extendParams: function () {
      return { // 初始化传递空值
        taskBeginDate: this.sreachObj.taskBeginDate,
        taskEndDate: this.sreachObj.taskEndDate,
        taskSreachID: this.sreachObj.taskSreachID,
        taskSreachName: this.sreachObj.taskSreachName,
        pageNo: this.pageObj.pageNo,
        pageSize: this.pageObj.pageSize
      }
    },
    /* 获得已处理的任务 */
    getProcessedTaskData: function (params) {
      console.log(params)
      let _this = this
      /* http请求 */
      let api = 'http://localhost:8888/api/rhdo/done/' + _this.pageObj.pageNo + '/' + _this.pageObj.pageSize + '/' + _this.etype + '/?edates=' + _this.sreachObj.taskBeginDate + '&edaten=' + _this.sreachObj.taskEndDate + '&taskID=' + _this.sreachObj.taskSreachID + '&taskName=' + _this.sreachObj.taskSreachName
      if (!_this.mock) {
        api = 'http://localhost:2225/rhdo/done/' + _this.pageObj.pageNo + '/' + _this.pageObj.pageSize + '/' + _this.etype + '/?edates=' + _this.sreachObj.taskBeginDate + '&edaten=' + _this.sreachObj.taskEndDate + '&taskID=' + _this.sreachObj.taskSreachID + '&taskName=' + _this.sreachObj.taskSreachName
      }
      _this.axios.get(api).then(
        (res) => {
          // _this.ProcessedTaskData = utils.formatRes(res.data.list) // 去null
          _this.ProcessedTaskData = res.data.todoall.list // 去null
          _this.pageObj.pageNo = res.data.pageNo
          _this.pageObj.pageSize = res.data.pageSize
          _this.pageTotal = res.data.pageTotal
          if (_this.loading) {
            _this.loading = false // 关闭loading
          }
          console.log(_this.ProcessedTaskData)
        })
    },
    /* 搜索子组件sreach.vue触发请求 */
    getprocessedTaskDataBySreach: function (obj) { // obj为子组件传递的参数，这里为sreach.vue传递
      /* 根据子组件传递参数重设 */
      this.sreachObj = obj
      this.pageObj = { // 搜索请求无需考虑页码
        pageNo: 1, // 当前页
        pageSize: 10 // 每页多少
      }
      this.getProcessedTaskData(this.extendParams())
    },
    /* 分页子组件page.vue触发请求 */
    getprocessedTaskDataByPage: function (obj) { // obj为子组件传递的参数，这里为page.vue传递
      this.pageObj = obj // 根据子组件传递参数重设
      this.getProcessedTaskData(this.extendParams())
    }
  }
}
</script>

<style>
@import './../assets/css/table.css';
</style>

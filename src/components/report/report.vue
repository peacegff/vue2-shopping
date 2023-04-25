<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>数据统计</el-breadcrumb-item>
        <el-breadcrumb-item>数据报表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <div id="main" style="width: 600px;height:400px;"></div>
    </el-card>
  </div>
</template>

<script>
import * as echarts from 'echarts'

//let echarts = require('echarts')
export default {
  data() {
    return {
      options: { title: { text: '用户来源' }, tooltip: { trigger: 'axis', axisPointer: { type: 'cross', label: { backgroundColor: '#E9EEF3' } } }, grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true }, xAxis: [ { boundaryGap: false } ], yAxis: [ { type: 'value' } ] }
    }
  },
  created() { },
  //此时页面上的元素已经渲染完毕了
  async mounted() {
    // 基于准备好的dom，初始化echarts实例
    var myChart = echarts.init(document.getElementById('main'));
    const { data: res } = await this.$http.get('reports/type/1')
    if(res.meta.status !== 200) return this.$message.error('获取数据失败') 
    const result = {...this.options, ...res.data}
    myChart.setOption(result);
  },
  methods: {}
}
</script>

<style lang="less" scoped>

</style>

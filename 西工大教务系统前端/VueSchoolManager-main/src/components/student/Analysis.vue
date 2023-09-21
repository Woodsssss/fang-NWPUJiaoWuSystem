<template>
  <div>
    <el-card class="card_box">
      <div id="main" style="width: 1200px; height: 550px"></div>
    </el-card>
  </div>
</template>
<script>
// 1.导入echarts
import * as echarts from 'echarts'
import Vue from 'vue'

export default {
  data() {
    return {
      id:window.sessionStorage.getItem('user'),
      avgScoreList: [],
      analysisScoreList: {}
    }
  },
  created() {},
  // 此时页面上的元素渲染完毕
  mounted() {
    this.refresh()
  },
  methods: {
    // 更新
    async refresh() {
      await this.getData()
      this.drawChart()
    },
    // 获取统计数据
    async getData() {
      const { data: res } = await this.$http.get('eduacademic/academicgrade/info-sid/'+ this.id)
      console.log(res)
      if (res.code !== 200) return this.$message.error('获取数据失败')
      this.allScoreList = res.academicGrade
      var scoreList = []
      var cnameList = []
      this.allScoreList.forEach(item => {
        scoreList.push(item.grade.grades)
        cnameList.push(item.course.cname)
      });
      Vue.set(this.analysisScoreList,'grades',scoreList)
      Vue.set(this.analysisScoreList,'cname',cnameList)
    },
    // 成绩分布图
    drawChart() {
      var myChart = echarts.init(document.getElementById('main'))
      var colors = ['#5470C6', '#91CC75', '#EE6666']
      var option = {
        title: {
          text: '各门成绩情况',
          x: 'center',
        },
        tooltip: {
          trigger: 'axis',
        },
        legend: {
          orient: 'vertical',
          x: 'right',
          y: 'center',
        },
        toolbox: {
          show: true,
          feature: {
            magicType: { show: true, type: ['line', 'bar'] },
            restore: { show: true },
            saveAsImage: { show: true },
          },
        },
        calculable: true,
        // 横轴
        xAxis: [
          {
            type: 'category',
            data: this.analysisScoreList.cname,
          },
        ],
        yAxis: [
          {
            type: 'value',
            name: '分数',
            min: 0,
            max: 100,
            position: 'left',
            axisLine: {
              show: true,
              lineStyle: {
                color: colors[0],
              },
            },
          },
        ],
        series: [
          {
            type: 'bar',
            name: '课程成绩',
            data: this.analysisScoreList.grades,
          },
        ],
      }
      // 5.使用刚指定的配置项和数据显示图表。
      myChart.setOption(option)
    },
  },
}
</script>
<style lang="less" scoped>
.card_box {
  margin-top: 20px;
}
.pagination {
  margin-top: 15px;
}
</style>
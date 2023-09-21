<template>
  <div>
    <!-- 导航-->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/teacher/welcome' }"
        >首页</el-breadcrumb-item
      >
      <el-breadcrumb-item>教学相关</el-breadcrumb-item>
      <el-breadcrumb-item>课程班级成绩查询</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <!-- 搜索区域 -->
      <el-row :gutter="20">
        <el-col :span="5">
          <el-input
            placeholder="请输入课程号"
            prefix-icon="el-icon-search"
            v-model="searchQuery.course"
            clearable
            @clear="getstudentscore"
          >
          </el-input>
        </el-col>
        <el-col :span="5">
          <el-input
            placeholder="请输入教师号"
            prefix-icon="el-icon-search"
            v-model="searchQuery.teacher"
            clearable
            @clear="getstudentscore"
          >
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button @click="getstudentscore" type="primary"> 搜索 </el-button>
        </el-col>
      </el-row>
      <!--学生成绩列表区域-->
      <el-table :data="scorelistShow" border stripe :cell-style="cellStyle">
        <!-- <el-table-column
          :label="this.selected + '   学期学生成绩单'"
          align="center"
        >
        </el-table-column> -->
        <el-table-column type="index"></el-table-column>
        <el-table-column
          label="学号"
          prop="student.scode"
          align="center"
        ></el-table-column>
        <el-table-column
          label="姓名"
          prop="student.sname"
          align="center"
        ></el-table-column>
        <el-table-column
          label="课程号"
          prop="course.ccode"
          align="center"
        ></el-table-column>
        <el-table-column
          label="课程名"
          prop="course.cname"
          align="center"
        ></el-table-column>
        <el-table-column
          label="教师号"
          prop="teacher.tcode"
          align="center"
        ></el-table-column>
        <el-table-column
          label="教师名"
          prop="teacher.tname"
          align="center"
        ></el-table-column>
        <el-table-column
          label="成绩"
          prop="grade.grades"
          align="center"
        ></el-table-column>
      </el-table>
      <!--页码栏-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[1, 2, 3, 5]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
  </div>
</template>

<script>
// import html2canvas from 'html2canvas'
// import printJS from 'print-js'
export default {
  data() {
    return {
      searchQuery:{
        course: '',
        teacher: '',
      },
      id:window.sessionStorage.getItem('user'),
      selected: '', // 搜索选择
      input1: '',
      input2: '',
      scorelist: [],
      scorelistShow: [],
      total: 0, // 列表总数
      currentPage: 1, // 当前页面
      pageSize: 3, // 每页展示列表数
    }
  },
  created() {
    this.getstudentscore()
  },
  methods: {
    async getstudentscore() {
      if(this.searchQuery.course!=""&&this.searchQuery.teacher!=""){
        const { data: res } = await this.$http.post('eduacademic/academicgrade/info-both', {
          ccode:this.searchQuery.course,tcode:this.searchQuery.teacher
        })
        if (res.code !== 200) return this.$message.error('获取学生成绩列表失败')
        this.scorelist = res.academicGrade
        this.total = res.total
      }
      else if(this.searchQuery.course!=""){
        const { data: res } = await this.$http.post('eduacademic/academicgrade/info-ccode/'+this.searchQuery.course)
        if (res.code !== 200) return this.$message.error('获取学生成绩列表失败')
        this.scorelist = res.academicGrade
        this.total = res.total
      }
      else if(this.searchQuery.teacher!=""){
        const { data: res } = await this.$http.post('eduacademic/academicgrade/info-tcode/'+this.searchQuery.teacher)
        if (res.code !== 200) return this.$message.error('获取学生成绩列表失败')
        this.scorelist = res.academicGrade
        this.total = res.total
      }else{
        const { data: res } = await this.$http.post('eduacademic/academicgrade/info',{})
        console.log(res)
        if (res.code !== 200) return this.$message.error('获取学生成绩列表失败')
        this.scorelist = res.academicGrade
        this.total = res.total
      }
      this.currentPage = 1
      var start = 0
      var end = start + this.pageSize
      this.scorelistShow = this.scorelist.slice(start, end)
    },
    handleSizeChange(pageSize) {
      this.pageSize = pageSize
      let start = 0
      let end = start + pageSize
      this.scorelistShow = this.scorelist.slice(start, end)
    },
    // 监听当前页面变化
    handleCurrentChange(currentPage) {
      this.currentPage = currentPage
      let start = (currentPage - 1) * this.pageSize
      let end = start + this.pageSize
      this.scorelistShow = this.scorelist.slice(start, end)
    },
    cellStyle(row, column, rowIndex, columnIndex) {
      if (row.column.label == '成绩' && row.row.score < 60) {
        return 'color:#FF6930'
      }
    },
  },
}
</script>

<style lang="less" scoped>
.demo-input-label {
  display: inline-block;
  width: 130px;
}
</style>
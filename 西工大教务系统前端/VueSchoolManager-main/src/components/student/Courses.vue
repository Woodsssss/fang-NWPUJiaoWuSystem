<template>
  <div>
    <!-- 卡片视图 -->
    <el-card class="box-card">
      <!-- 学生课程查询视图 -->
      <el-form :inline="true">
        <el-form-item label="课程号">
          <el-input
            placeholder="请输入课程号"
            v-model="searchQuery.course"
          ></el-input>
        </el-form-item>
        <el-form-item label="教师号">
          <el-input
            placeholder="请输入教师号"
            v-model="searchQuery.teacher"
          ></el-input>
        </el-form-item>
        <el-form-item>
          <el-button
              type="primary"
              icon="el-icon-refresh-right"
              @click="clearCourseList(false)"
          >刷新</el-button>
          <el-button type="primary" icon="el-icon-search" @click="getCourseList()">查询课程</el-button>
        </el-form-item>
      </el-form>

      <!-- 列表区域 -->
      <el-table :data="courseListShow" border stripe>
        <!-- 自定义索引 -->
        <el-table-column type="index"> </el-table-column>
        <el-table-column
          prop="course.ccode"
          label="课程号"
        ></el-table-column>
        <el-table-column
          prop="course.cname"
          label="课程名"
        ></el-table-column>
        <el-table-column
          prop="course.time"
          label="上课时间"
        ></el-table-column>
        <el-table-column
          prop="course.credit"
          label="学分"
        ></el-table-column>
        <el-table-column
          prop="teacher.tcode"
          label="教师号"
        ></el-table-column>
        <el-table-column
          prop="teacher.tname"
          label="教师名"
        ></el-table-column>
        <el-table-column prop="grade.grades" label="成绩"></el-table-column>
        <el-table-column label="教学大纲">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-download" @click="getUrl(scope.row.course.ccode)">点击下载教学大纲</el-button>
          </template>

        </el-table-column>
      </el-table>
      <!-- 分页栏 -->
      <el-pagination
        class="pagination"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[5, 10, 15]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
  </div>
</template>
<script>
import axios from 'axios'

export default {
  data() {
    return {
      id:window.sessionStorage.getItem('user'),
      searchQuery: {
        course: '',
        teacher: '',
      },
      courseList: [], // 课程列表
      courseListShow: [], // 展示的课程列表
      total: 0, // 课程列表总数
      currentPage: 1, // 当前页面
      pageSize: 5, // 每页展示列表数
    }
  },
  created() {
    this.getCourseList()
  },
  methods: {
    clearCourseList(){
      this.searchQuery.course=""
      this.searchQuery.teacher=""
      this.getCourseList()
    },
    // 获取url
    async getUrl(ccode){
      // console.log(ccode)
      let urlDown = 'eduacademic/academiccourse/getUrl?ccode='
      urlDown = urlDown + ccode
      axios({
        method:'GET',
        url: urlDown,  
      }).then(res=>{
        console.log(res)
        if(res.data.code===200){
          this.$message({
                  message: "下载成功",
                  type: "success",
          });
          let url = res.data.url
          window.open(url)
        }
      })
    },

    // 获取课程列表
    async getCourseList() {
      if(this.searchQuery.course!=""&&this.searchQuery.teacher!=""){
        const { data: res } = await this.$http.post('eduacademic/academicgrade/info-sccode/', {sid:this.id,ccode:this.searchQuery.course,tcode:this.searchQuery.teacher})  //修改
        if (res.code !== 200) return this.$message.error('获取课程信息列表失败')
        this.courseList = res.academicGrade //修改
        this.total = res.total
      }
      else if(this.searchQuery.course!=""){
        const { data: res } = await this.$http.post('eduacademic/academicgrade/info-sccode/', {sid:this.id,ccode:this.searchQuery.course})
        if (res.code !== 200) return this.$message.error('获取成绩信息列表失败')  
        this.courseList = res.academicGrade //修改
        this.total = res.total
      }
      else if(this.searchQuery.teacher!=""){
        const { data: res } = await this.$http.post('eduacademic/academicgrade/info-stcode/', {sid:this.id,tcode:this.searchQuery.teacher})
        if (res.code !== 200) return this.$message.error('获取成绩信息列表失败')
        this.courseList = res.academicGrade 
        this.total = res.total
      }else{
        const { data: res } = await this.$http.get('eduacademic/academicgrade/info-sid/'+this.id)
        if (res.code !== 200) return this.$message.error('获取成绩信息列表失败')
        this.courseList = res.academicGrade 
        this.total = res.total
      }
      this.currentPage = 1
      var start = 0
      var end = start + this.pageSize
      this.courseListShow = this.courseList.slice(start, end)
      // console.log(res)
    },
    // 监听分页变化
    handleSizeChange(pageSize) {
      this.pageSize = pageSize
      let start = 0
      let end = start + pageSize
      this.scoreListShow = this.courseList.slice(start, end)
    },
    // 监听当前页面变化
    handleCurrentChange(currentPage) {
      this.currentPage = currentPage
      let start = (currentPage - 1) * this.pageSize
      let end = start + this.pageSize
      this.scoreListShow = this.courseList.slice(start, end)
    },
  },
}
</script>
<style lang="less" scoped>
.select {
  width: 120px;
}
.pagination {
  margin-top: 15px;
}
.text {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
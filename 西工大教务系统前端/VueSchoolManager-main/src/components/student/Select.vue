<template>
  <div>
    <!-- 查询开课 -->
    <el-card class="box-card">
      <!-- 查询区域 -->
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
          <el-button type="primary" icon="el-icon-search" @click="getCourseList()"
            >查询课程</el-button
          >
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
        <el-table-column prop="course.credit" label="学分"></el-table-column>
        <el-table-column
          prop="teacherListCode"
          label="教师号"
        ></el-table-column>
        <el-table-column
          prop="teacherList"
          label="教师名"
        ></el-table-column>
        <el-table-column prop="course.place" label="上课地点"></el-table-column>
        <el-table-column prop="course.time" label="上课时间"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-circle-plus-outline"
              size="mini"
              @click="addCourse(scope.row)"
            >
            </el-button>
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
    <!-- 已选选列表 -->
    <el-card class="box-card">
      <div><h1>已选课程</h1></div>
      <!-- 列表区域 -->
      <el-table :data="SelectListShow" border stripe>
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
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="deleteCourse(scope.row)"
            >
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>
<script>
export default {
  data() {
    return {
      id: window.sessionStorage.getItem('user'),
      searchQuery: {
        course: '',
        teacher: '',
      },
      courseList: [], // 选课列表
      courseListShow: [], // 展示的选课列表
      total: 0, // 选课列表总数
      currentPage: 1, // 当前页面
      pageSize: 5, // 每页展示列表数
      SelectListShow: [], // 已选课程
    }
  },
  created() {
    this.getCourseList()
    this.getSelectCourseList()
  },
  methods: {
    clearCourseList(){
      this.searchQuery.course=""
      this.searchQuery.teacher=""
      this.getCourseList()
    },
    // 获取选课列表
    async getCourseList() {
      if(this.searchQuery.course!=""&&this.searchQuery.teacher!=""){
        const { data: res } = await this.$http.post('eduacademic/academiccourse/info', {ccode:this.searchQuery.course,tcode:this.searchQuery.teacher})  
        if (res.code !== 200) return this.$message.error('获取课程信息列表失败')
        this.courseList = res.academicCourse 
        var teachers = []
        var teachersCode= []
        this.courseList.forEach(item=>{
          teachers = []
          teachersCode = []
          item.teachers.forEach(item1=>{
            teachers.push(item1.tname)
            teachersCode.push(item1.tcode)
          })
          this.$set(item,'teacherList',teachers)
          this.$set(item,'teacherListCode',teachersCode)
        })
        this.total = res.total
      }
      else if(this.searchQuery.course!=""){
        const { data: res } = await this.$http.get('eduacademic/academiccourse/info-ccode/'+ this.searchQuery.course) 
        if (res.code !== 200) return this.$message.error('获取课程信息列表失败')
        this.courseList = res.academicCourse 
        var teachers = []
        var teachersCode= []
        this.courseList.forEach(item=>{
          teachers = []
          teachersCode = []
          item.teachers.forEach(item1=>{
            teachers.push(item1.tname)
            teachersCode.push(item1.tcode)
          })
          this.$set(item,'teacherList',teachers)
          this.$set(item,'teacherListCode',teachersCode)
        })
        this.total = res.total
      }
      else if(this.searchQuery.teacher!=""){
        const { data: res } = await this.$http.get('eduacademic/academiccourse/info-tcode/'+ this.searchQuery.teacher)
        if (res.code !== 200) return this.$message.error('获取课程信息列表失败')
        this.courseList = res.academicCourse 
        var teachers = []
        var teachersCode= []
        this.courseList.forEach(item=>{
          teachers = []
          teachersCode = []
          item.teachers.forEach(item1=>{
            teachers.push(item1.tname)
            teachersCode.push(item1.tcode)
          })
          this.$set(item,'teacherList',teachers)
          this.$set(item,'teacherListCode',teachersCode)
        })
        this.total = res.total
      }else{
        const { data: res } = await this.$http.post('eduacademic/academiccourse/info/',{})
        if (res.code !== 200) return this.$message.error('获取课程信息列表失败')
        this.courseList = res.academicCourse
        var teachers = []
        var teachersCode= []
        this.courseList.forEach(item=>{
          teachers = []
          teachersCode = []
          item.teachers.forEach(item1=>{
            teachers.push(item1.tname)
            teachersCode.push(item1.tcode)
          })
          this.$set(item,'teacherList',teachers)
          this.$set(item,'teacherListCode',teachersCode)
        })
        this.total = res.total
      }
      this.currentPage = 1
      var start = 0
      var end = start + this.pageSize
      this.courseListShow = this.courseList.slice(start, end)
    },
    // 获取已选列表
    async getSelectCourseList() {
      const { data: res } = await this.$http.get('eduacademic/academicselcourse/info-sid/'+ this.id)
      if (res.code !== 200) return this.$message.error('获取选课列表失败')
      this.SelectListShow = res.academicSelCourse
    },
    // 添加选课
    async addCourse(row) {
      // console.log(row)
      const { data: res } = await this.$http.post(
        'eduacademic/academicselcourse/save-sid' , {id:this.id,ccode:row.course.ccode}
      )
      if (res.code === 200) this.$message.success('添加成功！')
      else this.$message.error('添加失败！\n' + '已选过该课程')
      // 更新选课列表
      this.getSelectCourseList()
    },
    // 删除选课
    deleteCourse(row) {
      // 弹框提示
      this.$confirm('此操作将永久删除该选课, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      })
        .then(async () => {
          // 确定删除
          const { data: res } = await this.$http.post('eduacademic/academicselcourse/delete-sid',{
              id: this.id, ccode: row.course.ccode
            }
          )
          if (res.code != 200)
            return this.$message({
              type: 'error',
              message: '删除失败!',
            })
          this.$message({
            type: 'success',
            message: '删除成功!',
          })
          // 刷新选课列表
          this.getSelectCourseList()
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除',
          })
        })
    },
    // 监听分页变化
    handleSizeChange(pageSize) {
      this.pageSize = pageSize
      let start = 0
      let end = start + pageSize
      this.courseListShow = this.courseList.slice(start, end)
    },
    // 监听当前页面变化
    handleCurrentChange(currentPage) {
      this.currentPage = currentPage
      let start = (currentPage - 1) * this.pageSize
      let end = start + this.pageSize
      this.courseListShow = this.courseList.slice(start, end)
    },
  },
}
</script>
<style lang="less" scoped>
.box-card {
  margin-top: 10px;
}
.select {
  width: 120px;
}
.pagination {
  margin-top: 15px;
}
</style>
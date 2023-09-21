<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/admin/home' }"
        >首页</el-breadcrumb-item
      >
      <el-breadcrumb-item>教务管理</el-breadcrumb-item>
      <el-breadcrumb-item>选课列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card class="box-card">
      <!-- 学生选课结果查询视图 -->
      <el-form :inline="true">
        <el-form-item label="课程号">
          <el-input
            placeholder="请输入课程号"
            v-model="searchQuery.course"
          ></el-input>
        </el-form-item>
        <el-form-item label="学号">
          <el-input
            placeholder="请输入学号"
            v-model="searchQuery.student"
          ></el-input>
        </el-form-item>
        <!-- <el-form-item label="教师号">
          <el-input
            placeholder="请输入教师号"
            v-model="searchQuery.teacher"
          ></el-input>
        </el-form-item> -->
        <el-form-item>
          <el-button
              type="primary"
              icon="el-icon-refresh-right"
              @click="clearSelectionList(false)"
          >刷新</el-button>
          <el-button type="primary" icon="el-icon-search" @click="getSelectionList(true)">查询选课
          </el-button>
          <el-button type="primary" icon="el-icon-plus" @click="addDialogVisible = true">
            添加课程
          </el-button>
        </el-form-item>
      </el-form>

      <!-- 列表区域 -->
      <el-table :data="courseListShow" border stripe>
        <!-- 自定义索引 -->
        <el-table-column type="index"> </el-table-column>
        <el-table-column
          prop="student.scode"
          label="学号"
        ></el-table-column>
        <el-table-column
          prop="student.sname"
          label="姓名"
        ></el-table-column>
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
            <!-- <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.id)"
            ></el-button> -->
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="deleteSelection(scope.row.id)"
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
    <!-- 添加选课 -->
    <el-dialog
      title="添加课程" 
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 内容主题区域 -->
      <el-form label-position="left"
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="90px"
      >
        <el-form-item label="学号" prop="scode">
          <el-input
            v-model="addForm.scode"
            placeholder="请输入学生学号"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="课程号" prop="ccode">
          <el-input
            v-model="addForm.ccode"
            placeholder="请输入课程号"
            clearable
          ></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addSelection"> 确 定 </el-button>
      </span>
    </el-dialog>
    <!-- 编辑选课 -->
    <el-dialog
      title="编辑选课"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <!-- 内容主题区域 -->
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="90px"
      >
        <el-form-item label="学号" prop="scode">
          <el-input
            v-model="editForm.scode"
            placeholder="请输入学号"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="课程号" prop="ccode">
          <el-input
            v-model="editForm.ccode"
            placeholder="请输入课程号"
            clearable
          ></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editSelection"> 确 定 </el-button>
      </span>
    </el-dialog>
    
  </div>
</template>
<script>

export default {
  data() {
    return {
      searchQuery: {
        course: '',
        student:''
      },
      scoreList: [], // 选课列表
      scoreListShow: [], // 展示的选课列表
      courseList:[],
      courseListShow: [],
      total: 0, // 选课列表总数
      currentPage: 1, // 当前页面
      pageSize: 5, // 每页展示列表数
      // 控制添加对话框显示与隐藏
      addDialogVisible: false,
      addForm:{
        scode: '',
        ccode: '',
      },
      addFormRules:{
        scode: [
          { required: true, message: '请输入学生号', trigger: 'blur' },
          {
            min: 1,
            max: 10,
            message: '学生号长度在1~10个数字之间',
            trigger: 'blur',
          },
        ],     
        ccode: [
          { required: true, message: '请输入课程号', trigger: 'blur' },
          {
            min: 2,
            max: 6,
            message: '课程号长度在2~6个数字之间',
            trigger: 'blur',
          },
        ],
      },
      editDialogVisible: false,
      // 添加选课的表单数据
      editForm: {
        id:'',
        scode: '',
        ccode: '',
      },
      // 添加表单的验证规则
      editFormRules: {
        scode: [
          { required: true, message: '请输入学生号', trigger: 'blur' },
          {
            min: 1,
            max: 10,
            message: '学生号长度在1~10个数字之间',
            trigger: 'blur',
          },
        ],     
        ccode: [
          { required: true, message: '请输入课程号', trigger: 'blur' },
          {
            min: 2,
            max: 6,
            message: '课程号长度在2~6个数字之间',
            trigger: 'blur',
          },
        ],
      },
    }
  },
  created() {
    this.getSelectionList()
  },
  methods: {
    // 获取选课列表
    clearSelectionList(){
      this.searchQuery.course=""
      this.searchQuery.student=""
      this.getSelectionList()
    },
    async getSelectionList() {
      if(this.searchQuery.course!=""&&this.searchQuery.student!=""){
        const { data: res } = await this.$http.post('eduacademic/academicselcourse/info', {ccode:this.searchQuery.course,scode:this.searchQuery.student})
        if (res.code !== 200) return this.$message.error('获取选课列表失败')
        this.courseList = res.academicSelCourse
        this.total = res.total
      }
      else if(this.searchQuery.course!=""){
        // console.log(this.searchQuery.course)
        const { data: res } = await this.$http.get('eduacademic/academicselcourse/info-ccode/'+ this.searchQuery.course)
        if (res.code !== 200) return this.$message.error('获取选课列表失败')
        this.courseList = res.academicSelCourse
        this.total = res.total
      }
      else if(this.searchQuery.student!=""){
        const { data: res } = await this.$http.get('eduacademic/academicselcourse/info-scode/'+ this.searchQuery.student)
        if (res.code !== 200) return this.$message.error('获取选课列表失败')
        this.courseList = res.academicSelCourse
        this.total = res.total
      }else{
        const { data: res } = await this.$http.post('eduacademic/academicselcourse/info/',{})
        // console.log(res)
        if (res.code !== 200) return this.$message.error('获取选课列表失败') 
        this.courseList = res.academicSelCourse
        this.total = res.total
      }
      this.currentPage = 1
      var start = 0
      var end = start + this.pageSize
      this.courseListShow = this.courseList.slice(start, end)
    },
    addDialogClosed(){
      this.$refs.addFormRef.resetFields()
    },
    // 添加课程
    addSelection() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        // 校验正确则发起请求
        const { data: res } = await this.$http.post('/eduacademic/academicselcourse/save', {ccode:this.addForm.ccode,scode:this.addForm.scode})
        if (res.code === 200) this.$message.success('添加成功！')
        else this.$message.error('添加失败！\n')
        this.addDialogVisible = false
        // 重新获取课程列表
        this.getSelectionList()
      })
    },
    async showEditDialog(id) {
      // console.log(id)
      this.editForm.id = id
      this.editDialogVisible = true
      //   console.log(res)
    },
    // 监听修改对话框的关闭事件
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 修改课程信息
    editSelection() {
      this.$refs.editFormRef.validate(async (valid) => {
        // console.log(valid)
        if (!valid) return
        // 校验正确则发起请求
        const { data: res } = await this.$http.post('/eduacademic/academicselcourse/update', 
        {id: this.editForm.id,ccode:this.editForm.ccode,scode:this.editForm.scode})
        console.log(this.editForm.ccode,this.editForm.scode)
        if (res.code !== 200)
          this.$message.error('修改失败！\n' + res.detail[0])
        // console.log(res)
        this.$message.success('更新成功！')
        this.editDialogVisible = false
        // 重新获取课程列表
        this.getSelectionList()
      })
    },
    // 删除选课
    deleteSelection(id) {
      // 弹框提示
      this.$confirm('此操作将永久删除该选课, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      })
        .then(async () => {
          // 确定删除

          const { data: res } = await this.$http.post('eduacademic/academicselcourse/delete', [id])
          console.log([id])     
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
          this.getSelectionList()
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
      this.courseListShow = this.scoreList.slice(start, end)
    },
    // 监听当前页面变化
    handleCurrentChange(currentPage) {
      this.currentPage = currentPage
      let start = (currentPage - 1) * this.pageSize
      let end = start + this.pageSize
      this.courseListShow = this.scoreList.slice(start, end)
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
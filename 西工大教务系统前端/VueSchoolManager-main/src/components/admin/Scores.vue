<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/admin/home' }"
        >首页</el-breadcrumb-item
      >
      <el-breadcrumb-item>教务管理</el-breadcrumb-item>
      <el-breadcrumb-item>成绩管理</el-breadcrumb-item>
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
        <el-form-item>
          <el-button
              type="primary"
              icon="el-icon-refresh-right"
              @click="clearScoreList(false)"
          >刷新</el-button>
          <el-button type="primary" icon="el-icon-search" @click="getScoreList()">查询成绩</el-button>
          <el-button type="primary" icon="el-icon-plus" @click="addDialogVisible = true">
            添加成绩
          </el-button>
        </el-form-item>
      </el-form>

      <!-- 列表区域 -->
      <el-table :data="scoreListShow" border stripe>
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
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row)"
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
    <!-- 添加成绩对话框 -->
    <el-dialog
      title="添加学生成绩信息"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 内容主题区域 -->
      <el-form
        label-position="left"
        :model="editForm"
        :rules="editFormRules"
        ref="addFormRef"
        label-width="90px"
      >
        <el-form-item label="学号">
          <el-input v-model="addForm.scode"></el-input>
        </el-form-item>
        <el-form-item label="课程号">
          <el-input v-model="addForm.ccode"></el-input>
        </el-form-item>
        <el-form-item label="成绩">
          <el-input v-model.number="addForm.score"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addScore"> 确 定 </el-button>
      </span>
    </el-dialog>
    <!-- 修改成绩对话框 -->
    <el-dialog
      title="修改开课信息"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <!-- 内容主题区域 -->
      <el-form
        label-position="left"
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="90px"
      >
        <el-form-item label="学号">
          <el-input v-model="editForm.scode" disabled></el-input>
        </el-form-item>
        <el-form-item label="课程号">
          <el-input v-model="editForm.ccode" disabled></el-input>
        </el-form-item>
        <el-form-item label="成绩">
          <el-input v-model.number="editForm.score"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editScore"> 确 定 </el-button>
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
        student: '',
      },
      scoreList: [], // 选课列表
      scoreListShow: [], // 展示的选课列表
      total: 0, // 选课列表总数
      currentPage: 1, // 当前页面
      pageSize: 5, // 每页展示列表数
      addDialogVisible:false,
      addForm:{
        scode:'',
        ccode:'',
        score:0.0,
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
        score:[
          { required: true, message: '请输入成绩', trigger: 'blur' },
          {
            type: 'number',
            message: '成绩为数字',
            trigger: 'blur',
          },
        ]
      },
      // 控制修改对话框显示与隐藏
      editDialogVisible: false,
      // 修改选课的表单数据
      editForm: {
        id:'',
        scode: '',
        ccode:'',
        score: 0.0,
      },
      // 修改表单的验证规则
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
        score: [
          { required: true, message: '请输入成绩', trigger: 'blur' },
          {
            type: 'number',
            message: '成绩为数字',
            trigger: 'blur',
          },
        ],
      },
    }
  },
  created() {
    this.getScoreList()
  },
  methods: {
    clearScoreList(){
      this.searchQuery.course="",
      this.searchQuery.student="",
      this.getScoreList()
    },
    // 获取选课列表
    async getScoreList() {
      if(this.searchQuery.course!=""&&this.searchQuery.student!=""){
        const { data: res } = await this.$http.post('eduacademic/academicgrade/info', {ccode:this.searchQuery.course,scode:this.searchQuery.student})
        if (res.code !== 200) return this.$message.error('获取成绩列表失败')
        this.scoreList = res.academicGrade
        this.total = res.total
      }
      else if(this.searchQuery.course!=""){
        const { data: res } = await this.$http.get('eduacademic/academicgrade/info-ccode/'+ this.searchQuery.course)
        if (res.code !== 200) return this.$message.error('获取成绩列表失败')
        this.scoreList = res.academicGrade
        this.total = res.total
      }
      else if(this.searchQuery.student!=""){
        const { data: res } = await this.$http.get('eduacademic/academicgrade/info-scode/'+ this.searchQuery.student)
        if (res.code !== 200) return this.$message.error('获取成绩列表失败')
        this.scoreList = res.academicGrade
        this.total = res.total
      }
      else{
        const { data: res } = await this.$http.post('eduacademic/academicgrade/info/',{})
        if (res.code !== 200) return this.$message.error('获取成绩列表失败')
        this.scoreList = res.academicGrade
        this.total = res.total
      }
      this.currentPage = 1
      var start = 0
      var end = start + this.pageSize
      this.scoreListShow = this.scoreList.slice(start, end)
      // console.log(res)
    },
    addDialogClosed(){
      this.$refs.addFormRef.resetFields()
    },
    // 添加成绩
    addScore() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        // 校验正确则发起请求
        const { data: res } = await this.$http.post('/eduacademic/academicgrade/save', {ccode:this.addForm.ccode,scode:this.addForm.scode,grades:this.addForm.score})
        if (res.code === 200) this.$message.success('添加成功！')
        else this.$message.error('添加失败！\n')
        this.addDialogVisible = false
        // 重新获取课程列表
        this.getScoreList()
      })
    },
    // 根据开课号查询开课信息
    async showEditDialog(row) {
      this.editDialogVisible = true
      this.editForm.id = row.id
      this.editForm.scode = row.student.scode
      this.editForm.ccode = row.course.ccode 
      // console.log(res)
    },
    // 监听修改对话框的关闭事件
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 修改开课信息
    async editScore() {
      this.$refs.editFormRef.validate(async (valid) => {
        // console.log(valid)
        if (!valid) return
        // 校验正确则发起请求
        console.log({id: this.editForm.id, grades:this.editForm.score})
        const { data: res } = await this.$http.post('/eduacademic/academicgrade/update', 
        {id: this.editForm.id, grades:this.editForm.score})
        if (res.code !== 200)
          this.$message.error('修改失败！\n' + res.detail[0])
        console.log(res)
        this.$message.success('更新成功！')
        this.editDialogVisible = false
        // 重新获取开课列表
        this.getScoreList()
      })
    },
    // 监听分页变化
    handleSizeChange(pageSize) {
      this.pageSize = pageSize
      let start = 0
      let end = start + pageSize
      this.scoreListShow = this.scoreList.slice(start, end)
    },
    // 监听当前页面变化
    handleCurrentChange(currentPage) {
      this.currentPage = currentPage
      let start = (currentPage - 1) * this.pageSize
      let end = start + this.pageSize
      this.scoreListShow = this.scoreList.slice(start, end)
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
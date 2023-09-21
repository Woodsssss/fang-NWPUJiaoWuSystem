<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/admin/home' }"
        >首页</el-breadcrumb-item
      >
      <el-breadcrumb-item>学生管理</el-breadcrumb-item>
      <el-breadcrumb-item>学生</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card class="box-card">
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="16">
          <!-- 搜索与添加区域 -->
          <el-input
            :placeholder="'请输入' + selected"
            v-model="input"
            clearable
          >
            <el-select
              v-model="selected"
              class="select"
              slot="prepend"
              placeholder="请选择"
            >
              <el-option
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getStudentList(true)"
            ></el-button>
            <el-button
              slot="append"
              icon="el-icon-refresh-right"
              @click="getStudentList(false)"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true"
            >添加学生</el-button
          >
        </el-col>
      </el-row>
      <!-- 列表区域 -->
      <el-table :data="studentListShow" border stripe>
        <!-- 自定义索引 -->
        <el-table-column type="index"> </el-table-column>
        <el-table-column prop="scode" label="学号"> </el-table-column>
        <el-table-column prop="sname" label="姓名"></el-table-column>
        <el-table-column
          prop="classId"
          label="班级"
        ></el-table-column>
        <el-table-column prop="telephone" label="联系方式"></el-table-column>
        <el-table-column prop="college" label="院系名"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.id)"
            >
            </el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="deleteStudent(scope.row.id)"
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
    <!-- 添加学生对话框 -->
    <el-dialog
      title="添加学生"
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
            placeholder="请输入学号"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="姓名" prop="sname">
          <el-input
            v-model="addForm.sname"
            placeholder="请输入姓名"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="passwords">
          <el-input
            v-model="addForm.passwords"
            placeholder="请输入密码"
            show-password
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="联系方式" prop="telephone">
          <el-input
            v-model="addForm.telephone"
            placeholder="请输入联系方式"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="班级" prop="classId">
          <el-input
            v-model="addForm.classId"
            placeholder="请输入班级"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="院系" prop="college">
          <el-input
            v-model="addForm.college"
            placeholder="请输入院系号"
            clearable
          ></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addStudent"> 确 定 </el-button>
      </span>
    </el-dialog>
    <!-- 修改学生对话框 -->
    <el-dialog
      title="修改学生信息"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <!-- 内容主题区域 -->
      <el-form label-position="left"
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="90px"
      >
        <el-form-item label="学号" prop="scode">
          <el-input v-model="editForm.scode" 
            placeholder="请输入姓名"
            clearable></el-input>
        </el-form-item>
        <el-form-item label="姓名" prop="sname">
          <el-input
            v-model="editForm.sname"
            placeholder="请输入姓名"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="班级" prop="class_id">
          <el-input
            v-model="editForm.class_id"
            placeholder="请输入班级"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="院系" prop="college">
            <el-input
            v-model="editForm.college"
            placeholder="请输入学院号"
            clearable
          ></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editStudent"> 确 定 </el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      options: [
        {
          value: '学号',
          label: '学号',
        },
        {
          value: '班级名',
          label: '班级名',
        },
        {
          value: '学院号',
          label: '学院号',
        },
      ],
      selected: '学号', // 搜索选择
      input: '', // 搜索输入框
      studentList: [], // 学生列表
      studentListShow: [], // 展示的学生列表
      total: 0, // 学生列表总数
      currentPage: 1, // 当前页面
      pageSize: 5, // 每页展示列表数
      // 控制添加对话框显示与隐藏
      addDialogVisible: false,
      // 添加学生的表单数据
      addForm: {
        scode: '',
        sname: '',
        passwords: '',
        telephone:'',
        classId: '',
        college: '',
      },
      // 添加表单的验证规则
      addFormRules: {
        scode: [
          { required: true, message: '请输入学号', trigger: 'blur' },
          {
            min: 10,
            max: 10,
            message: '学号长度为8个字符',
            trigger: 'blur',
          },
        ],
        sname: [
          { required: true, message: '请输入姓名', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '姓名长度在3~10个字符之间',
            trigger: 'blur',
          },
        ],
        passwords: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 6,
            max: 10,
            message: '密码长度在6~10个字符之间',
            trigger: 'blur',
          },
        ],
        telephone: [
        { required: true, message: '请输入联系方式', trigger: 'blur' },
          {
            min: 11,
            max: 11,
            message: '电话长度为11个字符',
            trigger: 'blur',
          },
        ],
        classId: [
        { required: true, message: '请输入班级', trigger: 'blur' },
          {
            min: 2,
            max: 6,
            message: '班级长度在4~6个字符之间',
            trigger: 'blur',
          },
        ],
        college: [{ required: true, message: '请选择院系号', trigger: 'blur' }],
      },
      // 控制修改对话框显示与隐藏
      editDialogVisible: false,
      // 查询到的学生信息
      editForm: {
        id:'',
        scode: '',
        sname: '',
        class_id: '',
        college: '',
      },
      // 修改表单的验证规则
      editFormRules: {
        sname: [
          { required: false, message: '请输入姓名', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '姓名长度在3~10个字符之间',
            trigger: 'blur',
          },
        ]
      },
    }
  },
  created() {
    this.getStudentList()
  },
  methods: {
    // 获取学生列表
    async getStudentList(search) {
      var query = {}
      if (search) {
        if (this.selected === '学号') {
          query = { scode: this.input }
        } else if (this.selected === '班级名') {
          query = { classId: this.input }
        } else if (this.selected === '学院号') {
          query = { college: this.input }
        }
      }

      if(typeof query.scode != "undefined"){
        const { data: res } = await this.$http.get('/edustaff/staffstudent/info-scode/' + query.scode)
        if (res.code !== 200) return this.$message.error('获取学生列表失败')
        this.studentList = res.staffStudent
        this.total = res.total
      }else if(typeof query.classId != "undefined"){
        const { data: res } = await this.$http.get('/edustaff/staffstudent/info-classId/' + query.classId)
        if (res.code !== 200) return this.$message.error('获取学生列表失败')
        this.studentList = res.staffStudent
        this.total = res.total
      }else if(typeof query.college != "undefined"){
        const { data: res } = await this.$http.get('/edustaff/staffstudent/info-college/' + query.college)
        if (res.code !== 200) return this.$message.error('获取学生列表失败')
        this.studentList = res.staffStudent
        this.total = res.total
      }else{
        const { data: res } = await this.$http.get('/edustaff/staffstudent/info/')
        if (res.code !== 200) return this.$message.error('获取学生列表失败')
        this.studentList = res.page.list
        this.total = res.total
      }
      this.currentPage = 1
      var start = 0
      var end = start + this.pageSize
      this.studentListShow = this.studentList.slice(start, end)
      // console.log(res)
    },
    // 监听对话框关闭事件
    addDialogClosed() {
      // 重置表单
      this.$refs.addFormRef.resetFields()
    },
    // 添加学生
    addStudent() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        // 校验正确则发起请求
        const { data: res } = await this.$http.post('/edustaff/staffstudent/save', this.addForm)
        if (res.code === 200) this.$message.success('添加成功！')
        else this.$message.error('添加失败！\n')
        this.addDialogVisible = false
        // 重新获取学生列表
        this.getStudentList()
      })
    },
    // 修改学生信息
    async showEditDialog(id) {
      console.log(id)
      this.editForm.id = id
      this.editDialogVisible = true

      // console.log(res)
    },
    // 监听修改对话框的关闭事件
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 修改学生信息
    editStudent() {
      this.$refs.editFormRef.validate(async (valid) => {
        // console.log(valid)
        if (!valid) return
        // 校验正确则发起请求
        const { data: res } = await this.$http.post('/edustaff/staffstudent/update', {id:this.editForm.id, scode:this.editForm.scode, sname: this.editForm.sname, class_id:this.editForm.class_id,college:this.editForm.college})
        if (res.code !== 200) return this.$message.error('获取学生列表失败')
        this.$message.success('更新成功！')
        this.editDialogVisible = false
        // 重新获取学生列表
        this.getStudentList()
      })
    },
    // 删除学生
    deleteStudent(id) {
      // 弹框提示
      this.$confirm('此操作将永久删除该学生, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      })
        .then(async () => {
          // 确定删除
          const { data: res } = await this.$http.post(
            '/edustaff/staffstudent/delete' , [id]
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
          // 刷新学生列表
          this.getStudentList()
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
      this.studentListShow = this.studentList.slice(start, end)
    },
    // 监听当前页面变化
    handleCurrentChange(currentPage) {
      this.currentPage = currentPage
      let start = (currentPage - 1) * this.pageSize
      let end = start + this.pageSize
      this.studentListShow = this.studentList.slice(start, end)
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
</style>
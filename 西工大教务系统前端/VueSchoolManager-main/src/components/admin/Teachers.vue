<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/admin/home' }"
        >首页</el-breadcrumb-item
      >
      <el-breadcrumb-item>教师管理</el-breadcrumb-item>
      <el-breadcrumb-item>教师</el-breadcrumb-item>
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
              @click="getTeacherList(true)"
            ></el-button>
            <el-button
              slot="append"
              icon="el-icon-refresh-right"
              @click="getTeacherList(false)"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true"
            >添加教师</el-button
          >
        </el-col>
      </el-row>
      <!-- 列表区域 -->
      <el-table :data="teacherListShow" border stripe>
        <!-- 自定义索引 -->
        <el-table-column type="index"> </el-table-column>
        <el-table-column prop="tcode" label="工号"> </el-table-column>
        <el-table-column prop="tname" label="姓名"></el-table-column>
        <el-table-column prop="title" label="职位"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="college" label="院系号"></el-table-column>
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
              @click="deleteTeacher(scope.row.id)"
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
    <!-- 添加教师对话框 -->
    <el-dialog
      title="添加教师"
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
        <el-form-item label="工号" prop="tcode">
          <el-input
            v-model="addForm.tcode"
            placeholder="请输入工号"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="姓名" prop="tname">
          <el-input
            v-model="addForm.tname"
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
        <el-form-item label="职位" prop="title">
          <el-input
            v-model="addForm.title"
            placeholder="请输入职位"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="院系" prop="college">
          <el-input
            v-model="addForm.college"
            placeholder="请输入院系"
            clearable
          ></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addTeacher"> 确 定 </el-button>
      </span>
    </el-dialog>
    <!-- 修改教师对话框 -->
    <el-dialog
      title="修改教师信息"
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
        <el-form-item label="工号" prop="tcode">
          <el-input v-model="editForm.tcode"  
            placeholder="请输入工号"
            clearable></el-input>
        </el-form-item>
        <el-form-item label="姓名" prop="tname">
          <el-input
            v-model="editForm.tname"
            placeholder="请输入姓名"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="职位" prop="title">
          <el-input
            v-model="editForm.title"
            placeholder="请输入职位"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="院系" prop="college">
          <el-input
            v-model="editForm.college"
            placeholder="请输入院系"
            clearable
          ></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editTeacher"> 确 定 </el-button>
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
          value: '工号',
          label: '工号',
        },
        {
          value: '院系号',
          label: '院系号',
        },
        {
          value: '职位',
          label: '职位',
        },
      ],
      selected: '工号', // 搜索选择
      input: '', // 搜索输入框
      teacherList: [], // 教师列表
      teacherListShow: [], // 展示的教师列表
      total: 0, // 教师列表总数
      currentPage: 1, // 当前页面
      pageSize: 5, // 每页展示列表数
      collegeList: [], // 院系列表
      positionList: [
        { id: 1, name: '教授' },
        { id: 2, name: '副教授' },
        { id: 3, name: '院长' },
        { id: 4, name: '副院长' },
        { id: 5, name: '讲师' },
        { id: 6, name: '博导' },
      ], // 职位列表
      // 控制添加对话框显示与隐藏
      addDialogVisible: false,
      // 添加教师的表单数据
      addForm: {
        tcode: '',
        tname: '',
        passwords: '',
        title: '',
        college: '',
      },
      // 添加表单的验证规则
      addFormRules: {
        tcode: [
          { required: true, message: '请输入工号', trigger: 'blur' },
          {
            min: 10,
            max: 10,
            message: '工号长度为10个字符',
            trigger: 'blur',
          },
        ],
        tname: [
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
        title: [
          { required: true, message: '请输入职位', trigger: 'blur' },
          {
            min: 2,
            max: 4,
            message: '职位长度为2~4个字符',
            trigger: 'blur',
          },
        ],
        college: [
          { required: true, message: '请输入学院号', trigger: 'blur' },
          {
            min: 2,
            max: 2,
            message: '学院号长度为2个字符',
            trigger: 'blur',
          },
        ],
      },
      // 控制修改对话框显示与隐藏
      editDialogVisible: false,
      // 查询到的教师信息
      editForm: {
        id:'',
        tcode: '',
        tname: '',
        title: '',
        college: '',
      },
      // 修改表单的验证规则
      editFormRules: {
        tname: [
          { required: false, message: '请输入姓名', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '姓名长度在3~10个字符之间',
            trigger: 'blur',
          },
        ],
      },
    }
  },
  created() {
    // this.getCollegeList()
    this.getTeacherList()
  },
  methods: {
    // 获取教师列表
    async getTeacherList(search) {
      var query = {}
      if (search) {
        if (this.selected === '工号') {
          query = { tcode: this.input }
        } else if (this.selected === '院系号') {
          query = { college: this.input }
        } else if (this.selected === '职位') {
          query = { title: this.input }
        }
      }
      if(typeof query.tcode != "undefined"){
        const { data: res } = await this.$http.get('/edustaff/staffteacher/info-tcode/' + query.tcode)
        if (res.code !== 200) return this.$message.error('获取教师列表失败')
        this.teacherList = res.staffTeacher
        this.total = res.total
      }else if(typeof query.college != "undefined"){
        const { data: res } = await this.$http.get('/edustaff/staffteacher/info-college/' + query.college)
        if (res.code !== 200) return this.$message.error('获取教师列表失败')
        this.teacherList = res.staffTeacher
        this.total = res.total
      }else if(typeof query.title != "undefined"){
        const { data: res } = await this.$http.get('/edustaff/staffteacher/info-title/' + query.title)
        if (res.code !== 200) return this.$message.error('获取教师列表失败')
        this.teacherList = res.staffTeacher
        this.total = res.total
      }else{
        const { data: res } = await this.$http.get('/edustaff/staffteacher/info/')
        if (res.code !== 200) return this.$message.error('获取教师列表失败')
        this.teacherList = res.page.list
        this.total = res.total
      }
      this.currentPage = 1
      var start = 0
      var end = start + this.pageSize
      this.teacherListShow = this.teacherList.slice(start, end)
      // console.log(res)
    },
    // 监听对话框关闭事件
    addDialogClosed() {
      // 重置表单
      this.$refs.addFormRef.resetFields()
    },
    // 添加教师
    addTeacher() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        // 校验正确则发起请求
        const { data: res } = await this.$http.post('/edustaff/staffteacher/save', this.addForm)
        if (res.code === 200) this.$message.success('添加成功！')
        else this.$message.error('添加失败！\n')
        this.addDialogVisible = false
        // 重新获取教师列表
        this.getTeacherList()
      })
    },
    // 根据工号查询教师信息
    async showEditDialog(id) {
      // console.log(id)
      this.editForm.id = id
      this.editDialogVisible = true
      // console.log(res)
    },
    // 监听修改对话框的关闭事件
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 修改教师信息
    editTeacher() {
      this.$refs.editFormRef.validate(async (valid) => {
        // console.log(valid)
        if (!valid) return
        // 校验正确则发起请求
        const { data: res } = await this.$http.post('/edustaff/staffteacher/update', {id:this.editForm.id, tcode:this.editForm.tcode, tname: this.editForm.tname, college:this.editForm.college,title:this.editForm.title})
        if (res.code !== 200) return this.$message.error('获取教师列表失败')
        this.$message.success('更新成功！')
        this.editDialogVisible = false
        // 重新获取教师列表
        this.getTeacherList()
      })
    },
    // 删除教师
    deleteTeacher(id) {
      // 弹框提示
      this.$confirm('此操作将永久删除该教师, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      })
        .then(async () => {
          // 确定删除
          const { data: res } = await this.$http.post(
            '/edustaff/staffteacher/delete' , [id]
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
          // 刷新教师列表
          this.getTeacherList()
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
      this.teacherListShow = this.teacherList.slice(start, end)
    },
    // 监听当前页面变化
    handleCurrentChange(currentPage) {
      this.currentPage = currentPage
      let start = (currentPage - 1) * this.pageSize
      let end = start + this.pageSize
      this.teacherListShow = this.teacherList.slice(start, end)
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
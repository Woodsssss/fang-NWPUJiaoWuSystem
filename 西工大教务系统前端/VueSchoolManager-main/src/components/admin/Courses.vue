<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/admin/home' }"
        >首页</el-breadcrumb-item
      >
      <el-breadcrumb-item>信息管理</el-breadcrumb-item>
      <el-breadcrumb-item>课程列表</el-breadcrumb-item>
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
              @click="getCourseList(true)"
            ></el-button>
            <el-button
              slot="append"
              icon="el-icon-refresh-right"
              @click="getCourseList(false)"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">
            添加课程
          </el-button>
        </el-col>
      </el-row>
      <!-- 列表区域 -->
      <el-table :data="courseListShow" border stripe>
        <!-- 自定义索引 -->
        <el-table-column type="index"> </el-table-column>
        <el-table-column prop="course.ccode" label="课程号"></el-table-column>
        <el-table-column prop="course.cname" label="课程名"></el-table-column>
        <el-table-column prop="course.credit" label="学分"></el-table-column>
        <el-table-column prop="course.place" label="上课地点"></el-table-column>
        <el-table-column prop="course.descrip" label="课程简介"></el-table-column>
        <el-table-column prop="teacherList" label="任课教师"></el-table-column>
        <el-table-column prop="teacherListCode" label="任课教师工号"></el-table-column>
        <el-table-column prop="course.date" label="上课日期"></el-table-column>
        <el-table-column prop="course.time" label="上课时间"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.course.id)"
            >
            </el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="deleteCourse(scope.row.course.id)"
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
    <!-- 添加课程对话框 -->
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
        <el-form-item label="课程号" prop="ccode">
          <el-input
            v-model="addForm.ccode"
            placeholder="请输入课程号"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="课程名" prop="cname">
          <el-input
            v-model="addForm.cname"
            placeholder="请输入课程名"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="学分" prop="credit">
          <el-input
            v-model.number="addForm.credit"
            placeholder="请输入学分"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="上课地点" prop="place">
          <el-input
            v-model="addForm.place"
            placeholder="请输入上课地点"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="课程简介" prop="descrip">
          <el-input
            v-model="addForm.descrip"
            placeholder="请输入课程简介"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="教师工号" prop="tcode">
          <el-input
            v-model="addForm.tcode"
            placeholder="请输入任课教师工号"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="日期" prop="date">
          <el-input
            v-model="addForm.date"
            placeholder="请输入上课日期"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="时间" prop="time">
          <el-input
            v-model="addForm.time"
            placeholder="请输入上课时间"
            clearable
          ></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCourse"> 确 定 </el-button>
      </span>
    </el-dialog>
    <!-- 修改课程对话框 -->
    <el-dialog
      title="修改课程信息"
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
        <el-form-item label="课程号" prop="ccode">
          <el-input v-model="editForm.ccode" placeholder="请输入课程号"
            clearable></el-input>
        </el-form-item>
        <el-form-item label="课程名" prop="cname">
          <el-input
            v-model="editForm.cname"
            placeholder="请输入课程名"
            clearable
          ></el-input>
        </el-form-item>
         <el-form-item label="学分" prop="credit">
          <el-input
            v-model.number="editForm.credit"
            placeholder="请输入学分"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="上课地点" prop="place">
          <el-input
            v-model="editForm.place"
            placeholder="请输入上课地点"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="课程简介" prop="descrip">
          <el-input
            v-model="editForm.descrip"
            placeholder="请输入课程简介"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="教师工号" prop="tcode">
          <el-input
            v-model="editForm.tcode"
            placeholder="请输入任课教师工号"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="日期" prop="date">
          <el-input
            v-model="editForm.date"
            placeholder="请输入上课日期"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="时间" prop="time">
          <el-input
            v-model="editForm.time"
            placeholder="请输入上课时间"
            clearable
          ></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCourse"> 确 定 </el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>

import datePicker from 'vuejs-datepicker';
export default {
  components: {
    datePicker
  },
  data() {
    return {
      value1:'',
      options: [
        {
          value: '课程号',
          label: '课程号',
        },
        {
          value: '课程名',
          label: '课程名',
        },
        {
          value: '学分',
          label: '学分',
        },
      ],
      selected: '课程号', // 搜索选择
      input: '', // 搜索输入框
      courseList: [], // 课程列表
      courseListShow: [], // 展示的课程列表
      teacherList:[],//教师列表
      teacherListShow:[],//展示的教师列表
      total: 0, // 课程列表总数
      currentPage: 1, // 当前页面
      pageSize: 5, // 每页展示列表数
      // 控制添加对话框显示与隐藏
      addDialogVisible: false,
      // 添加课程的表单数据
      addForm: {
        ccode: '',
        cname: '',
        credit: 0,
        place:'',
        descrip:'',
        tcode:'',
        date:null,
        time:'',
      },
      // 添加表单的验证规则
      addFormRules: {
        ccode: [
          { required: true, message: '请输入课程号', trigger: 'blur' },
          {
            min: 2,
            max: 6,
            message: '课程号长度在2~6个数字之间',
            trigger: 'blur',
          },
        ],
        cname: [
          { required: true, message: '请输入课程名', trigger: 'blur' },
          {
            min: 3,
            max: 20,
            message: '课程名长度在3~20个字符之间',
            trigger: 'blur',
          },
        ],
        credit: [
          { required: true, message: '请输入学分', trigger: 'blur' },
          {
            type: 'number',
            message: '学分为数字',
            trigger: 'blur',
          },
        ],
        place:[
          { required: true, message: '请输入上课地点', trigger: 'blur' },
          {
            min: 1,
            max: 10,
            message: '课程地点长度在1~10个字符之间',
            trigger: 'blur',
          },
        ],
        descrip:[
          { required: true, message: '请输入课程简介', trigger: 'blur' },
          {
            min: 1,
            max: 20,
            message: '课程简介长度在1~20个字符之间',
            trigger: 'blur',
          },
        ],
        tcode:[
          { required: true, message: '请输入教师工号', trigger: 'blur' },
          {
            min: 1,
            max: 10,
            message: '教师工号长度为1-10个字符',
            trigger: 'blur',
          },
        ],
        date:[
          { required: true, message: '请输入上课日期', trigger: 'blur' },
          {
            min: 1,
            max: 15,
            message: '上课日期长度为10个字符',
            trigger: 'blur',
          },
        ],
        time:[
          { required: true, message: '请输入上课时间', trigger: 'blur' },
          {
            min: 1,
            max: 15,
            message: '上课时间长度为15个字符',
            trigger: 'blur',
          },
        ]
      },
      // 控制修改对话框显示与隐藏
      editDialogVisible: false,
      // 查询到的课程信息
      editForm: {
        id:'',
        ccode: '',
        cname: '',
        credit: 0,
        place:'',
        descrip:'',
        tcode:'',
        date:'',
        time:'',
      },
      // 修改表单的验证规则
      editFormRules: {
        ccode: [
          { required: true, message: '请输入课程号', trigger: 'blur' },
          {
            min: 2,
            max: 6,
            message: '课程号长度在2~6个数字之间',
            trigger: 'blur',
          },
        ],
        cname: [
          { required: true, message: '请输入课程名', trigger: 'blur' },
          {
            min: 3,
            max: 20,
            message: '课程名长度在3~20个字符之间',
            trigger: 'blur',
          },
        ],
        credit: [
          { required: true, message: '请输入学分', trigger: 'blur' },
          {
            type: 'number',
            message: '学分为数字',
            trigger: 'blur',
          },
        ],
        place:[
          { required: true, message: '请输入上课地点', trigger: 'blur' },
          {
            min: 1,
            max: 10,
            message: '课程地点长度在1~10个字符之间',
            trigger: 'blur',
          },
        ],
        descrip:[
          { required: true, message: '请输入课程简介', trigger: 'blur' },
          {
            min: 1,
            max: 20,
            message: '课程简介长度在1~20个字符之间',
            trigger: 'blur',
          },
        ],
        tcode:[
          { required: true, message: '请输入教师工号', trigger: 'blur' },
          {
            min: 10,
            max: 10,
            message: '教师工号长度为10个字符',
            trigger: 'blur',
          },
        ],
        date:[
          { required: true, message: '请输入上课日期', trigger: 'blur' },
          {
            min: 1,
            max: 10,
            message: '上课日期长度为10个字符',
            trigger: 'blur',
          },
        ],
        time:[
        { required: true, message: '请输入上课时间', trigger: 'blur' },
          {
            min: 1,
            max: 15,
            message: '上课时间长度为15个字符',
            trigger: 'blur',
          },
        ]
      },
    }
  },
  created() {
    this.getCourseList()
  },
  methods: {
    // 获取课程列表
    async getCourseList(search) {
      var query = {}
      if (search) {
        if (this.selected === '课程号') {
          query = { ccode: this.input }
        } else if (this.selected === '课程名') {
          query = { cname: this.input }
        } else if (this.selected === '学分') {
          query = { credit: this.input }
        }
      }
      if(typeof query.ccode != "undefined"){
        const { data: res } = await this.$http.post('/eduacademic/academiccourse/info-ccode/' + query.ccode)
        if (res.code !== 200) return this.$message.error('获取课程列表失败')
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
      }else if(typeof query.cname != "undefined"){
        const { data: res } = await this.$http.post('/eduacademic/academiccourse/info-cname/' + query.cname)
        if (res.code !== 200) return this.$message.error('获取课程列表失败')
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
      }else if(typeof query.credit != "undefined"){
        const { data: res } = await this.$http.post('/eduacademic/academiccourse/info-credit/' + query.credit)
        if (res.code !== 200) return this.$message.error('获取课程列表失败')
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
      else{
        const { data: res } = await this.$http.post('/eduacademic/academiccourse/info/')
        if (res.code !== 200) return this.$message.error('获取课程列表失败')
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
    // 监听对话框关闭事件
    addDialogClosed() {
      // 重置表单
      this.$refs.addFormRef.resetFields()
    },
    // 添加课程
    addCourse() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        // 校验正确则发起请求
        const { data: res } = await this.$http.post('/eduacademic/academiccourse/save', this.addForm)
        if (res.code === 200) this.$message.success('添加成功！')
        else this.$message.error('添加失败！\n')
        this.addDialogVisible = false
        // 重新获取课程列表
        this.getCourseList()
      })
    },
    // 根据课程号课程信息
    async showEditDialog(id) {
      console.log(id)
      this.editForm.id = id
      this.editDialogVisible = true
      //   console.log(res)
    },
    // 监听修改对话框的关闭事件
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 修改课程信息
    editCourse() {
      this.$refs.editFormRef.validate(async (valid) => {
        // console.log(valid)
        if (!valid) return
        // 校验正确则发起请求
        const { data: res } = await this.$http.post('/eduacademic/academiccourse/update', 
        {id:this.editForm.id, ccode:this.editForm.ccode, cname: this.editForm.cname,credit:this.editForm.credit,
           place:this.editForm.place,descrip:this.editForm.descrip,tcode:this.editForm.tcode,date:this.editForm.date,time:this.editForm.time})
        if (res.code !== 200)
          this.$message.error('修改失败！\n' + res.detail[0])
        this.$message.success('更新成功！')
        this.editDialogVisible = false
        // 重新获取课程列表
        this.getCourseList()
      })
    },
    // 删除课程
    deleteCourse(id) {
      // 弹框提示
      this.$confirm('此操作将永久删除该课程, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      })
        .then(async () => {
          // 确定删除
          const { data: res } = await this.$http.post(
            '/eduacademic/academiccourse/delete' , [id]
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
          // 刷新课程列表
          this.getCourseList()
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
.select {
  width: 120px;
}
.pagination {
  margin-top: 15px;
}
</style>
<template>
  <div>
    <!-- 导航-->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/teacher/welcome' }"
        >首页</el-breadcrumb-item
      >
      <el-breadcrumb-item>教学相关</el-breadcrumb-item>
      <el-breadcrumb-item>课程教学大纲</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 搜索与添加区域 v-model提供数据绑定功能-->
      <el-row :gutter="20">
        <el-col :span="16">
          <el-input
            :placeholder="'请输入' + selected"
            prefix-icon="el-icon-search"
            v-model="input"
            clearable
            @clear="getcourseName"
          >
            <el-select
              v-model="selected"
              slot="prepend"
              placeholder="请选择"
              class="select"
            >
              <el-option
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
            <el-button @click="getcourseName" slot="append"> 搜索 </el-button>
          </el-input>
        </el-col>
      </el-row>
      <!--课程列表区域-->
      <el-table :data="courselistShow" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column
          label="课程号"
          prop="course.ccode"
          align="center"
        ></el-table-column>
        <el-table-column
          label="课程名称"
          prop="course.cname"
          align="center"
        ></el-table-column>
        <el-table-column
          label="教师号"
          prop="teacherListCode"
          align="center"
        ></el-table-column>
        <el-table-column
          label="教师名"
          prop="teacherList"
          align="center"
        ></el-table-column>
        <!--通过作用域插槽-->
        <el-table-column label="编辑" align="center">
          <template slot-scope="scope">
            <!-- 上传按钮 -->
            <el-upload
              action="http://101.37.83.166:7989/edu-web/eduacademic/academiccourse/upload"
              :before-upload="beforeAvatarUpload"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              :before-remove="beforeRemove"
              multiple
              :limit="3"
              :on-exceed="handleExceed"
            >
              <el-button slot="trigger" size="mini" type="text" @click="getCcode(scope.row.course.ccode)"
                >点击此处上传教学大纲</el-button
              >
            </el-upload>
            <!-- <form:form method="POST" action="/spring-mvc-xml/uploadFile" enctype="multipart/form-data">
            <table>
            <tr>
            <td><form:label path="file">Select a file to upload</form:label></td>
            <td><input type="file" name="file" /></td>  
            </tr>
            <tr>
            <td><input type="text" name="ccode" value= scope.row.ccode /></td>
            </tr>
            <tr>
            <td><input type="submit" value="Submit" /></td>
            </tr>
            </table>
            </form:form> -->
          </template>
        </el-table-column>
        <el-table-column
          label="教学大纲"
          prop="course.overviewPath"
          align="center"
        >
        <template slot-scope="scope">
          <span style="color:red;" v-if="scope.row.course.overviewPath==undefined">未上传!</span>
          <span v-else>{{ scope.row.course.overviewPath }}</span>
        </template>
        </el-table-column>
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
import axios from 'axios';

export default {

  data() {
    return {
      ccode:'',
      id: window.sessionStorage.getItem('user'),
      options: [
        {
          value: '课程号',
          label: '课程号',
        },
        {
          value: '教师号',
          label: '教师号',
        },
      ],
      selected: '', // 搜索选择
      input: '',
      courseList: [],
      total: 0, // 院系列表总数
      currentPage: 1, // 当前页面
      pageSize: 3, // 每页展示列表数
      courselistShow: [],
      formData: "",
    }
  },
  created() {
    this.getcourseName()
  },
  methods: {
    beforeAvatarUpload(file){
      let params = new FormData()
      params.append("ccode",this.ccode);
      params.append("file",file);
      axios({
        method:'post',
        url:'http://101.37.83.166:7989/edu-web/eduacademic/academiccourse/upload',
        data:params,
        headers:{
          'content-type':'multipart/form-data'
        }
      }).then(res=>{
        console.log(res)
      })
      .catch(err=>{
        console.log(err)
      })
    },
    getCcode(ccode){
      this.ccode = ccode
      console.log(ccode)
    },
    async getcourseName(select) {
      var query = {}
      if (select) {
        if (this.selected === '课程号') {
          query = { ccode: this.input }
        }
        else if (this.selected === '教师号') query = { tcode: this.input }} 
        else query = {}
      if(typeof query.ccode!="undefined"){
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
      }
      else if(typeof query.tcode!="undefined"){
        const { data: res } = await this.$http.post('/eduacademic/academiccourse/info-tcode/' + query.tcode)
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
      }else{
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
        // console.log(res)
        this.total = res.total
      }
      this.currentPage = 1
      var start = 0
      var end = start + this.pageSize
      this.courselistShow = this.courseList.slice(start, end)
    },
    //教学大纲上传
    handleRemove(file, fileList) {
      console.log(file, fileList)
    },
    handlePreview(file) {
      console.log("preview"+file)
      if (window.navigator.msSaveOrOpenBlob) {
        navigator.msSaveBlob(file.raw, file.name)
      } else {
        var link = document.createElement('a')
        link.download = file.name
        link.href = window.URL.createObjectURL(file.raw)
        link.click()
        window.URL.revokeObjectURL(link.href)
      }
    },
    handleExceed(files, fileList) {
      this.$message.warning(
        `当前限制选择 3 个文件，本次选择了 ${files.length} 个文件，共选择了 ${
          files.length + fileList.length
        } 个文件`
      )
    },
    beforeRemove(file, fileList) {
      return this.$confirm(`确定移除 ${file.name}？`)
    },
    handleSizeChange(pageSize) {
      console.log(pageSize)
      this.pageSize = pageSize
      let start = 0
      let end = start + pageSize
      this.courselistShow = this.courseList.slice(start, end)
    },
    // 监听当前页面变化
    handleCurrentChange(currentPage) {
      this.currentPage = currentPage
      let start = (currentPage - 1) * this.pageSize
      let end = start + this.pageSize
      this.courselistShow = this.courseList.slice(start, end)
    },
  },
}
</script>

<style lang="less" scoped>
.demo-input-label {
  display: inline-block;
  width: 130px;
}
.select {
  width: 120px;
}
</style>
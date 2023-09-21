<template>
  <div>
    <!-- 导航-->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/teacher/welcome' }"
        >首页</el-breadcrumb-item
      >
      <el-breadcrumb-item>教学相关</el-breadcrumb-item>
      <el-breadcrumb-item>学生名单</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 搜索与添加区域 v-model提供数据绑定功能-->
      <el-row :gutter="20">
        <el-col :span="7">
          <el-input
            placeholder="请输入课程号"
            prefix-icon="el-icon-search"
            v-model="input1"
            clearable
            @clear="getstudent"
          >
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button @click="getstudent" type="primary"> 搜索 </el-button>
        </el-col>
        <el-col :span="4">
          <el-button type="text" @click="toImg">转至打印页</el-button>
        </el-col>
      </el-row>
      <!--课程列表区域-->
      <div id="printMe" ref="printContent">
        <!-- <el-table><el-table-column :label="this.input2 +'学期'+this.studentlist[0].open.course.course_name +'课程学生名单'" align="center">
          </el-table-column></el-table> -->
        <el-table :data="studentlistShow" border stripe>
          <el-table-column type="index" align="center"></el-table-column>
          <el-table-column
            label="学号"
            prop="student.scode"
            width="100px"
            align="center"
          ></el-table-column>
          <el-table-column
            label="姓名"
            prop="student.sname"
            width="110px"
            align="center"
          ></el-table-column>
          <el-table-column
            label="课程号"
            prop="course.ccode"
            width="90px"
            align="center"
          ></el-table-column>
          <el-table-column
            label="课程名"
            prop="course.cname"
            width="150px"
            align="center"
          ></el-table-column>
          <el-table-column
            label="上课时间"
            prop="course.time"
            width="120px"
            align="center"
          ></el-table-column>
          <el-table-column label="出勤" align="center">
            <template slot-scope="scope">
              <el-checkbox-group v-model="checklist">
                <el-checkbox label="1"></el-checkbox>
                <el-checkbox label="2"></el-checkbox>
                <el-checkbox label="3"></el-checkbox>
                <el-checkbox label="4"></el-checkbox>
                <el-checkbox label="5"></el-checkbox>
                <el-checkbox label="6"></el-checkbox>
                <el-checkbox label="7"></el-checkbox>
                <el-checkbox label="8"></el-checkbox>
                <el-checkbox label="9"></el-checkbox>
                <el-checkbox label="10"></el-checkbox>
              </el-checkbox-group>
            </template>
          </el-table-column>
        </el-table>
      </div>

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
import html2canvas from 'html2canvas'
import printJS from 'print-js'
export default {
  data() {
    return {
      id:window.sessionStorage.getItem('user'),
      input1: '',
      studentlist: [],
      studentlistShow: [],
      total: 0, // 列表总数
      currentPage: 1, // 当前页面
      pageSize: 3, // 每页展示列表数
      checklist: [],
    }
  },
  created() {
    this.getstudent()
  },
  methods: {
    // checkBoxChange(val) { // 控制 单选
    //     if(this.checklist.indexOf(val) > -1){
    //       this.checklist = [];
    //       this.checklist.push(val);
    //     }
    //   },
    async getstudent() {
      var query = {}
      if (this.input1) {
        query.ccode = this.input1
      }else{
        query = {}
      }
      if(typeof query.ccode!="undefined"){
        const { data: res } = await this.$http.post('eduacademic/academicselcourse/info-ccode'+ query.ccode)
        if (res.code !== 200) return this.$message.error('获取学生列表失败')
        this.studentlist = res.academicSelCourse
        this.total = res.total
      }else{
        const { data: res } = await this.$http.post('eduacademic/academicselcourse/info', {id:this.id})
        if (res.code !== 200) return this.$message.error('获取学生列表失败')
        this.studentlist = res.academicSelCourse
        this.total = res.total
      }
      this.currentPage = 1
      var start = 0
      var end = start + this.pageSize
      this.studentlistShow = this.studentlist.slice(start, end)
      //   console.log(res.data.Scores[0].open.course.course_name)
    },

    handleSizeChange(pageSize) {
      console.log(pageSize)
      this.pageSize = pageSize
      let start = 0
      let end = start + pageSize
      this.studentlistShow = this.studentlist.slice(start, end)
    },
    // 监听当前页面变化
    handleCurrentChange(currentPage) {
      this.currentPage = currentPage
      let start = (currentPage - 1) * this.pageSize
      let end = start + this.pageSize
      this.studentlistShow = this.studentlist.slice(start, end)
    },
    //转至打印页
    toImg() {
      html2canvas(this.$refs.printContent, {
        backgroundColor: null,
        useCORS: true,
        windowHeight: document.body.scrollHeight,
      }).then((canvas) => {
        // let url = canvas.toDataURL('image/jpeg', 1.0)
        const url = canvas.toDataURL()
        this.img = url
        printJS({
          printable: url,
          type: 'image',
          documentTitle: '打印图片',
        })
        // console.log(url)
      })
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
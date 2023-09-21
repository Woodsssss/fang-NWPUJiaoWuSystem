<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/admin/home' }"
        >首页</el-breadcrumb-item
      >
      <el-breadcrumb-item>信息管理</el-breadcrumb-item>
      <el-breadcrumb-item>院系列表</el-breadcrumb-item>
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
              @click="getCollegeList(true)"
            ></el-button>
            <el-button
              slot="append"
              icon="el-icon-refresh-right"
              @click="getCollegeList(false)"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true"
            >添加院系</el-button
          >
        </el-col>
      </el-row>
      <!-- 列表区域 -->
      <el-table :data="collegeListShow" border stripe>
        <!-- 自定义索引 -->
        <el-table-column type="index"> </el-table-column>
        <el-table-column prop="collegecode" label="院系号"></el-table-column>
        <el-table-column prop="collegename" label="院系名"></el-table-column>
        <el-table-column prop="place" label="地址"></el-table-column>
        <el-table-column prop="decrip" label="院系介绍"></el-table-column>
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
              @click="deleteCollege(scope.row.id)"
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
    <!-- 添加院系对话框 -->
    <el-dialog
      title="添加院系"
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
        <el-form-item label="院系号" prop="collegecode">
          <el-input
            v-model="addForm.collegecode"
            placeholder="请输入院系号"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="院系名" prop="collegename">
          <el-input
            v-model="addForm.collegename"
            placeholder="请输入院系名"
            clearable
          ></el-input>    
        </el-form-item>
        <el-form-item label="地址" prop="place">
          <el-input
            v-model="addForm.place"
            placeholder="请输入院系地址"
            clearable
          ></el-input>    
        </el-form-item>
        <el-form-item label="介绍" prop="decrip">
          <el-input
            v-model="addForm.decrip"
            placeholder="请输入院系介绍"
            clearable
          ></el-input>    
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCollege"> 确 定 </el-button>
      </span>
    </el-dialog>
    <!-- 修改院系对话框 -->
    <el-dialog
      title="修改院系信息"
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
        <el-form-item label="院系号" prop="collegecode">
          <el-input v-model="editForm.collegecode" placeholder="请输入院系号"
            clearable></el-input>
        </el-form-item>
        <el-form-item label="院系名" prop="collegename">
          <el-input
            v-model="editForm.collegename"
            placeholder="请输入院系名"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="院系地址" prop="place">
          <el-input
            v-model="editForm.place"
            placeholder="请输入院系地址"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="院系介绍" prop="decrip">
          <el-input
            v-model="editForm.decrip"
            placeholder="请输入院系介绍"
            clearable
          ></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCollege"> 确 定 </el-button>
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
          value: '院系号',
          label: '院系号',
        },
        {
          value: '院系名',
          label: '院系名',
        },
      ],
      selected: '院系号', // 搜索选择
      input: '', // 搜索输入框
      collegeList: [], // 院系列表
      collegeListShow: [], // 展示的院系列表
      total: 0, // 院系列表总数
      currentPage: 1, // 当前页面
      pageSize: 5, // 每页展示列表数
      // 控制添加对话框显示与隐藏
      addDialogVisible: false,
      // 添加院系的表单数据
      addForm: {
        collegecode: '',
        collegename: '',
        place:'',
        decrip:'',
      },
      // 添加表单的验证规则
      addFormRules: {
        collegecode: [
          { required: true, message: '请输入院系号', trigger: 'blur' },
          {
            min: 1,
            max: 2,
            message: '院系号长度在1~2个之间',
            trigger: 'blur',
          },
        ],
        collegename: [
          { required: true, message: '请输入院系名', trigger: 'blur' },
          {
            min: 1,
            max: 20,
            message: '院系名长度在1~10个字符之间',
            trigger: 'blur',
          },
        ],
        place: [
          { required: true, message: '请输入院系地址', trigger: 'blur' },
          {
            min: 1,
            max: 20,
            message: '院系地址长度在1~20个字符之间',
            trigger: 'blur',
          },
        ],
        decrip: [
          { required: true, message: '请输入院系介绍', trigger: 'blur' },
          {
            min: 1,
            max: 20,
            message: '院系介绍长度在1~20个字符之间',
            trigger: 'blur',
          },
        ],
      },
      // 控制修改对话框显示与隐藏
      editDialogVisible: false,
      // 查询到的院系信息
      editForm: {
        id:'',
        collegecode:'',
        collegename: '',
        place:'',
        decrip:''
      },
      // 修改表单的验证规则
      editFormRules: {
        collegename: [
          { required: false, message: '请输入院系名', trigger: 'blur' },
          {
            min: 1,
            max: 20,
            message: '院系名长度在1~20个字符之间',
            trigger: 'blur',
          },
        ],
      },
    }
  },
  created() {
    this.getCollegeList()
  },
  methods: {
    // 获取院系列表
    async getCollegeList(search) {
      var query = {}
      if (search) {
        if (this.selected === '院系号') {
          query = { collegecode: this.input }
        } else if (this.selected === '院系名') {
          query = { collegename: this.input }
        }
      }
      if(typeof query.collegecode != "undefined"){
        const { data: res } = await this.$http.get('/edudaily/dailycollege/info-collegeCode/' + query.collegecode)
        if (res.code !== 200) return this.$message.error('获取学院列表失败')
        this.collegeList = res.dailyCollege
        this.total = res.total
      }else if(typeof query.collegename != "undefined"){
        const { data: res } = await this.$http.get('/edudaily/dailycollege/info-collegeName/' + query.collegename)
        if (res.code !== 200) return this.$message.error('获取学院列表失败')
        this.collegeList = res.dailyCollege
        this.total = res.total
      }else{
        const { data: res } = await this.$http.get('/edudaily/dailycollege/info/')
        if (res.code !== 200) return this.$message.error('获取学院列表失败')
        this.collegeList = res.page.list
        this.total = res.total
      }

      this.currentPage = 1
      var start = 0
      var end = start + this.pageSize
      this.collegeListShow = this.collegeList.slice(start, end)
    //   console.log(res)
    },
    // 监听对话框关闭事件
    addDialogClosed() {
      // 重置表单
      this.$refs.addFormRef.resetFields()
    },
    // 添加院系
    addCollege() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        // 校验正确则发起请求
        const { data: res } = await this.$http.post('/edudaily/dailycollege/save', this.addForm)
        if (res.code === 200) this.$message.success('添加成功！')
        else this.$message.error('添加失败！\n' + res.detail[0])
        this.addDialogVisible = false
        // 重新获取院系列表
        this.getCollegeList()
      })
    },
    // 根据院系号院系信息
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
    // 修改院系信息
    editCollege() {
      this.$refs.editFormRef.validate(async (valid) => {
        // console.log(valid)
        if (!valid) return
        // 校验正确则发起请求
        const { data: res } = await this.$http.post('/edudaily/dailycollege/update', {id:this.editForm.id, collegecode:this.editForm.collegecode, collegename: this.editForm.collegename, place:this.editForm.place,decrip:this.editForm.decrip})
        if (res.code !== 200)
          this.$message.error('修改失败！\n')
        this.$message.success('更新成功！')
        this.editDialogVisible = false
        // 重新获取院系列表
        this.getCollegeList()
      })
    },
    // 删除院系
    deleteCollege(id) {
      // 弹框提示
      this.$confirm('此操作将永久删除该院系, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      })
        .then(async () => {
          // 确定删除
          const { data: res } = await this.$http.post(
            '/edudaily/dailycollege/delete' , [id]
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
          // 刷新院系列表
          this.getCollegeList()
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
      this.collegeListShow = this.collegeList.slice(start, end)
    },
    // 监听当前页面变化
    handleCurrentChange(currentPage) {
      this.currentPage = currentPage
      let start = (currentPage - 1) * this.pageSize
      let end = start + this.pageSize
      this.collegeListShow = this.collegeList.slice(start, end)
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
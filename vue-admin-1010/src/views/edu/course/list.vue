<template>
  <div class="app-container">
    课程列表

    <!--查询表单-->
    <el-form :inline="true" class="demo-form-inline">
      <el-form-item>
        <el-input v-model="courseQuery.title" placeholder="课程名称"/>
      </el-form-item>

      <el-form-item>
        <el-select v-model="courseQuery.status" placeholder="课程状态">
          <el-option :value="1" label="已发布"/>
          <el-option :value="2" label="未发布"/>
        </el-select>
      </el-form-item>
      <!-- 讲师 -->
      <el-form-item>
            <el-select
            v-model="courseQuery.teacherId"
            placeholder="请选择讲师">
            <el-option
                v-for="teacher in teacherList"
                :key="teacher.id"
                :label="teacher.name"
                :value="teacher.id"/>
            </el-select>
      </el-form-item>
      <!-- 一级分类 -->
      <el-form-item label="课程类别">
        <el-select
            v-model="courseQuery.subjectParentId"
            placeholder="请选择"
            @change="subjectLevelOneChanged">
            <el-option
                v-for="subject in subjectNestedList"
                :key="subject.id"
                :label="subject.title"
                :value="subject.id"/>
        </el-select>

        <!-- 二级分类 -->
        <el-select v-model="courseQuery.subjectId" placeholder="请选择">
            <el-option
                v-for="subject in subSubjectList"
                :key="subject.id"
                :label="subject.title"
                :value="subject.id"/>
            </el-select>
      </el-form-item>

    

      <el-button type="primary" icon="el-icon-search" @click="getList()">查询</el-button>
      <el-button type="default" @click="resetData()">清空</el-button>
    </el-form>

    <!-- 表格 -->
    <el-table
        :data="list"
        element-loading-text="数据加载中"
        border
        fit
        highlight-current-row
        row-class-name="myClassList">

        <el-table-column
            label="序号"
            width="70"
            align="center">
            <template slot-scope="scope">
            {{ (page - 1) * limit + scope.$index + 1 }}
            </template>
        </el-table-column>

        <el-table-column label="课程信息" width="470" align="center">
            <template slot-scope="scope">
            <div class="info">
                <div class="pic">
                <img :src="scope.row.cover" alt="scope.row.title" width="150px">
                </div>
                <div class="title">
                <a href="">{{ scope.row.title }}</a>
                <p>{{ scope.row.lessonNum }}课时</p>
                </div>
            </div>

            </template>
        </el-table-column>

        <el-table-column label="创建时间" align="center">
            <template slot-scope="scope">
            {{ scope.row.gmtCreate.substr(0, 10) }}
            </template>
        </el-table-column>
        <el-table-column label="发布时间" align="center">
            <template slot-scope="scope">
            {{ scope.row.gmtModified.substr(0, 10) }}
            </template>
        </el-table-column>
        <el-table-column label="价格" width="100" align="center" >
            <template slot-scope="scope">
            {{ Number(scope.row.price) === 0 ? '免费' :
            '¥' + scope.row.price.toFixed(2) }}
            </template>
        </el-table-column>
        <el-table-column prop="buyCount" label="付费学员" width="100" align="center" >
            <template slot-scope="scope">
            {{ scope.row.buyCount }}人
            </template>
        </el-table-column>

        <el-table-column label="操作" width="150" align="center">
            <template slot-scope="scope">
            <router-link :to="'/course/info/'+scope.row.id">
                <el-button type="text" size="mini" icon="el-icon-edit">编辑课程信息</el-button>
            </router-link>
            <router-link :to="'/course/chapter/'+scope.row.id">
                <el-button type="text" size="mini" icon="el-icon-edit">编辑课程大纲</el-button>
            </router-link>
            <el-button type="text" size="mini" icon="el-icon-delete"  @click="removeDataById(scope.row.id)">删除</el-button>
            </template>
        </el-table-column>
        </el-table>

        <!-- 分页 -->
        <el-pagination
        :current-page="page"
        :page-size="limit"
        :total="total"
        style="padding: 30px 0; text-align: center;"
        layout="total, prev, pager, next, jumper"
        @current-change="getList"
    />

  </div>
</template>
<script>
//引入调用teacher.js文件
import course from '@/api/edu/course'
import teacher from '@/api/edu/teacher'
import subject from '@/api/edu/subject'
export default {
    //写核心代码位置
    // data:{
    // },
    data() { //定义变量和初始值
        return {
          list:null,//查询之后接口返回集合
          page:1,//当前页
          limit:10,//每页记录数
          total:0,//总记录数
          courseQuery:{ //条件封装对象
            title: '',
            status: '',
            teacherId: '',
            subjectParentId: '',
            subjectId: ''
          }, 
          teacherList:{}, //讲师列表
          subjectNestedList:[],
          subSubjectList:[]

        }
    },
    created() { //页面渲染之前执行，一般调用methods定义的方法
        //调用
        this.getList() 
        this.getTeacherList()
        this.getSubjectList()
    },
    methods:{  //创建具体的方法，调用teacher.js定义的方法
        //讲师列表的方法
        getList(page=1) {
            this.page = page
            course.getCourseListPage(this.page,this.limit,this.courseQuery)
                .then(response =>{//请求成功
                    //response接口返回的数据
                    //console.log(response)
                    this.list = response.data.rows
                    this.total = response.data.total
                    console.log(this.list)   
                    console.log(this.total)
                }) 
        },
        resetData() {//清空的方法
            //表单输入项数据清空
            this.courseQuery = {}
            this.subSubjectList = []
            //查询所有讲师数据
            this.getList()
        },
        //删除讲师的方法
        removeDataById(id) {
            this.$confirm('此操作将永久删除课程记录, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {  //点击确定，执行then方法
                //调用删除的方法
                course.deleteCourseId(id)
                    .then(response =>{//删除成功
                    //提示信息
                    this.$message({
                        type: 'success',
                        message: '删除成功!'
                    });
                    //回到列表页面
                    this.getList()
                })
            }) //点击取消，执行catch方法
        },
        // 查询所有讲师
        getTeacherList(){
            teacher.getAllTeacher()
                .then(response =>{
                    this.teacherList = response.data.items
                })
        },
        // 得到所有的一级分类
        getSubjectList(){
            subject.getSubjectList()
                .then(response =>{
                    this.subjectNestedList = response.data.list
                })
        },
        // 得到二级分类
        subjectLevelOneChanged(value){
            for(let i = 0; i < this.subjectNestedList.length; i++){
                if(this.subjectNestedList[i].id === value){
                    this.subSubjectList = this.subjectNestedList[i].children
                    this.courseQuery.subjectId = '' // 写这句话的时候courseQuery要提前写上这条属性，不然选上不显示，点击别处才显示
                }
            }
        }
        
 
    }
}
</script>
<style scoped>
.myClassList .info {
    width: 450px;
    overflow: hidden;
}
.myClassList .info .pic {
    width: 150px;
    height: 90px;
    overflow: hidden;
    float: left;
}
.myClassList .info .pic a {
    display: block;
    width: 100%;
    height: 100%;
    margin: 0;
    padding: 0;
}
.myClassList .info .pic img {
    display: block;
    width: 100%;
}
.myClassList td .info .title {
    width: 280px;
    float: right;
    height: 90px;
}
.myClassList td .info .title a {
    display: block;
    height: 48px;
    line-height: 24px;
    overflow: hidden;
    color: #00baf2;
    margin-bottom: 12px;
}
.myClassList td .info .title p {
    line-height: 20px;
    margin-top: 5px;
    color: #818181;
}
</style>
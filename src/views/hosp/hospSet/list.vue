<template>
  <div class="app-container">
    <el-form :inline="true" class="demo-form-inline">
      <el-form-item>
        <el-input v-model="searchObj.hosname" placeholder="医院名称"/>
      </el-form-item>
      <el-form-item>
        <el-input v-model="searchObj.hoscode" placeholder="医院编号"/>
      </el-form-item>
      <el-button type="primary" icon="el-icon-search" @click="fetchData()">查询</el-button>
    </el-form>

    <!-- banner列表 -->
    <!-- 工具条 -->
    <div>
      <el-button type="danger" size="mini" @click="removeRows()">批量删除</el-button>
    </div>

    <el-table
      :data="list"
      stripe
      style="width: 100%"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" stripe width="100%"/>
      <!--<el-table-column type="index" width="50"/>-->
      <el-table-column prop="hosname" label="医院名称"/>
      <el-table-column prop="hoscode" label="医院编号"/>
      <el-table-column prop="apiUrl" label="api基础路径" width="200"/>
      <el-table-column prop="contactsName" label="联系人姓名"/>
      <el-table-column prop="contactsPhone" label="联系人手机"/>
      <el-table-column label="状态" width="80">
        <template slot-scope="scope">
          {{ scope.row.status === 1 ? '可用' : '不可用' }}
        </template>
      </el-table-column>
      <!--<el-table-column label="操作" width="280" align="center">-->
      <!--  <template slot-scope="scope">-->
      <!--    <el-button type="danger" size="mini"-->
      <!--               icon="el-icon-delete" @click="removeDataById(scope.row.id)"-->
      <!--    ></el-button>-->
      <!--  </template>-->
      <!--</el-table-column>-->


      <el-table-column label="操作" width="280" align="center">

        <template slot-scope="scope">

          <router-link :to="'/hospSet/edit/'+scope.row.id">
              <el-button type="primary" size="mini" icon="el-icon-edit"/>
          </router-link>
          <el-button type="danger" size="mini"
                     icon="el-icon-delete" @click="removeDataById(scope.row.id)"
          >删除
          </el-button>
          <el-button v-if="scope.row.status==1" type="primary" size="mini"
                     icon="el-icon-delete" @click="lockHostSet(scope.row.id,0)"
          >锁定
          </el-button>
          <el-button v-if="scope.row.status==0" type="danger" size="mini"
                     icon="el-icon-delete" @click="lockHostSet(scope.row.id,1)"
          >取消锁定
          </el-button>
        </template>
      </el-table-column>

    </el-table>
    <!-- 分页 -->
    <el-pagination
      :current-page="current"
      :page-size="limit"
      :total="total"
      style="padding: 30px 0; text-align: center;"
      layout="total, prev, pager, next, jumper"
      @current-change="fetchData"
    />
  </div>
</template>


<script>
import hospitalSet from '@/api/hosp/hospitalSet'

export default {
  name: 'list',
  // 定义数据模型
  data() {
    return {
      searchObj: {
        'hosname': '',
        'hoscode': ''
      },
      list: [], // 列表
      multipleSelection: [], // 批量选择中选择的记录列表
      current: 1, //当前页
      limit: 10, //每页显示记录数
      total: 0//总记录数
    }
  },
  // 页面渲染成功后获取数据
  created() {
    this.fetchData()
  },

  methods: {
    // 加载列表数据
    fetchData(page = 1) {
      // console.log('翻页。。。' + page)
      // 异步获取远程数据（ajax）
      this.current = page
      // console.log("this.searchObj**",this.searchObj)
      hospitalSet.getPageList(this.current, this.limit, this.searchObj).then(
        async response => {
          //返回集合赋值list
          this.list = await response.data.records
          //总记录数
          this.total = await response.data.total
        }).catch(error => {//请求失败
        console.log(error)
      })
    },
    //删除医院设置的方法
    removeDataById(id) {
      // console.log(`${id}`)
      this.$confirm('此操作将永久删除医院是设置信息, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => { //确定执行then方法
        //调用接口
        hospitalSet.deleteHospSet(id)
          .then(async response => {
            //提示
            this.$message({
              type: 'success',
              message: '删除成功!'
            })
            this.current = 1
            //刷新页面
            await this.fetchData(this.current)
          })
      })
    },
    // 当表格复选框选项发生变化的时候触发
    handleSelectionChange(selection) {
      this.multipleSelection = selection
      // console.log("this.multipleSelection:",this.multipleSelection)
    },
    //批量删除
    removeRows() {
      // alert("111")
      this.$confirm('此操作将永久删除医院是设置信息, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => { //确定执行then方法
        let idList = []
        //遍历数组得到每个id值，设置到idList里面
        for (let i = 0; i < this.multipleSelection.length; i++) {
          let obj = this.multipleSelection[i]
          let id = obj.id
          idList.push(id)
        }
        console.log('idList:', idList)
        //调用接口
        hospitalSet.batchRemoveHospSet(idList)
          .then(async response => {
            //提示
            this.$message({
              type: 'success',
              message: '删除成功!'
            })
            //刷新页面
            this.current = 1
            await this.fetchData(this.current)
          })
      })
    },
    //锁定和取消锁定
    lockHostSet(id, status) {
      hospitalSet.lockHospSet(id, status)
        .then(response => {
          //刷新
          this.fetchData(this.current)
        }).catch(error => {
        alert(error)
      })
    }

  }
}

</script>

<style scoped>

</style>

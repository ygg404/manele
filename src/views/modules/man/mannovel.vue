<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="pageIndex=1,getDataList()">查询</el-button>
        <el-button v-if="isAuth('man:mannovel:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('man:mannovel:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table :data="dataList" border v-loading="dataListLoading" @selection-change="selectionChangeHandle" @sort-change="changeSort" style="width: 100%;">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="manid" header-align="center" align="center" label="漫画ID" sortable="custom" :sort-orders="['descending','ascending']"></el-table-column>
      <el-table-column prop="manname" header-align="center" align="center" label="漫画名" ></el-table-column>
      <el-table-column prop="othername" header-align="center" align="center" label="别名" :show-overflow-tooltip="true"></el-table-column>
      <el-table-column prop="imgUrl" header-align="center" align="center" label="标题图片"></el-table-column>
      <el-table-column prop="author" header-align="center" align="center" label="作者"></el-table-column>
      <el-table-column prop="keys" header-align="center" align="center" label="关键字" :show-overflow-tooltip="true"></el-table-column>
      <el-table-column prop="content" header-align="center" align="center" label="简介" :show-overflow-tooltip="true"></el-table-column>
      <el-table-column prop="area" header-align="center" align="center" label="地区"></el-table-column>
      <el-table-column prop="category" header-align="center" align="center" label="类别"></el-table-column>
      <el-table-column prop="paurl" header-align="center" align="center" label="爬虫地址" :show-overflow-tooltip="true"></el-table-column>
      <el-table-column prop="status" header-align="center" align="center" label="状态" :show-overflow-tooltip="true"></el-table-column>
      <el-table-column prop="isenable" header-align="center" align="center" label="是否有效">
        <template slot-scope="scope">
          <el-tag type="primary" v-if="scope.row.isenable == 1">有效</el-tag>
          <el-tag type="danger" v-else>无效</el-tag>
        </template>

      </el-table-column>
      <el-table-column prop="displayTime" header-align="center" align="center" label="上映时间"  width="110" sortable="custom" :sort-orders="['descending','ascending']"></el-table-column>
      <el-table-column prop="label" header-align="center" align="center" label="标签"></el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button type="primary" size="mini" @click="addOrUpdateHandle(scope.row.manid)">修改</el-button>
          <el-button type="danger" size="mini" @click="deleteHandle(scope.row.manid)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[25, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
  import AddOrUpdate from './mannovel-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          key: '',
          sidx: 'manid',
          order: 'desc'
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 25,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 排序字段改变
      changeSort (val) {
        console.log(val)
        switch (val.order) {
          case 'ascending':
            this.dataForm.order = 'asc'
            break
          case 'descending':
            this.dataForm.order = 'desc'
            break
          default:
            this.dataForm.order = 'desc'
        }
        this.dataForm.sidx = val.prop
        this.getDataList()
      },
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/man/mannovel/page'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.key,
            'sidx': this.dataForm.sidx,
            'order': this.dataForm.order
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.manid
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/man/mannovel/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500
              })
              this.getDataList()
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      }
    }
  }
</script>

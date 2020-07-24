<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm">
      <el-form-item label="选择书名：">
        <el-select v-model="novelId" >
          <el-option v-for="item in novelList" :label="item.novelName" :key="item.id" :value="item.id"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="getSectionList()">查询</el-button>
        <el-button v-if="isAuth('article:content:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('article:content:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="sectionTreeList"
      style="width: 100%;margin-bottom: 20px;"
      row-key="id"
      border
      default-expand-all
      :tree-props="{children: 'children', hasChildren: 'hasChildren'}">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="orderNum" header-align="center" align="center" label="序列号"></el-table-column>
      <el-table-column prop="sectionName" header-align="center" align="center" label="章名"></el-table-column>
      <el-table-column prop="id" header-align="center" align="center" label="章节ID"></el-table-column>
      <el-table-column prop="parentId" header-align="center" align="center" label="章节父id"></el-table-column>
      <el-table-column prop="affixFile" header-align="center" align="center" label="附件"></el-table-column>
      <el-table-column prop="menuFlag" header-align="center" align="center" label="是否菜单">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.menuFlag" type="primary">是</el-tag>
          <el-tag v-else type="info"> 否</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="createTime" header-align="center" align="center" label="创建时间"></el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button type="primary" size="mini" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="danger" size="mini" @click="deleteHandle(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getSectionList"></add-or-update>
  </div>
</template>

<script>
  import AddOrUpdate from './content-add-or-update'
  import {treeDataTranslate} from '../../../utils'
  export default {
    data () {
      return {
        dataForm: {
          key: '',
          sidx: 'id',
          order: 'desc'
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 25,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        novelList: [],
        novelId: 1,
        sectionTreeList: []
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getNovelList()
      this.getSectionList()
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
      // 获取书名列表
      getNovelList () {
        this.$http({
          url: this.$http.adornUrl('/article/novel/list'),
          method: 'get',
          params: this.$http.adornParams({})
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.novelList = data.list
          } else {
            this.dataList = []
          }
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
        let item = {
          id: id,
          novelId: this.novelId,
          novelName: this.novelList.find(novel => novel.id === this.novelId)['novelName']
        }
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(item)
        })
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/article/content/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500
              })
              this.getSectionList()
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      },
      // 获取文章列表
      getSectionList () {
        this.$http({
          url: this.$http.adornUrl(`/article/content/getSectionList`),
          method: 'get',
          params: this.$http.adornParams({
            novelId: this.novelId
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.list
            this.sectionTreeList = treeDataTranslate(data.list)
          }
        })
      },
    }
  }
</script>

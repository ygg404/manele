<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="章名" prop="sectionName">
      <el-input v-model="dataForm.sectionName" placeholder="章名"></el-input>
    </el-form-item>
    <el-form-item label="" prop="articleId">
      <el-input v-model="dataForm.articleId" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="书名" prop="articleName">
      <el-input v-model="dataForm.articleName" placeholder="书名"></el-input>
    </el-form-item>
    <el-form-item label="章节父id" prop="parentId">
      <el-input v-model="dataForm.parentId" placeholder="章节父id"></el-input>
    </el-form-item>
    <el-form-item label="内容" prop="content">
      <el-input v-model="dataForm.content" placeholder="内容"></el-input>
    </el-form-item>
    <el-form-item label="附件" prop="affixFile">
      <el-input v-model="dataForm.affixFile" placeholder="附件"></el-input>
    </el-form-item>
    <el-form-item label="创建时间" prop="createTime">
      <el-input v-model="dataForm.createTime" placeholder="创建时间"></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          sectionName: '',
          articleId: '',
          articleName: '',
          parentId: '',
          content: '',
          affixFile: '',
          createTime: ''
        },
        dataRule: {
          sectionName: [
            { required: true, message: '章名不能为空', trigger: 'blur' }
          ],
          articleId: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          articleName: [
            { required: true, message: '书名不能为空', trigger: 'blur' }
          ],
          parentId: [
            { required: true, message: '章节父id不能为空', trigger: 'blur' }
          ],
          content: [
            { required: true, message: '内容不能为空', trigger: 'blur' }
          ],
          affixFile: [
            { required: true, message: '附件不能为空', trigger: 'blur' }
          ],
          createTime: [
            { required: true, message: '创建时间不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/article/articlecontent/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.sectionName = data.articlecontent.sectionName
                this.dataForm.articleId = data.articlecontent.articleId
                this.dataForm.articleName = data.articlecontent.articleName
                this.dataForm.parentId = data.articlecontent.parentId
                this.dataForm.content = data.articlecontent.content
                this.dataForm.affixFile = data.articlecontent.affixFile
                this.dataForm.createTime = data.articlecontent.createTime
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/article/articlecontent/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'sectionName': this.dataForm.sectionName,
                'articleId': this.dataForm.articleId,
                'articleName': this.dataForm.articleName,
                'parentId': this.dataForm.parentId,
                'content': this.dataForm.content,
                'affixFile': this.dataForm.affixFile,
                'createTime': this.dataForm.createTime
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500
                })
                this.visible = false
                this.$emit('refreshDataList')
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>

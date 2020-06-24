<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="漫画名" prop="manname">
      <el-input v-model="dataForm.manname" placeholder="漫画名"></el-input>
    </el-form-item>
    <el-form-item label="别名" prop="othername">
      <el-input v-model="dataForm.othername" placeholder="别名"></el-input>
    </el-form-item>
    <el-form-item label="标题图片" prop="imgUrl">
      <el-input v-model="dataForm.imgUrl" placeholder="标题图片"></el-input>
    </el-form-item>
    <el-form-item label="作者" prop="author">
      <el-input v-model="dataForm.author" placeholder="作者"></el-input>
    </el-form-item>
    <el-form-item label="关键字" prop="keys">
      <el-input v-model="dataForm.keys" placeholder="关键字"></el-input>
    </el-form-item>
    <el-form-item label="简介" prop="content">
      <el-input v-model="dataForm.content" placeholder="简介"></el-input>
    </el-form-item>
    <el-form-item label="地区" prop="area">
      <el-input v-model="dataForm.area" placeholder="地区"></el-input>
    </el-form-item>
    <el-form-item label="类别" prop="category">
      <el-input v-model="dataForm.category" placeholder="类别"></el-input>
    </el-form-item>
    <el-form-item label="爬虫地址" prop="paurl">
      <el-input v-model="dataForm.paurl" placeholder="爬虫地址"></el-input>
    </el-form-item>
    <el-form-item label="状态" prop="status">
      <el-input v-model="dataForm.status" placeholder="状态"></el-input>
    </el-form-item>
    <el-form-item label="是否有效" prop="isenable">
      <el-switch v-model="dataForm.isenable"  ></el-switch>
    </el-form-item>
    <el-form-item label="上映时间" prop="displayTime">
      <el-input v-model="dataForm.displayTime" placeholder="上映时间"></el-input>
    </el-form-item>
    <el-form-item label="标签" prop="label">
      <el-input v-model="dataForm.label" placeholder="标签"></el-input>
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
          manid: 0,
          manname: '',
          othername: '',
          imgUrl: '',
          author: '',
          keys: '',
          content: '',
          area: '',
          category: '',
          paurl: '',
          status: '',
          isenable: '',
          displayTime: '',
          label: ''
        },
        dataRule: {
          manname: [
            { required: true, message: '漫画名不能为空', trigger: 'blur' }
          ],
          othername: [
            { required: true, message: '别名不能为空', trigger: 'blur' }
          ],
          imgUrl: [
            { required: true, message: '标题图片不能为空', trigger: 'blur' }
          ],
          author: [
            { required: false, message: '作者不能为空', trigger: 'blur' }
          ],
          keys: [
            { required: true, message: '关键字不能为空', trigger: 'blur' }
          ],
          content: [
            { required: true, message: '简介不能为空', trigger: 'blur' }
          ],
          area: [
            { required: true, message: '地区不能为空', trigger: 'blur' }
          ],
          category: [
            { required: true, message: '类别不能为空', trigger: 'blur' }
          ],
          paurl: [
            { required: true, message: '爬虫地址不能为空', trigger: 'blur' }
          ],
          status: [
            { required: true, message: '状态不能为空', trigger: 'blur' }
          ],
          isenable: [
            { required: true, message: '是否有效（0、无效；1、有效）不能为空', trigger: 'blur' }
          ],
          displayTime: [
            { required: true, message: '上映时间不能为空', trigger: 'blur' }
          ],
          label: [
            { required: true, message: '标签不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.manid = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.manid) {
            this.$http({
              url: this.$http.adornUrl(`/man/mannovel/info/${this.dataForm.manid}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.manname = data.manNovel.manname
                this.dataForm.othername = data.manNovel.othername
                this.dataForm.imgUrl = data.manNovel.imgUrl
                this.dataForm.author = data.manNovel.author
                this.dataForm.keys = data.manNovel.keys
                this.dataForm.content = data.manNovel.content
                this.dataForm.area = data.manNovel.area
                this.dataForm.category = data.manNovel.category
                this.dataForm.paurl = data.manNovel.paurl
                this.dataForm.status = data.manNovel.status
                this.dataForm.isenable = data.manNovel.isenable
                this.dataForm.displayTime = data.manNovel.displayTime
                this.dataForm.label = data.manNovel.label
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
              url: this.$http.adornUrl(`/man/mannovel/${!this.dataForm.manid ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'manid': this.dataForm.manid || undefined,
                'manname': this.dataForm.manname,
                'othername': this.dataForm.othername,
                'imgUrl': this.dataForm.imgUrl,
                'author': this.dataForm.author,
                'keys': this.dataForm.keys,
                'content': this.dataForm.content,
                'area': this.dataForm.area,
                'category': this.dataForm.category,
                'paurl': this.dataForm.paurl,
                'status': this.dataForm.status,
                'isenable': this.dataForm.isenable,
                'displayTime': this.dataForm.displayTime,
                'label': this.dataForm.label
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

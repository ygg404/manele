<template>
  <el-dialog width="80%"
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false" @opened="initEditor()"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" label-width="80px">
      <el-form-item label="书名ID" prop="novelId">
        <el-input v-model="dataForm.novelId" placeholder="" disabled></el-input>
      </el-form-item>
      <el-form-item label="书名" prop="novelName">
        <el-input v-model="dataForm.novelName" placeholder="书名" disabled></el-input>
      </el-form-item>
      <el-form-item label="章名" prop="sectionName">
        <el-input v-model="dataForm.sectionName" placeholder="章名"></el-input>
      </el-form-item>
      <el-form-item label="章节父id" prop="parentId">
        <el-popover
          ref="menuListPopover"
          placement="bottom-start"
          trigger="click">
          <el-tree
            :data="sectionTreeList"
            :props="menuListTreeProps"
            node-key="menuId"
            ref="menuListTree"
            @current-change="sectionTreeCurrentChangeHandle"
            :default-expand-all="true"
            :highlight-current="true"
            :expand-on-click-node="false">
          </el-tree>
        </el-popover>
        <el-input v-model="dataForm.parentName" v-popover:menuListPopover :readonly="true" placeholder="点击选择上级章节" class="menu-list__input" clearable></el-input>
      </el-form-item>
      <el-form-item label="章节父id" prop="parentId">
        <el-input v-model="dataForm.parentId" placeholder="章节父id"></el-input>
      </el-form-item>
      <el-form-item label="关键字" prop="mainKey">
        <el-input v-model="dataForm.mainKey" placeholder="关键字"></el-input>
      </el-form-item>
      <el-form-item label="内容" prop="content">
        <div style="text-align:left"  :id="id" :ref="id" v-loading="loadingVisible" element-loading-text="图片上传中......"></div>
<!--        <wang-editor :key="dataForm.id" :id="id" :content="dataForm.content"  :sectionNo="sectionNo" @refreshContent="getReportHandle"></wang-editor>-->
      </el-form-item>
      <el-form-item label="是否菜单" prop="menuFlag">
        <el-switch v-model="dataForm.menuFlag"  ></el-switch>
      </el-form-item>
      <el-form-item label="附件" prop="affixFile">
        <el-input v-model="dataForm.affixFile" placeholder="附件"></el-input>
      </el-form-item>
      <el-form-item label="排序号" prop="orderNum">
        <el-input-number v-model="dataForm.orderNum" controls-position="right" :min="0" label="排序号"></el-input-number>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import E from 'wangeditor'
  import {treeDataTranslate} from '../../../utils'
  import Vue from 'vue'

  export default {
    data () {
      return {
        visible: false,
        id: 'qualityEditor',
        dataForm: {
          id: 0,
          sectionName: '',
          novelId: '',
          novelName: '',
          parentId: '',
          parentName: '',
          content: '',
          affixFile: '',
          createTime: '',
          menuFlag: false,
          orderNum: '',
          mainKey: ''
        },
        sectionNo: '',
        dataRule: {
          sectionName: [
            { required: true, message: '章名不能为空', trigger: 'blur' }
          ],
          novelId: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          novelName: [
            { required: true, message: '书名不能为空', trigger: 'blur' }
          ],
          orderNum: [
            { required: true, message: '序列号不能为空', trigger: 'blur' }
          ]
        },
        loadingVisible: false,
        // 章节列表
        sectionList: [],
        sectionTreeList: [],
        menuListTreeProps: {
          label: 'sectionName',
          children: 'children'
        }
      }
    },
    methods: {
      initEditor () {
        let that = this
        this.editor = new E('#' + this.id)
        this.editor.customConfig.uploadImgShowBase64 = false // base 64 存储图片
        this.editor.customConfig.uploadImgServer = window.SITE_CONFIG['baseUrl'] + '/article/content/uploadEditorImg'
        this.editor.customConfig.uploadFileName = 'file' // 后端接受上传文件的参数名
        this.editor.customConfig.uploadImgMaxSize = 3 * 1024 * 1024 // 将图片大小限制为 3M
        this.editor.customConfig.uploadImgTimeout = 2 * 60 * 1000 // 设置超时时间
        this.editor.customConfig.uploadImgParams = {
          sectionNo: this.sectionNo
        }
        this.editor.customConfig.uploadImgHeaders = {
          'token': Vue.cookie.get('token')
        }
        this.editor.customConfig.onchange = (html) => {
          this.dataForm.content = html
          // this.$emit('refreshContent', html)
        }
        // 设置上传本地图片的钩子事件
        this.editor.customConfig.uploadImgHooks = {
          before: function (xhr, editor, files) {
            // 图片上传之前触发
            // xhr 是 XMLHttpRequst 对象，editor 是编辑器对象，files 是选择的图片文件
            // 如果返回的结果是 {prevent: true, msg: 'xxxx'} 则表示用户放弃上传
            // return {
            //     prevent: true,
            //     msg: '放弃上传'
            // }
            that.value = 0
            that.timer = setInterval(function () {
              that.value += 1
            }, 200)
            that.loadingVisible = true
          },
          success: function (xhr, editor, result) {
            // 图片上传并返回结果，图片插入成功之后触发
            // xhr 是 XMLHttpRequst 对象，editor 是编辑器对象，result 是服务器端返回的结果
            that.editor.cmd.do('insertHtml', '<img src="' + window.SITE_CONFIG['uploadUrl'] + 'report/' + result.imgName + '" style="max-width:100%;"/>')
            that.$message('上传图片成功！')
            that.loadingVisible = false
            clearInterval(that.timer)
          },
          fail: function (xhr, editor, result) {
            // 图片上传并返回结果，但图片插入错误时触发
            // xhr 是 XMLHttpRequst 对象，editor 是编辑器对象，result 是服务器端返回的结果
            that.$message.error('上传图片失败！')
            that.loadingVisible = false
            clearInterval(that.timer)
          },
          error: function (xhr, editor) {
            // 图片上传出错时触发
            // xhr 是 XMLHttpRequst 对象，editor 是编辑器对象
            that.$message.error('上传图片失败！')
            that.loadingVisible = false
            clearInterval(that.timer)
          },
          timeout: function (xhr, editor) {
            // 图片上传超时时触发
            // xhr 是 XMLHttpRequst 对象，editor 是编辑器对象
            that.loadingVisible = false
            that.$message.error('上传图片超时！')
            clearInterval(that.timer)
          },
          customInsert: function (insertImg, result, editor) {
            // 图片上传并返回结果，自定义插入图片的事件（而不是编辑器自动插入图片！！！）
            // insertImg 是插入图片的函数，editor 是编辑器对象，result 是服务器端返回的结果
            // 举例：假如上传图片成功后，服务器端返回的是 {url:'....'} 这种格式，即可这样插入图片：
            // result 必须是一个 JSON 格式字符串！！！否则报错
          }
        }
        this.editor.create()  // 生成编辑器
        this.editor.$textContainerElem.css('height', '500px') // 设置高度
        this.editor.txt.html(this.dataForm.content)
        // 设置监听粘贴板内容
        // const IDS = document.getElementById(this.editor.textElemId)
        // IDS.addEventListener('paste', (e) => {
        //   let clipboard = e.clipboardData
        //   let type = clipboard.items[0].type
        //   if (type.match(/image/)) {
        //     // 如果是图片文件 则上传到服务器
        //     let blob = clipboard.items[0].getAsFile()
        //     let file = new FileReader()
        //     file.addEventListener('loadend', function (e) {
        //       that.upReportImgHandle( e.target.result )
        //     })
        //     file.readAsDataURL(blob)
        //   }
        // })
      },
      init (item) {
        console.log(item)
        this.dataForm.id = item.id || 0
        this.dataForm.novelName = item.novelName
        this.dataForm.novelId = item.novelId
        this.visible = true
        this.$nextTick(() => {
          this.getSectionList()
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/article/content/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.sectionName = data.articleContent.sectionName
                this.dataForm.articleId = data.articleContent.articleId
                this.dataForm.articleName = data.articleContent.articleName
                this.dataForm.parentId = data.articleContent.parentId
                this.dataForm.content = data.articleContent.content
                this.dataForm.affixFile = data.articleContent.affixFile
                this.dataForm.createTime = data.articleContent.createTime
                this.dataForm.orderNum = data.articleContent.orderNum
                this.dataForm.menuFlag = data.articleContent.menuFlag
                this.dataForm.mainKey = data.articleContent.mainKey
                this.getSectionList().then(list => {
                  if (data.articleContent.parentId != null && data.articleContent.parentId !== '') {
                    this.dataForm.parentName = list.find(dat => dat.id === data.articleContent.parentId)['sectionName']
                  } else {
                    this.dataForm.parentName = ''
                  }
                  this.sectionList = list
                  this.sectionTreeList = treeDataTranslate(list)
                })
              }
            })
          } else {

          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/article/content/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'sectionName': this.dataForm.sectionName,
                'novelId': this.dataForm.novelId,
                'novelName': this.dataForm.novelName,
                'parentId': this.dataForm.parentId,
                'content': this.dataForm.content,
                'affixFile': this.dataForm.affixFile,
                'orderNum': this.dataForm.orderNum,
                'menuFlag': this.dataForm.menuFlag,
                'mainKey': this.dataForm.mainKey
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
      },
      // 菜单树选中
      sectionTreeCurrentChangeHandle (data, node) {
        this.dataForm.parentId = data.id
        this.dataForm.parentName = data.sectionName
      },
      // 获取文章列表
      getSectionList () {
        return new Promise((resolve, reject) => {
          this.$http({
            url: this.$http.adornUrl(`/article/content/getSectionList`),
            method: 'get',
            params: this.$http.adornParams({
              novelId: this.dataForm.novelId
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              resolve(data.list)
            }
          })
        })
      },
    }
  }
</script>

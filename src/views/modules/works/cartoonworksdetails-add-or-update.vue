<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="作品id" prop="worksId">
      <el-input v-model="dataForm.worksId" placeholder="作品id"></el-input>
    </el-form-item>
    <el-form-item label="漫画章节ID 第几话" prop="cartoonChapterId">
      <el-input v-model="dataForm.cartoonChapterId" placeholder="漫画章节ID 第几话"></el-input>
    </el-form-item>
    <el-form-item label="漫画章节名字" prop="cartoonChapterName">
      <el-input v-model="dataForm.cartoonChapterName" placeholder="漫画章节名字"></el-input>
    </el-form-item>
    <el-form-item label="漫画页数" prop="cartoonPages">
      <el-input v-model="dataForm.cartoonPages" placeholder="漫画页数"></el-input>
    </el-form-item>
    <el-form-item label="注册时间" prop="createTime">
      <el-input v-model="dataForm.createTime" placeholder="注册时间"></el-input>
    </el-form-item>
    <el-form-item label="修改时间" prop="editTime">
      <el-input v-model="dataForm.editTime" placeholder="修改时间"></el-input>
    </el-form-item>
    <el-form-item label="逻辑删除状态 0-已删除 1-未删除" prop="deleteStatus">
      <el-input v-model="dataForm.deleteStatus" placeholder="逻辑删除状态 0-已删除 1-未删除"></el-input>
    </el-form-item>
    <el-form-item label="审核状态 0-审核中 1-审核成功 2-审核失败" prop="reviewStatus">
      <el-input v-model="dataForm.reviewStatus" placeholder="审核状态 0-审核中 1-审核成功 2-审核失败"></el-input>
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
          worksId: '',
          cartoonChapterId: '',
          cartoonChapterName: '',
          cartoonPages: '',
          createTime: '',
          editTime: '',
          deleteStatus: '',
          reviewStatus: ''
        },
        dataRule: {
          worksId: [
            { required: true, message: '作品id不能为空', trigger: 'blur' }
          ],
          cartoonChapterId: [
            { required: true, message: '漫画章节ID 第几话不能为空', trigger: 'blur' }
          ],
          cartoonChapterName: [
            { required: true, message: '漫画章节名字不能为空', trigger: 'blur' }
          ],
          cartoonPages: [
            { required: true, message: '漫画页数不能为空', trigger: 'blur' }
          ],
          createTime: [
            { required: true, message: '注册时间不能为空', trigger: 'blur' }
          ],
          editTime: [
            { required: true, message: '修改时间不能为空', trigger: 'blur' }
          ],
          deleteStatus: [
            { required: true, message: '逻辑删除状态 0-已删除 1-未删除不能为空', trigger: 'blur' }
          ],
          reviewStatus: [
            { required: true, message: '审核状态 0-审核中 1-审核成功 2-审核失败不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/works/cartoonworksdetails/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 20000) {
                this.dataForm.worksId = data.cartoonWorksDetails.worksId
                this.dataForm.cartoonChapterId = data.cartoonWorksDetails.cartoonChapterId
                this.dataForm.cartoonChapterName = data.cartoonWorksDetails.cartoonChapterName
                this.dataForm.cartoonPages = data.cartoonWorksDetails.cartoonPages
                this.dataForm.createTime = data.cartoonWorksDetails.createTime
                this.dataForm.editTime = data.cartoonWorksDetails.editTime
                this.dataForm.deleteStatus = data.cartoonWorksDetails.deleteStatus
                this.dataForm.reviewStatus = data.cartoonWorksDetails.reviewStatus
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
              url: this.$http.adornUrl(`/works/cartoonworksdetails/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'worksId': this.dataForm.worksId,
                'cartoonChapterId': this.dataForm.cartoonChapterId,
                'cartoonChapterName': this.dataForm.cartoonChapterName,
                'cartoonPages': this.dataForm.cartoonPages,
                'createTime': this.dataForm.createTime,
                'editTime': this.dataForm.editTime,
                'deleteStatus': this.dataForm.deleteStatus,
                'reviewStatus': this.dataForm.reviewStatus
              })
            }).then(({data}) => {
              if (data && data.code === 20000) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
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

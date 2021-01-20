<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    @close="dialogClose"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="100px">
    <el-form-item label="???" prop="attrGroupName">
      <el-input v-model="dataForm.attrGroupName" placeholder="???"></el-input>
    </el-form-item>
    <el-form-item label="???" prop="sort">
      <el-input v-model="dataForm.sort" placeholder="???"></el-input>
    </el-form-item>
    <el-form-item label="???" prop="descript">
      <el-input v-model="dataForm.descript" placeholder="???"></el-input>
    </el-form-item>
    <el-form-item label="??ͼ?" prop="icon">
      <el-input v-model="dataForm.icon" placeholder="??ͼ?"></el-input>
    </el-form-item>
    <el-form-item label="????????id" prop="catelogPath">
<!--      <el-input v-model="dataForm.catelogId" placeholder="????????id"></el-input>-->
      <el-cascader
        filterable
        placeholder="试试搜索：手机"
        v-model="dataForm.catelogPath"
        :props="props"
        :options="categories"></el-cascader>
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
        categories: [],
        props: {
          value: 'catId',
          label: 'name'
        },
        dataForm: {
          attrGroupId: 0,
          attrGroupName: '',
          sort: '',
          descript: '',
          icon: '',
          catelogPath: [],
          catelogId: 0
        },
        dataRule: {
          attrGroupName: [
            { required: true, message: '???不能为空', trigger: 'blur' }
          ],
          sort: [
            { required: true, message: '???不能为空', trigger: 'blur' }
          ],
          descript: [
            { required: true, message: '???不能为空', trigger: 'blur' }
          ],
          icon: [
            { required: true, message: '??ͼ?不能为空', trigger: 'blur' }
          ],
          catelogPath: [
            { required: true, message: '????????ids不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    created() {
      this.getCategories();
    },
    methods: {
      init (id) {
        this.dataForm.attrGroupId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.attrGroupId) {
            this.$http({
              url: this.$http.adornUrl(`/product/attrgroup/info/${this.dataForm.attrGroupId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.attrGroupName = data.attrGroup.attrGroupName
                this.dataForm.sort = data.attrGroup.sort
                this.dataForm.descript = data.attrGroup.descript
                this.dataForm.icon = data.attrGroup.icon
                this.dataForm.catelogId = data.attrGroup.catelogId
                //查出catelogId的完整路径
                this.dataForm.catelogPath = data.attrGroup.catelogPath
              }
            })
          }
        })
      },
      getCategories() {
        this.$http({
          url: this.$http.adornUrl('/product/category/list/tree'),
          method: 'get'
        }).then(({data}) => {
          this.categories = data.data
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/product/attrgroup/${!this.dataForm.attrGroupId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'attrGroupId': this.dataForm.attrGroupId || undefined,
                'attrGroupName': this.dataForm.attrGroupName,
                'sort': this.dataForm.sort,
                'descript': this.dataForm.descript,
                'icon': this.dataForm.icon,
                'catelogId': this.dataForm.catelogPath[this.dataForm.catelogPath.length - 1]
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
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
      },
      dialogClose() {
        this.dataForm.catelogPath = [];
      }
    }
  }
</script>

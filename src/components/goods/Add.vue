<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-alert
        title="添加商品信息"
        type="info"
        center
        show-icon
        :closable="false"
      >
      </el-alert>
      <!-- 步骤条 -->
      <el-steps
        :space="200"
        :active="activeIndex - 0"
        finish-status="success"
        align-center
      >
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- 表单区域 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef">
        <!-- tab栏区域 -->
        <el-tabs
          v-model="activeIndex"
          :tab-position="'left'"
          lable-position="top"
          :before-leave="beforTabLeave"
          @tab-click="tabClicked"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <!-- 选择商品分类的级联选择框 -->
              <el-cascader
                expandTrigger="hover"
                v-model="addForm.goods_cat"
                :options="cateList"
                :props="cateProps"
                @change="handleChange"
                clearable
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <!-- 渲染表单item项 -->
            <el-form-item
              :label="item.attr_name"
              :key="item.attr_id"
              v-for="item in manyTableData"
            >
              <!-- 使用数组渲染复选框组 -->
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  border
                  :label="val"
                  v-for="(val, i) in item.attr_vals"
                  :key="i"
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item
              :label="item.attr_name"
              v-for="item in onlyTableData"
              :key="item.attr_id"
            >
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <!-- action 图片要上传的后台地址 -->
            <el-upload
              :action="uploadURL"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :headers="herderObj"
              :on-success="uploadSuccess"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <quill-editor v-model="addForm.goods_introduce">

            </quill-editor>
            <el-button type="primary" class="btnAdd" @click="add">确认添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 预览图片对话框 -->
    <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
      <img :src="previewPath" class="previewImg" />
    </el-dialog>
  </div>
</template>

<script>
// 官方推荐将lodash导入为_
import _ from 'lodash'
export default {
  data() {
    return {
      activeIndex: '0',
      // 添加商品表单
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        // 商品图片数组
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      cateProps: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      addFormRules: {
        goods_name: [
          {
            required: true,
            message: '请输入商品名称',
            trigger: 'blur'
          }
        ],
        goods_price: [
          {
            required: true,
            message: '请输入商品价格',
            trigger: 'blur'
          }
        ],
        goods_weight: [
          {
            required: true,
            message: '请输入商品重量',
            trigger: 'blur'
          }
        ],
        goods_number: [
          {
            required: true,
            message: '请输入商品数量',
            trigger: 'blur'
          }
        ],
        goods_cat: [
          {
            required: true,
            message: '请选择商品分类商品数量',
            trigger: 'blur'
          }
        ]
      },
      cateList: [],
      // 动态参数列表数据
      manyTableData: [],
      // 静态属性列表数据
      onlyTableData: [],
      uploadURL: 'http://127.0.0.1:8999/upload',
      // 请求头对象
      herderObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      // 保存预览图片的url地址
      previewPath: '',
      // 控制预览图片对话框的显示和隐藏
      previewVisible: false
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')

      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类数据失败')
      }
      this.cateList = res.data
    },
    // 级联选择器选中变化，则触发此函数
    handleChange() {
      // 如果用户选择的不是三级分类,该次选择无效，因为必须选择三级分类
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    beforTabLeave(activeName, oldActiveName) {
      // 在tab栏切换之前触发，两个形参为切换前，后的tab栏name
      if (oldActiveName === '0') {
        // 在第一个标签页的时候
        if (this.addForm.goods_cat.length !== 3) {
          this.$message.error('请选择商品的分类')
          return false
        }
        // else if (this.addForm.goods_name.trim() === '') {
        //   this.$message.error('请输入商品名称')
        //   return false
        // } else if (this.addForm.goods_price.trim() === '0') {
        //   this.$message.error('请输入商品价格')
        //   return false
        // } else if (this.addForm.goods_weight.trim() === '0') {
        //   this.$message.error('请输入商品重量')
        //   return false
        // } else if (this.addForm.goods_number.trim() === '0') {
        //   this.$message.error('请输入商品数量')
        //   return false
        // }
      }
    },
    async tabClicked() {
      if (this.activeIndex === '1') {
        if (!this.cateId) {
          return this.$message.error('未选中三级对象')
        }
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: { sel: 'many' }
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取动态列表失败')
        }
        res.data.forEach(item => {
          var emptyVals = item.attr_vals.length === 0
          item.attr_vals = emptyVals ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: { sel: 'only' }
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取静态属性失败')
        }
        this.onlyTableData = res.data
      }
    },
    uploadSuccess(response) {
      // 可能后端上传出错
      if (response.meta.status !== 200) {
        this.$message.error(response.meta.msg)
      }
      this.addForm.pics.push({ pic: response.data.tmp_path })
      // console.log(this.addForm.pics)
    },
    // 处理图片预览
    handlePreview(file) {
      // 当用户点击图片进行预览时执行，处理图片预览
    // 形参file就是用户预览的那个文件
      this.previewPath = file.response.data.url
      // 显示预览图片对话框
      this.previewVisible = true
    },
    // 处理图片移除操作
    handleRemove(file) {
      // 获取路径，在file对象中，使用console.log查看路径在哪个位置
      // 然后从数组pics中移除
      const filePath = file.response.data.tem_path
      // 使用findIndex来查找符合条件的索引
      const index = this.addForm.pics.findIndex(item => item.pic === filePath)
      // 移除索引对应的图片
      this.addForm.pics.splice(index, 1)
    },
    // 编写点击事件完成商品添加
    add() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return this.$message.error('请填写必要的表单项!')

        // 将addForm进行深拷贝，避免goods_cat数组转换字符串之后导致级联选择器报错
        const form = _.cloneDeep(this.addForm)
        // 将goods_cat从数组转换为"1,2,3"字符串形式
        form.goods_cat = form.goods_cat.join(',')
        // 处理attrs数组，数组中需要包含商品的动态参数和静态属性
        // 将manyTableData（动态参数）处理添加到attrs
        // 处理动态参数
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          this.addForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTableData.forEach(item => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals }
          this.addForm.attrs.push(newInfo)
        })
        form.attrs = this.addForm.attrs
        console.log(form)

        // 发起请求添加商品
        // 商品的名称，必须是唯一的
        const { data: res } = await this.$http.post('goods', form)

        if (res.meta.status !== 201) {
          return this.$message.error('添加商品失败！')
        }

        this.$message.success('添加商品成功！')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    cateId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 10px 0 0 !important;
}
.previewImg {
  width: 100%;
}
.btnAdd {
  margin-top: 15px;
}
</style>

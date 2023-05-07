<template>
  <div>
    <el-form ref="elForm" :model="formData" :rules="rules" size="medium" label-width="100px" class="el-form"
      label-position="left">
      <el-form-item label-width="126px" label="云函数地址" prop="url">
        <el-input v-model="formData.url" placeholder="请输入云函数地址" clearable :style="{ width: '100%' }"></el-input>
      </el-form-item>
      <el-form-item label-width="125px" label="访问凭证 (PAT)" prop="pat">
        <el-input v-model="formData.pat" placeholder="请输入访问凭证 (PAT)" clearable :style="{ width: '100%' }">
        </el-input>
      </el-form-item>
      <el-form-item label-width="125px" label="存储桶名称" prop="BUCKET_NAME">
        <el-input v-model="formData.BUCKET_NAME" placeholder="请输入存储桶名称" clearable :style="{ width: '100%' }">
        </el-input>
      </el-form-item>
      <el-form-item size="large" label-width="125px">
        <el-button v-loading.fullscreen.lock="fullscreenLoading" element-loading-text="正在上传中，大概会有1-2分钟" type="primary"
          @click="submitForm">提交</el-button>
        <el-button @click="resetForm">重置</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>
<script>
import { ElMessage, ElMessageBox } from 'element-plus'
export default {
  components: {},
  props: [],
  data() {
    return {
      fullscreenLoading: false,
      formData: {
        url: "",
        pat: "",
        BUCKET_NAME: "",
      },
      rules: {
        url: [{
          required: true,
          message: '请输入云函数地址',
          trigger: 'blur'
        },
        {
          validator: (rule, value, callback) => {
            const reg = /^https:\/\/[a-z0-9]{6,}\.[a-z0-9]{1,}\.[a-z0-9]{1,}\/[a-z0-9_]{1,}$/
            if (!reg.test(value)) {
              callback(new Error('云函数地址格式不正确'))
            } else {
              callback()
            }
          }
        }
        ],
        pat: [{
          required: true,
          message: '请输入访问凭证',
          trigger: 'blur',
        }, {
          validator: (rule, value, callback) => {
            const reg = /^laf_/
            if (!reg.test(value)) {
              callback(new Error('访问凭证必须以 laf_ 开头'))
            } else {
              callback()
            }
          }
        }],
        BUCKET_NAME: [{
          required: true,
          message: '请输入存储桶名称',
          trigger: 'blur'
        },
        // 判断格式为 y7lnyy-test -前面是 6 位数字加字母组合，后面数量不一定的数字加字母组合
        {
          validator: (rule, value, callback) => {
            const reg = /^[a-z0-9]{6,}-[a-z0-9]{1,}$/
            if (!reg.test(value)) {
              callback(new Error('存储桶名称格式不正确'))
            } else {
              callback()
            }
          }
        }
        ],
      },
    }
  },
  computed: {},
  watch: {},
  created() { },
  mounted() { },
  methods: {
    submitForm() {
      this.$refs['elForm'].validate(valid => {
        if (!valid) return
        console.log(this.formData);
        // TODO 提交表单
        uni.request({
          method: 'POST',
          url: 'https://y7lnyy.laf.dev/add_task', //仅为示例，并非真实接口地址。
          data: this.formData,
          success: async (res) => {
            console.log(res.data);
            this.fullscreenLoading = true

            // 每 30 秒获取一次状态
            const timer = setInterval(async () => {
              const res = await this.getStatus();
              console.log(res);
              // 如果返回值为 1，就停止
              if (res.status === 1) {
                this.fullscreenLoading = false
                this.open()
                clearInterval(timer);
              }
            }, 10000);
          }
        });
      })
    },
    resetForm() {
      this.$refs['elForm'].resetFields()
    },
    async getStatus() {
      const res = await uni.request({
        method: 'POST',
        url: 'https://y7lnyy.laf.dev/get_status',
        data: {
          BUCKET_NAME: this.formData.BUCKET_NAME
        }
      });
      console.log(res.data);
      return {
        status: res.data.status
      }
    },

    open() {
      ElMessageBox.alert('您的 AI 客户端已成功上传，点击跳转即可直接跳转', '提示', {
        confirmButtonText: '跳转',
        callback: () => {
          const BUCKET_NAME = this.formData.BUCKET_NAME
          const url = this.formData.url
          console.log(url);
          const domain = url.split('/')[2].split('.').slice(-2).join('.')
          console.log(domain);
          window.open(`https://${BUCKET_NAME}.site.${domain}`)
        },
      })
    }
  }
}

</script>
<style>
.el-form {
  margin: 2% 20%;
}
</style>

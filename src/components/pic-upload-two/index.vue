<template>
  <div>
    <el-upload
      class="pic-uploader-component"
      :action="uploadApi"
      :headers="headers"
      :show-file-list="false"
      :on-success="handleUploadSuccess"
      :before-upload="beforeAvatarUpload"
    >
      <img v-if="value" :src="resourcesUrl + value" class="pic">
      <i v-else class="el-icon-plus pic-uploader-icon" />
    </el-upload>
  </div>
</template>

<script>
import { getToken } from '@/utils/auth'
import { mapGetters } from 'vuex'
export default {
  props: {
    value: {
      default: '',
      type: String
    }
  },
  data() {
    return {
      resourcesUrl: '',
      headers: {
        'Authorization': getToken()
      }
    }
  },
  computed: {
    ...mapGetters([
      'uploadApi'
    ])
  },
  methods: {
    // 图片上传
    handleUploadSuccess(response, file, fileList) {
      console.log(file)
      this.$emit('input', file.response.Data)
    },
    // 限制图片上传大小
    beforeAvatarUpload(file) {
      const isLt2M = file.size / 1024 / 1024 < 2
      if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 2MB!')
      }
      return isLt2M
    }
  }
}
</script>

<style lang="scss" scoped>
  .pic-uploader-component .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
    .pic-uploader-icon {
      font-size: 28px;
      color: #8c939d;
      width: 100%;
      height: 40px;
      //line-height: 178px;
      text-align: center;
    }
    .pic {
      width: 100%;
      height: 40px;
      display: block;
    }
  }
  .pic-uploader-component .el-upload:hover {
    border-color: #409EFF;
  }

</style>

<template>
  <div class="review-material">
    <h2>复习资料上传/下载</h2>
    <el-upload
      class="upload-demo"
      action="/api/material/upload"
      :on-success="handleUploadSuccess"
      :on-error="handleUploadError"
      :show-file-list="false"
      :headers="uploadHeaders"
      :before-upload="beforeUpload"
      :data="{}"
    >
      <el-button size="small" type="primary">上传资料</el-button>
      <span style="margin-left: 10px; color: #888; font-size: 12px;">支持 pdf/doc/docx/ppt/pptx/jpg/jpeg/png，最大 20MB</span>
    </el-upload>
    <el-table :data="materials" style="width: 100%; margin-top: 20px;">
      <el-table-column prop="fileName" label="文件名" />
      <el-table-column prop="fileSize" label="大小" :formatter="formatSize" />
      <el-table-column prop="uploadTime" label="上传时间" />
      <el-table-column label="操作" width="100">
        <template slot-scope="scope">
          <el-button type="text" @click="downloadFile(scope.row)">下载</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'reviewMaterial',
  data() {
    return {
      materials: [],
      uploadHeaders: {
        // 如需 token 可在此添加
      }
    }
  },
  created() {
    this.fetchMaterials()
  },
  methods: {
    fetchMaterials() {
      axios.get('/api/material/list').then(res => {
        this.materials = res.data || []
      })
    },
    handleUploadSuccess(response, file) {
      console.log('上传成功', response, file)
      this.$message.success('上传成功')
      this.fetchMaterials()
    },
    handleUploadError(err, file) {
      console.error('上传失败', err, file)
      this.$message.error('上传失败，请检查网络或联系管理员')
    },
    beforeUpload(file) {
      const isAllowed = [
        'application/pdf',
        'application/msword',
        'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
        'application/vnd.ms-powerpoint',
        'application/vnd.openxmlformats-officedocument.presentationml.presentation',
        'image/jpeg',
        'image/png',
        'image/jpg'
      ].includes(file.type)
      const isLt20M = file.size / 1024 / 1024 < 20
      if (!isAllowed) {
        this.$message.error('仅支持 pdf/doc/docx/ppt/pptx/jpg/jpeg/png 格式')
      }
      if (!isLt20M) {
        this.$message.error('文件大小不能超过 20MB!')
      }
      return isAllowed && isLt20M
    },
    formatSize(row, column, cellValue) {
      if (!cellValue) return ''
      if (cellValue < 1024) return cellValue + ' B'
      if (cellValue < 1024 * 1024) return (cellValue / 1024).toFixed(1) + ' KB'
      return (cellValue / 1024 / 1024).toFixed(2) + ' MB'
    },
    downloadFile(row) {
      axios({
        url: `/api/material/download?fileName=${encodeURIComponent(row.fileName)}`,
        method: 'GET',
        responseType: 'blob'
      }).then(res => {
        const url = window.URL.createObjectURL(new Blob([res.data]))
        const link = document.createElement('a')
        link.href = url
        link.setAttribute('download', row.fileName)
        document.body.appendChild(link)
        link.click()
        document.body.removeChild(link)
      })
    }
  }
}
</script>

<style scoped>
.review-material {
  padding: 30px;
  max-width: 700px;
  margin: 0 auto;
}
</style> 
<script setup>
import FileUploader from './components/FileUploader.vue'

const handleSuccess = (response) => {
    console.log('上传成功:', response)
}

const handleError = (error) => {
    console.error('上传错误:', error)
}

const handleFilesChange = (files) => {
    console.log('当前文件列表:', files)
}
</script>

<template>
    <div class="content-box">
        <span class="text">文件上传自定义组件</span>
        <el-form label-width="120px" style="max-width: 800px;">
            <el-form-item label="单文件上传：">
                <FileUploader
                    class="uploader1"
                    :max-files="1"
                    :max-size="5"
                    :allowed-types="['image/png', 'application/pdf', 'application/msword', 
                        'application/vnd.openxmlformats-officedocument.wordprocessingml.document']"
                    action="http://127.0.0.1:8008/post"
                    @upload-success="handleSuccess"
                    @upload-error="handleError"
                    @files-change="handleFilesChange"
                />
            </el-form-item>
                
            <el-form-item label="多个文件上传：">
                <FileUploader
                    :max-files="3"
                    :max-size="5"
                    action="http://127.0.0.1:8008/post"
                    input-width="400px"
                    placeholder="请选择文件"
                    @upload-success="handleSuccess"
                    @upload-error="handleError"
                    @files-change="handleFilesChange"
                >
                    <template #upload-button="{uploading, disabled, upload}">
                        <el-button type="primary" @click="upload" :disabled="disabled">
                            {{ uploading ? '文件提交中...' : '点击提交' }}
                        </el-button>
                    </template>
                </FileUploader>
            </el-form-item>
        </el-form>
    </div>
    
</template>

<style scoped>
.uploader1 {
    padding: 0px;
}

.text {
    display: block;
    text-align: center;
    font-weight: bold;
    font-size: 1.5em;
    margin: 0 auto;
    padding: 20px 0 40px 0;
    width: fit-content;
}
.content-box {
    padding: 20px;
}
</style>
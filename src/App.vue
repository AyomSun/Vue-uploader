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
    <div>
        单个文件上传
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

        多个文件上传
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
    </div>
    

</template>


<style scoped>
    .uploader1 {
        padding: 20px;
    }
</style>
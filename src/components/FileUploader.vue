<script setup>
    import { ref, computed, nextTick, useAttrs } from 'vue'
    import axios from 'axios'
    import { Upload } from '@element-plus/icons-vue'
    import { ElMessage } from 'element-plus'

    const props = defineProps({
        maxFiles: {type: Number, default: 3},
        maxSize: {type: Number, default: 3},
        allowedTypes: {type: Array, default: () => ['image/png', 'application/pdf']},
        action: {type: String, required: true, default:"#"},
        headers: {type: Object, default: {'Content-Type': 'multipart/form-data'}},
        method: {type: String, default: "post"},
        inputWidth: {type: String, default: "350px"},
    })

    const emits = defineEmits(['upload-success', 'upload-error', 'files-change'])
    
    // 透传属性
    defineOptions({
        inheritAttrs: false
    })

    const attrs = useAttrs()

    const inputAttrs = computed(() => {
        const { class: _, style: __, ...rest } = attrs
        
        // 正确属性覆盖顺序
        return {
            placeholder: '未选择文件', 
            clearable: true,
            ...rest                  
        }
    })    
    
    const fileInputs = ref(null)
    const inputRef = ref(null)
    const files = ref([])
    const uploading = ref(false)
    const popoverStyle = ref({})

    // 计算属性控制multiple、显示文件名
    const enableMultiple = computed(() => props.maxFiles != 1);
    const fileNames = computed({        
        get: () => files.value.map(file => file.name).join(', ') || '',
        set: () => {} // 保持只读特性
    })

    // 触发文件选择对话框    
    const triggerFileInput = () => fileInputs.value.click()

    // 定位悬浮框
    const updatePopoverPosition = () => {
        nextTick(() => {
            if (!inputRef.value) return

            const inputEl = inputRef.value.$el.querySelector('.el-input__wrapper')            
            popoverStyle.value = { width: `${inputEl.clientWidth}px` }
        })
    }
    
    // 处理文件选择 带多种校验
    const handleFileSelect = (event) => {
        const selectedFiles = event.target.files
        if (!selectedFiles.length) return
        
        const newFiles = Array.from(selectedFiles)
        
        if (props.maxFiles != 1) {            
            const remainSlots = props.maxFiles - files.value.length

            if (newFiles.length > remainSlots) {
                ElMessage.error(`最多选择 ${props.maxFiles} 个文件，还可添加 ${remainSlots} 个`)
                event.target.value = ''
                return
            }
        }
        
        if (newFiles.some(f => f.size > props.maxSize * 1024 * 1024)) {
            ElMessage.error(`文件大小不能超过 ${props.maxSize} MB`) 
            event.target.value = ''
            return
        }

        const invalidTypes = newFiles
            .filter(f => !props.allowedTypes.includes(f.type))
            .map(f => f.name)
        if (invalidTypes.length) {
            ElMessage.error(`无效文件类型：${invalidTypes.join(', ')}`)
            event.target.value = ''
            return
        }
        
        if (props.maxFiles === 1) {
            files.value = newFiles
        } else {
            files.value = [...files.value, ...newFiles]     
        }

        emits('files-change', files.value)
        updatePopoverPosition()
    }

    // 清除文件
    const clearFile = () => {
        files.value = []
        fileInputs.value.value = '' // 重要：清空input值以便重新选择相同文件
        emits('files-change', [])
    }

    // 去掉单个文件
    const removeFile = (index) => {
        files.value.splice(index, 1)
        emits('files-change', files.value)
    }

    // 执行文件上传
    const uploadFile = async () => {
        if (!files.value.length) return
    
        const formData = new FormData()
        files.value.forEach((file) => formData.append('files[]', file))
    
        try {
            uploading.value = true
            const response = await axios({
                url: props.action, 
                method: props.method,
                data: formData, 
                headers: props.headers
            })
                
            ElMessage.success('文件上传成功')
            emits('upload-success', response.data)
            clearFile()
        } catch (error) {
            ElMessage.error(`上传失败：${error.message}`)
            emits('upload-error', error)
        } finally {
           uploading.value = false
        }
    }

    // 窗口变化时重新定位悬浮框
    window.addEventListener('resize', updatePopoverPosition)

    // 暴露必要状态给插槽
    defineExpose({
        uploadFile,
        files,
        uploading
    })
</script>


<template>
    <div :class="['upload-container', $attrs.class]" :style="$attrs.style">
        <!-- 隐藏的文件输入 -->
        <input type="file" ref="fileInputs" :multiple="enableMultiple" @change="handleFileSelect" style="display: none" />        
        
        <div class="input-wrapper">
            <!-- 显示文件名输入框 -->
            <el-input v-model="fileNames" ref="inputRef" v-bind="inputAttrs" @clear="clearFile" :style="{ width: inputWidth }">
                <!-- 选择文件按钮 -->
                <template #append>
                    <el-button :icon="Upload" @click="triggerFileInput" />
                </template>
            </el-input> 
            
            <!-- 文件悬浮框列表 -->
            <div class="file-list-popover" v-show="files.length > 0 && props.maxFiles > 1" :style="popoverStyle">
                <ul class="file-list">
                    <li class="file-item" v-for="(file, index) in files" :key="file.name">
                        <span>{{ file.name }}</span>
                        <el-button type="danger" @click="removeFile(index)">删除</el-button>
                    </li>
                </ul>
            </div>
        </div>        
    
        <!-- 上传按钮插槽 -->
        <slot
            name="upload-button"
            :uploading="uploading"
            :fileCount="files.length"
            :upload="uploadFile"
            :disabled="!files.length || uploading"
        >    
            <!-- 默认插槽 -->
            <el-button type="success" @click="uploadFile" :disabled="!files.length || uploading">
                {{ uploading ? '上传中...' : '上传文件' }}
            </el-button>
        </slot>
        
    </div>
</template>


<style scoped>
    .upload-container {
        display: flex;
        gap: 10px;
        align-items: center;
        padding: 20px;
    }
    
    .input-wrapper {
        position: relative;
    }

    .file-list-popover {
        position: absolute;
        background: white;
        border: 1px solid #ebeef5;
        border-radius: 4px;
        box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
        z-index: 2000;
        max-height: 300px;
        overflow-y: auto;
    }
    
    .file-list {
        list-style: none;
        margin: 0;
        padding: 0;
    }

    .file-item {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 8px 10px;
        transition: background-color 0.3s;
    }

    .file-item:hover {
        background-color: #f5f7fa;
    }

    .file-item + .file-item {
        border-top: 1px solid #f0f0f0;
    }

    :deep(.el-input__suffix) {
        right: 8px;
    }
</style>
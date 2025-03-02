<template>
  <el-dialog title="新建视频" v-model="showDialog" width="600px" destroy-on-close>
    <el-upload
      :action="UPLOAD_URL"
      :headers="HEADERS"
      multiple
      :limit="1"
      :file-list="fileList"
      :data="uploadData"
      :before-upload="beforeVideoUpload"
      :on-progress="() => (uploading = true)"
      :on-success="handleUploadSuccess"
      ref="uploadVideoRef"
      :auto-upload="false"
      class="mb-5"
    >
      <template #trigger>
        <el-button type="primary" plain>选择视频</el-button>
      </template>
      <span class="el-upload__tip" style="margin-left: 10px"
        >格式支持 MP4，文件大小不超过 10MB</span
      >
    </el-upload>
    <el-divider />
    <el-form :model="uploadData" :rules="uploadRules" ref="uploadFormRef" v-loading="uploading">
      <el-form-item label="标题" prop="title">
        <el-input
          v-model="uploadData.title"
          placeholder="标题将展示在相关播放页面，建议填写清晰、准确、生动的标题"
        />
      </el-form-item>
      <el-form-item label="描述" prop="introduction">
        <el-input
          :rows="3"
          type="textarea"
          v-model="uploadData.introduction"
          placeholder="介绍语将展示在相关播放页面，建议填写简洁明确、有信息量的内容"
        />
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button @click="showDialog = false">取 消</el-button>
      <el-button type="primary" @click="submitVideo" :loading="uploading" :disabled="uploading"
        >提 交</el-button
      >
    </template>
  </el-dialog>
</template>

<script setup lang="ts">
import type {
  FormInstance,
  FormRules,
  UploadInstance,
  UploadProps,
  UploadUserFile
} from 'element-plus'
import { HEADERS, UploadData, UPLOAD_URL, beforeVideoUpload, MaterialType } from './upload'

const message = useMessage()

const uploadRules: FormRules = {
  title: [{ required: true, message: '请输入标题', trigger: 'blur' }],
  introduction: [{ required: true, message: '请输入描述', trigger: 'blur' }]
}

const props = defineProps({
  modelValue: {
    type: Boolean,
    default: false
  }
})
const emit = defineEmits<{
  (e: 'update:modelValue', v: boolean)
  (e: 'uploaded', v: void)
}>()

const showDialog = computed({
  get() {
    return props.modelValue
  },
  set(val) {
    emit('update:modelValue', val)
  }
})

const uploading = ref(false)

const fileList = ref<UploadUserFile[]>([])

const uploadData: UploadData = reactive({
  type: MaterialType.Video,
  title: '',
  introduction: ''
})

const uploadFormRef = ref<FormInstance>()
const uploadVideoRef = ref<UploadInstance>()

const submitVideo = () => {
  uploadFormRef.value?.validate((valid) => {
    if (!valid) {
      return false
    }
    uploadVideoRef.value?.submit()
  })
}

const handleUploadSuccess: UploadProps['onSuccess'] = (res: any) => {
  uploading.value = false
  if (res.code !== 0) {
    message.error('上传出错：' + res.msg)
    return false
  }

  // 清空上传时的各种数据
  fileList.value = []
  uploadData.title = ''
  uploadData.introduction = ''

  showDialog.value = false
  message.notifySuccess('上传成功')
  emit('uploaded')
}
</script>

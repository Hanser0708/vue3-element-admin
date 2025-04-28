<template>
  <el-dialog
    v-model="dialogVisible"
    :title="title"
    width="600px"
    :before-close="handleClose"
    class="custom-dialog"
  >
    <el-form
      ref="formRef"
      :model="formData"
      :rules="rules"
      label-width="120px"
      label-position="left"
    >
      <!-- 环境名称 -->
      <el-form-item label="环境名称" prop="envName">
        <el-input v-model="formData.envName" />
      </el-form-item>

      <!-- 密码区域 -->
      <el-form-item label="SOPUSER密码" prop="sopPassword">
        <el-input v-model="formData.sopPassword" show-password />
      </el-form-item>

      <el-form-item label="ROOT密码" prop="rootPassword">
        <el-input v-model="formData.rootPassword" show-password />
      </el-form-item>

      <!-- 横向并排布局 -->
      <el-row :gutter="20">
        <!-- 业务面 -->
        <el-col :span="12">
          <el-form-item label="业务面IP" prop="bizIp">
            <el-input v-model="formData.bizIp" :disabled="isEdit" />
          </el-form-item>
          <el-form-item label="运维面账号" prop="opsAccount">
            <el-input v-model="formData.opsAccount" :disabled="isEdit" />
          </el-form-item>
          <el-form-item label="运维面密码" prop="opsPassword">
            <el-input v-model="formData.opsPassword" show-password />
          </el-form-item>
        </el-col>

        <!-- 管理面 -->
        <el-col :span="12">
          <el-form-item label="管理面IP" prop="mgrIp">
            <el-input v-model="formData.mgrIp" :disabled="isEdit" />
          </el-form-item>
          <el-form-item label="管理面账号" prop="mgrAccount">
            <el-input v-model="formData.mgrAccount" :disabled="isEdit" />
          </el-form-item>
          <el-form-item label="管理面密码" prop="mgrPassword">
            <el-input v-model="formData.mgrPassword" show-password />
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>

    <template #footer>
      <span class="dialog-footer">
        <el-button @click="handleClose">取消</el-button>
        <el-button type="primary" @click="handleSubmit">确定</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script setup lang="ts">
import { ref, reactive, computed, watch } from 'vue'
import type { FormInstance, FormRules } from 'element-plus'

interface FormData {
  envName: string
  sopPassword: string
  rootPassword: string
  bizIp: string
  opsAccount: string
  opsPassword: string
  mgrIp: string
  mgrAccount: string
  mgrPassword: string
}

const props = defineProps({
  visible: {
    type: Boolean,
    required: true
  },
  formData: {
    type: Object as () => FormData,
    default: () => ({})
  },
  type: {
    type: String as () => 'create' | 'edit',
    default: 'create'
  }
})

const emit = defineEmits(['update:visible', 'submit'])

const formRef = ref<FormInstance>()
const localFormData = reactive<FormData>({
  envName: '',
  sopPassword: '',
  rootPassword: '',
  bizIp: '',
  opsAccount: '',
  opsPassword: '',
  mgrIp: '',
  mgrAccount: '',
  mgrPassword: ''
})

// 深度监听传入的formData
watch(() => props.formData, (newVal) => {
  Object.assign(localFormData, newVal)
}, { deep: true, immediate: true })

const isEdit = computed(() => props.type === 'edit')

const title = computed(() => isEdit.value ? '修改配置' : '新建配置')

const rules = reactive<FormRules>({
  envName: [{ required: true, message: '请输入环境名称', trigger: 'blur' }],
  sopPassword: [{ required: true, message: '请输入SOPUSER密码', trigger: 'blur' }],
  rootPassword: [{ required: true, message: '请输入ROOT密码', trigger: 'blur' }],
  bizIp: [{ required: true, message: '请输入业务面IP', trigger: 'blur' }],
  opsAccount: [{ required: true, message: '请输入运维面账号', trigger: 'blur' }],
  opsPassword: [{ required: true, message: '请输入运维面密码', trigger: 'blur' }],
  mgrIp: [{ required: true, message: '请输入管理面IP', trigger: 'blur' }],
  mgrAccount: [{ required: true, message: '请输入管理面账号', trigger: 'blur' }],
  mgrPassword: [{ required: true, message: '请输入管理面密码', trigger: 'blur' }]
})

const handleClose = () => {
  emit('update:visible', false)
  formRef.value?.resetFields()
}

const handleSubmit = async () => {
  const valid = await formRef.value?.validate()
  if (valid) {
    emit('submit', { ...localFormData })
    handleClose()
  }
}
</script>

<style scoped>
.custom-dialog :deep(.el-dialog__header) {
  text-align: center;
}
</style>

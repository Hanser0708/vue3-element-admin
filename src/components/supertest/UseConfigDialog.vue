<template>
  <ConfigDialog
    v-model:visible="dialogVisible"
    :form-data="formData"
    :type="dialogType"
    @submit="handleSubmit"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import ConfigDialog from './ConfigDialog.vue'

const dialogVisible = ref(false)
const dialogType = ref<'create' | 'edit'>('create')
const formData = ref({})

// 打开创建弹窗
const openCreate = () => {
  dialogType.value = 'create'
  formData.value = {}
  dialogVisible.value = true
}

// 打开编辑弹窗
const openEdit = (data: any) => {
  dialogType.value = 'edit'
  formData.value = { ...data }
  dialogVisible.value = true
}

// 提交处理
const handleSubmit = (data: any) => {
  if (dialogType.value === 'create') {
    // 调用创建接口
  } else {
    // 调用修改接口
  }
}
</script>

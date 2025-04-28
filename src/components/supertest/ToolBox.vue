<template>
  <div class="page-container">
    <!-- 顶部操作栏 -->
    <div class="operation-bar">
      <!-- 左侧搜索区 -->
      <div class="left-section">
        <el-input
          v-model="searchParams.searchKey"
          placeholder="搜索关键词"
          clearable
          @change="emitSearch"
          style="width: 300px"
        >
          <template #prefix>
            <el-icon><Search /></el-icon>
          </template>
        </el-input>
      </div>

      <!-- 右侧操作区 -->
      <div class="right-section">
        <el-button type="primary" @click="emitCreate">
          <el-icon><Plus /></el-icon>新增
        </el-button>

        <BatchActions
          :actions="batchActions"
          :selected-rows="selectedRows"
          @action="handleBatchAction"
        />
      </div>
    </div>

    <!-- 内容区域 -->
    <div class="content-wrapper">
      <!-- 插槽用于放置不同页面的表格/列表 -->
      <slot :search-params="searchParams" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, defineProps, defineEmits } from 'vue'
import { Search, Plus } from '@element-plus/icons-vue'
import type { PropType } from 'vue'

// 类型定义
export interface BatchAction {
  label: string
  command: string
  icon?: string
  divided?: boolean
  danger?: boolean
}

const props = defineProps({
  batchActions: {
    type: Array as PropType<BatchAction[]>,
    required: true
  },
  selectedRows: {
    type: Array as PropType<any[]>,
    default: () => []
  }
})

const emit = defineEmits(['search', 'create', 'batch-action'])

// 搜索参数
const searchParams = ref({
  searchkey: '',
  isQueryAll: false
  // 可扩展其他搜索参数
})

const emitSearch = () => {
  emit('search', { ...searchParams.value })
}

const emitCreate = () => {
  emit('create')
}

const handleBatchAction = (command: string) => {
  emit('batch-action', command)
}
</script>

<style scoped>
.page-container {
  padding: 20px;
  background: #fff;
  border-radius: 4px;
}

.operation-bar {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

.left-section {
  display: flex;
  gap: 15px;
}

.right-section {
  display: flex;
  gap: 12px;
  align-items: center;
}
</style>

<template>
  <el-dropdown
    trigger="click"
    :disabled="!selectedRows.length"
    @command="handleCommand"
  >
    <el-button :disabled="!selectedRows.length">
      批量操作<el-icon><ArrowDown /></el-icon>
    </el-button>

    <template #dropdown>
      <el-dropdown-menu>
        <el-dropdown-item
          v-for="(action, index) in actions"
          :key="index"
          :command="action.command"
          :divided="action.divided"
          :class="{ 'danger-item': action.danger }"
        >
          <el-icon v-if="action.icon"><component :is="action.icon" /></el-icon>
          {{ action.label }}
        </el-dropdown-item>
      </el-dropdown-menu>
    </template>
  </el-dropdown>
</template>

<script setup lang="ts">
import { defineProps } from 'vue'
import { ArrowDown } from '@element-plus/icons-vue'
import type { PropType } from 'vue'
import type { BatchAction } from './BasicLayout.vue'

defineProps({
  actions: {
    type: Array as PropType<BatchAction[]>,
    required: true
  },
  selectedRows: {
    type: Array as PropType<any[]>,
    required: true
  }
})

const emit = defineEmits(['action'])

const handleCommand = (command: string) => {
  emit('action', command)
}
</script>

<style scoped>
.danger-item {
  color: var(--el-color-danger);
}
</style>

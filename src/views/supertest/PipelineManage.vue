<template>
  <div class="pipeline-wrapper">
    <!-- 标题区 -->
    <div class="header">
      <h2>{{ title }}</h2>
    </div>

    <!-- 表单区 -->
    <div class="form-container">
      <el-form :model="formData" label-width="100px">
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="名称">
              <el-input v-model="formData.name" placeholder="请输入流程名称" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="环境">
              <el-select
                v-model="formData.env"
                filterable
                remote
                :remote-method="searchEnv"
                placeholder="请选择环境"
              >
                <el-option
                  v-for="env in envOptions"
                  :key="env.value"
                  :label="env.label"
                  :value="env.value"
                />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="任务类型">
              <el-select v-model="formData.taskType" placeholder="请选择任务类型">
                <el-option label="定时任务" value="scheduled" />
                <el-option label="手动任务" value="manual" />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Cron表达式">
              <el-button @click="showCronDialog">
                {{ formData.cron || '设置执行时间' }}
              </el-button>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
    </div>

    <!-- 流水线区域 -->
    <div class="pipeline-area">
      <pipeline
        :tasks="pipelineData"
        @update="handlePipelineUpdate"
      />
    </div>

    <!-- Cron表达式弹窗 -->
    <el-dialog v-model="cronDialogVisible" title="Cron表达式生成器" width="700px">
      <!-- 这里可以接入cron表达式生成组件 -->
      <div class="cron-editor">
        <!-- 示例：简单实现 -->
        <el-input v-model="tempCron" placeholder="0 0 * * * ?" />
        <div class="cron-examples">
          <p>常用表达式：</p>
          <el-button-group>
            <el-button @click="tempCron = '0 0 * * * ?'">每天0点</el-button>
            <el-button @click="tempCron = '0 0/30 * * * ?'">每30分钟</el-button>
            <el-button @click="tempCron = '0 0 12 * * ?'">每天中午12点</el-button>
          </el-button-group>
        </div>
      </div>
      <template #footer>
        <el-button @click="cronDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="confirmCron">确定</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import Pipeline from './Pipeline.vue'

// 标题
const title = ref('工作流配置')

// 表单数据
const formData = reactive({
  name: '',
  env: '',
  taskType: '',
  cron: ''
})

// 环境选项
const envOptions = ref<Array<{value: string, label: string}>>([])
const allEnvOptions = [
  { value: 'dev', label: '开发环境' },
  { value: 'test', label: '测试环境' },
  { value: 'pre', label: '预发环境' },
  { value: 'prod', label: '生产环境' }
]

// 搜索环境
const searchEnv = (query: string) => {
  if (query) {
    envOptions.value = allEnvOptions.filter(item =>
      item.label.includes(query) || item.value.includes(query)
    )
  } else {
    envOptions.value = [...allEnvOptions]
  }
}

// Cron表达式相关
const cronDialogVisible = ref(false)
const tempCron = ref('')
const showCronDialog = () => {
  tempCron.value = formData.cron
  cronDialogVisible.value = true
}
const confirmCron = () => {
  formData.cron = tempCron.value
  cronDialogVisible.value = false
}

// 流水线数据
const pipelineData = ref([
  { name: '阶段1', subtasklist: [] },
  { name: '阶段2', subtasklist: [] }
])

const handlePipelineUpdate = (updatedTasks: any[]) => {
  pipelineData.value = updatedTasks
}
</script>

<style scoped>
.pipeline-wrapper {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.header {
  text-align: center;
  margin-bottom: 30px;
}

.header h2 {
  font-size: 24px;
  color: #333;
}

.form-container {
  background: #f5f7fa;
  padding: 20px;
  border-radius: 4px;
  margin-bottom: 30px;
}

.pipeline-area {
  border: 1px solid #ebeef5;
  border-radius: 4px;
  padding: 20px;
}

.cron-editor {
  padding: 20px;
}

.cron-examples {
  margin-top: 20px;
}

.cron-examples p {
  margin-bottom: 10px;
  color: #666;
}
</style>

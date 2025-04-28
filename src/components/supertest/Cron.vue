<template>
  <el-dialog
    v-model="visible"
    title="执行时间配置"
    width="680px"
    @close="handleClose"
  >
    <!-- 外层选项卡 -->
    <el-tabs v-model="activeTab">
      <el-tab-pane label="单次执行" name="single">
        <div class="time-picker">
          <el-date-picker
            v-model="singleTime"
            type="datetime"
            value-format="YYYY-MM-DD HH:mm:ss"
            placeholder="选择执行时间"
          />
        </div>
      </el-tab-pane>

      <el-tab-pane label="周期执行" name="cron">
        <div class="cron-config-container">
          <!-- 周期类型下拉框 -->
          <div class="type-select">
            <el-select
              v-model="cronType"
              placeholder="选择周期类型"
              style="width: 200px"
            >
              <el-option
                v-for="item in cronTypeOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </div>

          <!-- 时间选择 -->
          <div class="time-select">
            <span class="label">执行时间：</span>
            <el-time-picker
              v-model="baseTime"
              value-format="HH:mm"
              format="HH:mm"
              placeholder="选择时间"
            />
          </div>

          <!-- 具体配置区域 -->
          <div class="detail-config">
            <!-- 按天配置 -->
            <template v-if="cronType === 'day'">
              <div class="config-item">
                <div class="config-label">间隔天数：</div>
                <el-input-number
                  v-model="dayConfig.interval"
                  :min="1"
                  :max="30"
                />
              </div>
            </template>

            <!-- 按周配置 -->
            <template v-if="cronType === 'week'">
              <div class="config-item">
                <div class="config-label">选择星期：</div>
                <el-checkbox-group v-model="weekConfig.days">
                  <el-checkbox
                    v-for="day in weekDays"
                    :key="day.value"
                    :label="day.value"
                  >
                    {{ day.label }}
                  </el-checkbox>
                </el-checkbox-group>
              </div>
            </template>

            <!-- 按月配置 -->
            <template v-if="cronType === 'month'">
              <div class="config-item">
                <div class="config-label">每月日期：</div>
                <el-input-number
                  v-model="monthConfig.day"
                  :min="1"
                  :max="31"
                />
              </div>
            </template>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>

    <template #footer>
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="handleConfirm">确定</el-button>
    </template>
  </el-dialog>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'
import { parseExpression } from 'cron-parser'

// 类型定义
type CronType = 'day' | 'week' | 'month'

interface Props {
  modelValue?: string
  visible: boolean
}

const props = defineProps<Props>()
const emit = defineEmits(['update:visible', 'update:modelValue'])

// 选项卡状态
const activeTab = ref<'single' | 'cron'>('cron')

// 周期类型选项
const cronTypeOptions = [
  { value: 'day', label: '按天执行' },
  { value: 'week', label: '按周执行' },
  { value: 'month', label: '按月执行' }
]
const cronType = ref<CronType>('day')

// 其他状态和逻辑保持与之前实现一致...
// （单次时间、基础时间、各类型配置等代码参考前文实现）

</script>

<style scoped>
.cron-config-container {
  padding: 15px;
}

.type-select {
  margin-bottom: 20px;
}

.detail-config {
  border-top: 1px solid #eee;
  padding-top: 15px;
}

.config-item {
  margin: 15px 0;
  display: flex;
  align-items: center;

.config-label {
  width: 100px;
  margin-right: 15px;
  color: #666;
}
}

.time-select {
  margin: 20px 0;
  display: flex;
  align-items: center;

.label {
  width: 100px;
  margin-right: 15px;
  color: #666;
}
}

.el-checkbox {
  margin-right: 15px;
}
</style>

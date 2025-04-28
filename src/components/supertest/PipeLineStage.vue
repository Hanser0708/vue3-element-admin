<!-- PipelineStage.vue 子组件 -->
<template>
  <div class="stage-wrapper">
    <!-- 前加号按钮（非第一个阶段显示） -->
    <div
      v-if="!isFirst"
      class="add-stage-btn before"
      @click="$emit('add-stage-before')"
    >
      <el-icon>
        <Plus />
      </el-icon>
    </div>
    <div class="stage-container"
         @mouseenter="showDelete = true"
         @mouseleave="showDelete = false"
    >
      <!-- 新增删除按钮 -->
      <div v-show="showDelete" class="delete-stage" @click="handleRemove">
        <el-icon>
          <Close />
        </el-icon>
      </div>

      <div class="add-stage-btn left" @click="$emit('add-stage', index)">
        <el-icon>
          <Plus />
        </el-icon>
      </div>

      <div class="stage-content">
        <!-- 标题 -->
        <div class="stage-header">
          <el-input
            v-model="task.name"
            @blur="$emit('update-title', index, task.name)"
            class="title-input"
          />
        </div>

        <!-- 步骤区域 -->
        <div class="steps-container">
          <div
            v-for="(subtask, subIndex) in task.subtasklist"
            :key="subIndex"
            class="step-item"
          >
            {{ subtask.name }}
          </div>
          <el-button
            type="primary"
            class="add-step-btn"
            @click="showDialog = true"
          >
            添加任务
          </el-button>
        </div>
      </div>

      <!-- 后加号按钮（非最后一个阶段显示） -->
      <div
        v-if="!isLast"
        class="add-stage-btn after"
        @click="$emit('add-stage-after')"
      >
        <el-icon>
          <Plus />
        </el-icon>
      </div>

      <!-- 任务选择弹窗 -->
      <el-dialog v-model="showDialog" title="选择任务">
        <el-tabs v-model="activeTab">
          <el-tab-pane
            v-for="tab in 5"
            :key="tab"
            :label="`Tab ${tab}`"
          >
            <div class="task-grid">
              <div
                v-for="task in tasks"
                :key="task.id"
                class="task-card"
                :class="{ selected: selectedTasks.has(task.id) }"
                @click="toggleSelect(task.id)"
              >
                <img :src="task.icon" class="task-icon" />
                <div class="task-name">{{ task.name }}</div>
              </div>
            </div>
          </el-tab-pane>
        </el-tabs>
        <template #footer>
          <el-button @click="showDialog = false">取消</el-button>
          <el-button type="primary" @click="confirmSelection">确定</el-button>
        </template>
      </el-dialog>
    </div>
</template>

<script setup lang="ts">
import { ref, defineProps, defineEmits } from "vue";
import type { Task } from "./Pipeline.vue";


const props = defineProps<{
  task: Task[],
  index: number,
  isFirst: boolean,
  isLast: boolean
}>();

const emit = defineEmits([
  'update-title',
  'add-subtask',
  'add-stage-before',
  'add-stage-after',
  'remove-stage'
]);


const showDelete = ref(false)
const showDialog = ref(false)

const activeTab = ref("1");
const selectedTasks = ref<Set<number>>(new Set());
const tasks = ref<Task[]>(...props.task); // 这里应替换为接口数据

const toggleSelect = (id: number) => {
  if (selectedTasks.value.has(id)) {
    selectedTasks.value.delete(id);
  } else {
    selectedTasks.value.add(id);
  }
};



const handleRemove = () => {
  emit('remove-stage', props.index)
}

const confirmSelection = () => {
  const selected = tasks.value.filter(t => selectedTasks.value.has(t.id));
  emit("add-subtask", props.index, selected);
  showDialog.value = false;
  selectedTasks.value.clear();
};


</script>

<style scoped>
.stage-container {
  position: relative;
  min-width: 300px;
  border: 1px solid #ebeef5;
  border-radius: 4px;
  padding: 20px;
  margin: 0 40px;
}

.stage-header {
  margin-bottom: 20px;
  text-align: center;
}

.task-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 16px;
}

.task-card {
  border: 1px solid #ddd;
  padding: 12px;
  display: flex;
  align-items: center;
  cursor: pointer;
  transition: all 0.3s;
}

.task-card.selected {
  border-color: #409eff;
  background-color: #ecf5ff;
}

.task-icon {
  width: 40px;
  height: 40px;
  margin-right: 12px;
}

.add-step-btn {
  width: 100%;
  margin-top: 12px;
}

.steps-container {
  min-height: 200px;
}

.step-item {
  background: #f5f7fa;
  padding: 8px;
  margin-bottom: 8px;
  border-radius: 4px;
}

.add-stage-btn.left {
  left: -20px;
}

/* 新增删除按钮样式 */
.delete-stage {
  position: absolute;
  top: -10px;
  right: -10px;
  width: 24px;
  height: 24px;
  background: #ff4d4f;
  border-radius: 50%;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  z-index: 10;
  transition: all 0.3s;
}

.delete-stage:hover {
  background: #ff7875;
  transform: scale(1.1);
}

.stage-container {
  position: relative;
  min-width: 300px;
  border: 1px solid #ebeef5;
  border-radius: 4px;
  padding: 20px;
  margin: 0 40px;
  transition: all 0.3s;
}

.stage-container:hover {
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}

/* new*/
.stage-wrapper {
  position: relative;
  display: flex;
  align-items: center;
  min-height: 300px;
}

.stage-container {
  position: relative;
  min-width: 250px;
  border: 1px solid #ebeef5;
  border-radius: 4px;
  padding: 20px;
  transition: all 0.3s;
}

.add-stage-btn {
  cursor: pointer;
  font-size: 24px;
  color: #409eff;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.add-stage-btn:hover {
  transform: scale(1.2);
}

.add-stage-btn.before {
  margin-right: 10px;
}

.add-stage-btn.after {
  margin-left: 10px;
}

.add-stage-btn.middle {
  position: absolute;
  bottom: -20px;
  left: 50%;
  transform: translateX(-50%);
  background: white;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  z-index: 2;
}
</style>

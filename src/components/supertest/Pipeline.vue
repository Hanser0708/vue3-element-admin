<!-- Pipeline.vue 父组件 -->
<template>
  <div class="pipeline-container">
    <!--    <div v-for="(task, index) in taskList" :key="index" class="stage-wrapper">-->
    <pipeline-stage
      v-for="(task, index) in taskList"
      :key="index"
      :task="task"
      :index="index"
      :is-first="index === 0"
      :is-last="index === taskList.length - 1"
      @update-title="handleUpdateTitle"
      @add-subtask="handleAddSubtask"
      @add-stage-before="handleAddStage(index)"
      @add-stage-after="handleAddStage(index + 1)"
      @remove-stage="handleRemoveStage"
    />
    <!--      <div class="add-stage-btn" @click="addStage(index + 1)">-->
    <!--        <el-icon>-->
    <!--          <Plus />-->
    <!--        </el-icon>-->
    <!--      </div>-->
    <!--    </div>-->
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import PipelineStage from "./PipelineStage.vue";

interface Task {
  uuid: string;
  name: string;
  subtasklist: Task[];
}

const props = defineProps<{
  task: Task[]
}>();
const emit = defineEmits(["update"]);

const taskList = ref<Task[]>([{ name: "阶段1", subtasklist: [] }]);

watch(() => props.tasks, newVal = > {
  taskList.value = [...newVal],
}, {
  immediate: true,
  deep: true,
};
)

const updateTitle = (index: number, newTitle: string) => {
  taskList.value[index].name = newTitle;
  emitChanges();
};

const handleAddSubtask = (index: number, subtasks: Task[]) => {
  taskList.value[index].subtasklist.push(...subtasks);
  emitChanges();
};

const handleAddStage = (position: number) => {
  taskList.value.splice(position, 0, { name: `阶段${position + 1}`, subtasklist: [] });
  emitChanges();
};

// 通知外部变更
const emitChanges = () => {
  emit("update", [...internalTaskList.value]);
};


// 新增删除阶段方法
const handleRemoveStage  = (index: number) => {
  if (taskList.value.length > 1) {
    taskList.value.splice(index, 1);
    emitChanges();
  } else {
    ElMessage.warning("至少需要保留一个阶段");
  }
};
</script>

<style scoped>
.pipeline-container {
  display: flex;
  overflow-x: auto;
  padding: 20px 0;
  gap: 20px;
}

.stage-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.add-stage-btn {
  cursor: pointer;
  padding: 0 20px;
  font-size: 24px;
  color: #409eff;
}
</style>

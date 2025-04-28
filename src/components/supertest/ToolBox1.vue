<template>
  <div class="toolbar-container">
    <!-- 左侧搜索区域 -->
    <div class="left-section">
      <el-input
        v-model="searchParams.searchKey"
        placeholder="请输入关键词"
        clearable
        @change="handleSearch"
        style="width: 240px; margin-right: 16px"
      >
        <template #prefix>
          <el-icon>
            <Search />
          </el-icon>
        </template>
      </el-input>

      <el-switch
        v-model="searchParams.isQueryAll"
        active-text="启用状态"
        inactive-text="禁用状态"
        @change="handleSearch"
      />
    </div>

    <!-- 右侧操作区域 -->
    <div class="right-section">
      <el-button type="primary" @click="handleCreate">
        <el-icon>
          <Plus />
        </el-icon>
        新建
      </el-button>

      <el-dropdown
        trigger="click"
        :disabled="!selectedRows.length"
        @command="handleBatchCommand"
      >
        <el-button :disabled="!selectedRows.length">
          批量操作
          <el-icon>
            <ArrowDown />
          </el-icon>
        </el-button>

        <template #dropdown>
          <el-dropdown-menu>
            <el-dropdown-item
              v-for="(item, index) in batchActions"
              :key="index"
              :command="item.command"
              :divided="item.divided"
            >
              {{ item.label }}
            </el-dropdown-item>
          </el-dropdown-menu>
        </template>
      </el-dropdown>
    </div>
  </div>
</template>

<script setup>
import { ref, defineProps, defineEmits } from "vue";
import { ElMessage, ElMessageBox } from "element-plus";
import { Search, Plus, ArrowDown } from "@element-plus/icons-vue";
import api from "@/api";

const props = defineProps({
  // 批量操作配置
  batchActions: {
    type: Array,
    default: () => [
      { label: "批量删除", command: "delete" },
      { label: "批量导出", command: "export", divided: true },
    ],
  },
  // 已选中的行数据
  selectedRows: {
    type: Array,
    default: () => [],
  },
});

const emit = defineEmits([
  "create",
  "search",
  "batch-action",
]);

// 搜索参数
const searchParams = ref({
  searchKey: "",
  isQueryAll: false,
});

// 触发搜索
const handleSearch = () => {
  emit("search", searchParams.value);
};

// 新建操作
const handleCreate = () => {
  emit("create");
};

// 批量操作处理
const handleBatchCommand = async (command) => {
  if (!props.selectedRows.length) {
    return ElMessage.warning("请至少选择一项数据");
  }

  try {
    // 根据不同的命令执行不同操作
    switch (command) {
      case "delete":
        await handleBatchDelete();
        break;
      case "export":
        handleExport();
        break;
      default:
        // 自定义操作
        emit("batch-action", { command, data: props.selectedRows });
    }
  } catch (error) {
    console.error("批量操作失败:", error);
  }
};

// 批量删除
const handleBatchDelete = async () => {
  await ElMessageBox.confirm("确定要删除选中项吗？", "警告", {
    confirmButtonText: "确定",
    cancelButtonText: "取消",
    type: "warning",
  });

  const ids = props.selectedRows.map(item => item.id);
  await api.batchDelete(ids);
  ElMessage.success("删除成功");
  emit("batch-action", { command: "delete", data: props.selectedRows });
};

// 批量导出
const handleExport = async () => {
  const ids = props.selectedRows.map(item => item.id);
  const res = await api.exportData(ids);
  downloadFile(res);
  emit("batch-action", { command: "export", data: props.selectedRows });
};

// 文件下载方法
const downloadFile = (res) => {
  const blob = new Blob([res.data], { type: res.headers["content-type"] });
  const url = window.URL.createObjectURL(blob);
  const link = document.createElement("a");
  link.href = url;
  link.download = decodeURIComponent(
    res.headers["content-disposition"].split("filename=")[1],
  );
  document.body.appendChild(link);
  link.click();
  window.URL.revokeObjectURL(url);
  document.body.removeChild(link);
};
</script>

<style scoped>
.toolbar-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding: 16px;
  background: #fff;
  border-radius: 4px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, .1);
}

.left-section {
  display: flex;
  align-items: center;
}

.right-section {
  display: flex;
  gap: 12px;
}

.el-dropdown {
  margin-left: 12px;
}
</style>

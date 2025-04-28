<template>
  <BasicLayout
    :batch-actions="batchActions"
    :selected-rows="selectedRows"
    @search="handleSearch"
    @create="handleCreate"
    @batch-action="handleBatchAction"
  >
    <template #default="{ searchParams }">
      <!-- 表格组件 -->
      <el-table
        :data="tableData"
        @selection-change="handleSelectionChange"
      >
        <el-table-column type="selection" width="55" />
        <el-table-column prop="name" label="姓名" />
        <el-table-column prop="email" label="邮箱" />
        <!-- 其他列... -->
      </el-table>

      <!-- 分页 -->
      <el-pagination
        :current-page="pagination.current"
        :page-size="pagination.size"
        :total="pagination.total"
        @current-change="handlePageChange"
      />
    </template>
  </BasicLayout>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import BasicLayout from "./BasicLayout.vue";
import { getUserList, batchDeleteUsers } from "@/api/user";
import type { BatchAction } from "./BasicLayout.vue";

// 表格数据
const tableData = ref<any[]>([]);
const selectedRows = ref<any[]>([]);
const pagination = ref({
  current: 1,
  size: 10,
  total: 0,
});

// 批量操作配置
const batchActions = ref<BatchAction[]>([
  { label: "批量删除", command: "batchDelete", icon: "Delete", danger: true },
  { label: "批量更新", command: "batchUpdate", icon: "Update", divided: true },
]);

let timer;

// 获取数据
const loadData = async (params?: any) => {
  const res = await getUserList({
    ...params,
    page: pagination.value.current,
    size: pagination.value.size,
  });
  tableData.value = res.data.records;
  pagination.value.total = res.data.total;
};

// 搜索处理
const handleSearch = (params: any) => {
  pagination.value.current = 1;
  loadData(params);
};

// 批量操作处理
const handleBatchAction = async (command: string) => {
  switch (command) {
    case "batchDelete":
      await handleBatchDelete();
      break;
    case "batchUpdate":
      handleBatchUpdate();
      break;
  }
};

// 批量删除
const handleBatchDelete = async () => {
  const ids = selectedRows.value.map(item => item.id);
  await batchDeleteUsers(ids);
  loadData();
  selectedRows.value = [];
};

const handleBatchUpdate = async () => {
  const ids = selectedRows.value.map(item => item.id);
  await batchDeleteUsers(ids);
  if (timer) {
    clearInterval(timer);
  } else {
    timer = setInterval(loadData, 5 * 60);
  }
  selectedRows.value = [];
};

// 初始化加载
onMounted(() => {
  loadData();
});

onBeforeUnMount(() => {
  clearInterval(timer);
});
</script>

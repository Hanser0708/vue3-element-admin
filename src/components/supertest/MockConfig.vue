<template>
  <el-dialog v-model="visible" title="创建配置" width="600px">
    <el-form
      ref="formRef"
      :model="form"
      :rules="rules"
      label-width="120px"
      label-position="right"
    >
      <!-- 名称 -->
      <el-form-item label="名称" prop="name">
        <el-input v-model="form.name" placeholder="请输入名称" />
      </el-form-item>

      <!-- 安装包 -->
      <el-form-item label="安装包" prop="packageId">
        <el-select
          v-model="form.packageId"
          filterable
          remote
          :remote-method="searchPackages"
          placeholder="请搜索安装包"
          @change="handlePackageChange"
        >
          <el-option
            v-for="item in packageOptions"
            :key="item.id"
            :label="item.name"
            :value="item.id"
          />
        </el-select>
      </el-form-item>

      <!-- 产品名称 -->
      <el-form-item label="产品名称" prop="productId">
        <el-select
          v-model="form.productId"
          :disabled="!form.packageId"
          placeholder="请先选择安装包"
          @change="handleProductChange"
        >
          <el-option
            v-for="product in productOptions"
            :key="product.id"
            :label="product.name"
            :value="product.id"
          />
        </el-select>
      </el-form-item>

      <!-- 版本 -->
      <el-form-item label="版本" prop="versionId">
        <el-select
          v-model="form.versionId"
          :disabled="!form.productId"
          placeholder="请先选择产品"
        >
          <el-option
            v-for="version in versionOptions"
            :key="version.id"
            :label="version.name"
            :value="version.id"
          />
        </el-select>
      </el-form-item>

      <!-- VM IP -->
      <el-form-item label="VM IP" prop="vmIp">
        <el-select
          v-model="form.vmIp"
          filterable
          remote
          :remote-method="searchVmIps"
          :disabled="!form.versionId"
          placeholder="请输入关键词搜索"
        >
          <el-option
            v-for="ip in vmIpOptions"
            :key="ip"
            :label="ip"
            :value="ip"
          />
        </el-select>
      </el-form-item>

      <!-- 对接IP -->
      <el-form-item label="对接IP" prop="connectIp">
        <el-select
          v-model="form.connectIp"
          :disabled="!form.vmIp"
          placeholder="请先选择VM IP"
        >
          <el-option
            v-for="ip in connectIpOptions"
            :key="ip"
            :label="ip"
            :value="ip"
          />
        </el-select>
      </el-form-item>

      <!-- 端口 -->
      <el-form-item label="端口" prop="port">
        <div class="flex items-center">
          <el-input v-model.number="form.port" placeholder="请输入端口号" />
          <el-button
            class="ml-2"
            :loading="checkingPort"
            @click="checkPortConnection"
          >
            检测连通性
          </el-button>
        </div>
      </el-form-item>
    </el-form>

    <template #footer>
      <el-button @click="visible = false">取消</el-button>
      <el-button
        type="primary"
        :disabled="!formValid"
        @click="handleSubmit"
      >
        确定
      </el-button>
    </template>
  </el-dialog>
</template>

<script setup lang="ts">
import { ref, computed, watch } from "vue";
import type { FormInstance, FormRules } from "element-plus";
import axios from "axios";

interface Package {
  id: string
  name: string
}

interface Product {
  id: string
  name: string
}

interface Version {
  id: string
  name: string
}

const visible = ref(false);
const formRef = ref<FormInstance>();
const checkingPort = ref(false);

const form = ref({
  name: "",
  packageId: "",
  productId: "",
  versionId: "",
  vmIp: "",
  connectIp: "",
  port: null as number | null,
});

// 下拉选项
const packageOptions = ref<Package[]>([]);
const productOptions = ref<Product[]>([]);
const versionOptions = ref<Version[]>([]);
const vmIpOptions = ref<string[]>([]);
const connectIpOptions = ref<string[]>([]);

// 表单校验规则
const rules = ref<FormRules>({
  name: [{ required: true, message: "请输入名称", trigger: "blur" }],
  packageId: [{ required: true, message: "请选择安装包", trigger: "change" }],
  productId: [{ required: true, message: "请选择产品", trigger: "change" }],
  versionId: [{ required: true, message: "请选择版本", trigger: "change" }],
  vmIp: [{ required: true, message: "请选择VM IP", trigger: "change" }],
  connectIp: [{ required: true, message: "请选择对接IP", trigger: "change" }],
  port: [
    { required: true, message: "请输入端口号" },
    { type: "number", min: 1, max: 65535, message: "端口号范围1-65535" },
  ],
});

// 表单是否有效
const formValid = computed(() => {
  return Object.values(form.value).every(
    (value) => value !== "" && value !== null && value !== undefined,
  );
});

// 搜索安装包
const searchPackages = async (query: string) => {
  if (!query) return;
  const { data } = await axios.get("/api/packages", { params: { query } });
  packageOptions.value = data;
};

// 安装包变化处理
const handlePackageChange = async (packageId: string) => {
  form.value.productId = "";
  form.value.versionId = "";
  if (packageId) {
    const { data } = await axios.get(`/api/products?packageId=${packageId}`);
    productOptions.value = data;
  }
};

// 产品变化处理
const handleProductChange = async (productId: string) => {
  form.value.versionId = "";
  if (productId) {
    const { data } = await axios.get(`/api/versions?productId=${productId}`);
    versionOptions.value = data;
  }
};

// 搜索VM IP
const searchVmIps = async (query: string) => {
  if (!form.value.versionId || !query) return;
  const params = {
    versionId: form.value.versionId,
    query,
  };
  const { data } = await axios.get("/api/vm-ips", { params });
  vmIpOptions.value = data;
};

// 检测端口连通性
const checkPortConnection = async () => {
  if (!form.value.port) return;
  checkingPort.value = true;
  try {
    await axios.post("/api/check-port", {
      ip: form.value.vmIp,
      port: form.value.port,
    });
    ElMessage.success("端口连通正常");
  } catch {
    ElMessage.error("端口连接失败");
  } finally {
    checkingPort.value = false;
  }
};

// 提交表单
const handleSubmit = async () => {
  try {
    await formRef.value?.validate();
    await axios.post("/api/create", form.value);
    visible.value = false;
    ElMessage.success("创建成功");
    // 这里可以emit事件通知父组件
  } catch (error) {
    console.error("表单验证失败", error);
  }
};

// 打开弹窗时重置表单
watch(visible, (val) => {
  if (val) {
    formRef.value?.resetFields();
    packageOptions.value = [];
    productOptions.value = [];
    versionOptions.value = [];
    vmIpOptions.value = [];
    connectIpOptions.value = [];
  }
});

// 暴露打开方法
const open = () => {
  visible.value = true;
};

defineExpose({ open });
</script>

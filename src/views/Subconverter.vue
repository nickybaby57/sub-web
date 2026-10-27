<template>
  <div>
    <el-row style="margin-top: 10px">
      <el-col>
        <el-card>
          <template #header>
            Subscription Converter
            <!-- 已移除第三方 GitHub 推广图标 -->
            <div style="display: inline-block; position:absolute; right: 20px">{{ backendVersion }}</div>
          </template>
          <el-container>
            <el-form :model="form" label-width="140px" label-position="left" style="width: 100%">
              <el-form-item label="模式设置:">
                <el-radio v-model="advanced" value="1">基础模式</el-radio>
                <el-radio v-model="advanced" value="2">进阶模式</el-radio>
              </el-form-item>
              <el-form-item label="订阅链接:">
                <el-input v-model="form.sourceSubUrl" type="textarea" :rows="3"
                  placeholder="支持订阅或ss/ssr/vmess链接，多个链接每行一个或用 | 分隔" @blur="saveSubUrl" />
              </el-form-item>
              <el-form-item label="客户端:">
                <el-select v-model="form.clientType" style="width: 100%">
                  <el-option v-for="(v, k) in options.clientTypes" :key="k" :label="k" :value="v"></el-option>
                </el-select>
              </el-form-item>

              <div v-if="advanced === '2'">
                <el-form-item label="后端地址:">
                  <!-- 已移除前往项目仓库按钮，占位符已修改 -->
                  <el-autocomplete style="width: 100%" v-model="form.customBackend" :fetch-suggestions="backendSearch"
                    placeholder="输入本地私有后端服务地址。例：http://127.0.0.1:25500/sub?">
                  </el-autocomplete>
                </el-form-item>
                <el-form-item label="远程配置:">
                  <el-select v-model="form.remoteConfig" allow-create filterable placeholder="请选择" style="width: 100%">
                    <el-option-group v-for="group in options.remoteConfig" :key="group.label" :label="group.label">
                      <el-option v-for="item in group.options" :key="item.value" :label="item.label"
                        :value="item.value"></el-option>
                    </el-option-group>
                  </el-select>
                </el-form-item>
                <el-form-item label="Include:">
                  <el-input v-model="form.includeRemarks" placeholder="节点名包含的关键字，支持正则" />
                </el-form-item>
                <el-form-item label="Exclude:">
                  <el-input v-model="form.excludeRemarks" placeholder="节点名不包含的关键字，支持正则" />
                </el-form-item>
                <el-form-item label="FileName:">
                  <el-input v-model="form.filename" placeholder="返回的订阅文件名" />
                </el-form-item>

                <el-form-item v-for="(param, i) in customParams" :key="i">
                  <template #label>
                    <el-input v-model="param.name" placeholder="自定义参数名">
                      <template #suffix>
                        <div style="width: 10px;">:</div>
                      </template>
                    </el-input>
                  </template>
                  <el-input v-model="param.value" placeholder="自定义参数内容">
                    <template #suffix>
                      <el-button link :icon="Delete" style="margin-right: 5px" @click="customParams.splice(i, 1)" />
                    </template>
                  </el-input>
                </el-form-item>

                <el-form-item label-width="0px">
                  <el-row class="options-row">
                    <el-col>
                      <el-checkbox v-model="form.nodeList" border>输出为 Node List</el-checkbox>
                    </el-col>
                    <el-popover placement="bottom" trigger="click" v-model:visible="form.extraset">
                      <template #reference>
                        <el-button>更多选项</el-button>
                      </template>
                      <el-row>
                        <el-checkbox v-model="form.emoji">Emoji</el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.scv">跳过证书验证</el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox :model-value="form.udp === true" @change="form.udp = $event">启用 UDP</el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.appendType">节点类型</el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.sort">排序节点</el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.fdn">过滤非法节点</el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.expand">规则展开</el-checkbox>
                      </el-row>
                    </el-popover>
                    <el-popover placement="bottom" trigger="click">
                      <template #reference>
                        <el-button style="margin-left: 10px">定制功能</el-button>
                      </template>
                      <el-row>
                        <el-checkbox v-model="form.tpl.surge.doh">Surge.DoH</el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.tpl.clash.doh">Clash.DoH</el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.insert">网易云</el-checkbox>
                      </el-row>
                    </el-popover>
                    <el-popover placement="top-end" title="添加自定义转换参数" trigger="hover">
                      <template #reference>
                        <el-button @click="addCustomParam" style="margin-left: 10px">
                          <el-icon><Plus /></el-icon>
                        </el-button>
                      </template>
                      <el-link type="primary" :href="subDocAdvanced" target="_blank" :icon="InfoFilled">参考文档</el-link>
                    </el-popover>
                  </el-row>
                </el-form-item>
              </div>

              <div style="margin-top: 50px"></div>

              <el-divider content-position="center">
                <el-icon><MagicStick /></el-icon>
              </el-divider>

              <el-form-item label="定制订阅:">
                <el-input class="copy-content" disabled :model-value="customSubUrl">
                  <template #append>
                    <el-button :icon="DocumentCopy" @click="copyToClipboard(customSubUrl)">复制</el-button>
                  </template>
                </el-input>
              </el-form-item>
              <!-- 已移除“订阅短链”输入框 -->

              <!-- 操作按钮组 -->
              <el-form-item class="actions-row" label-width="0px" style="margin-top: 40px">
                <el-button
                  :style="{ width: '290px' }"
                  type="danger"
                  @click="makeUrlClick"
                  :disabled="!canGenerateUrl">
                  生成订阅链接
                </el-button>
                <!-- 已移除“生成短链接”按钮 -->
              </el-form-item>

              <el-form-item class="actions-row" label-width="0px">
                <el-button
                  :style="buttonStyle"
                  type="primary"
                  @click="dialogUploadConfigVisible = true"
                  :icon="UploadFilled"
                  :loading="loading">
                  上传配置
                </el-button>
                <el-button
                  :style="buttonStyle"
                  type="primary"
                  @click="clashInstall"
                  :icon="Connection"
                  :disabled="!canImportClash">
                  一键导入 Clash
                </el-button>
              </el-form-item>
              <!-- 已移除依赖第三方 API 的“从 URL 解析”功能 -->
            </el-form>
          </el-container>
        </el-card>
      </el-col>
    </el-row>

    <!-- 配置上传对话框 -->
    <ConfigUploadDialog
      v-model:visible="dialogUploadConfigVisible"
      :upload-config="uploadConfig"
      :result-url="uploadResultUrl"
      :loading="uploading"
      @cancel="handleUploadCancel"
      @confirm="handleConfigUpload"
    />
    <!-- 已移除 UrlParseDialog 对话框 -->
  </div>
</template>

<script>
import { h } from 'vue';
import {
  Connection,
  Delete,
  DocumentCopy,
  InfoFilled,
  MagicStick,
  Plus,
  UploadFilled
} from '@element-plus/icons-vue';

// 导入配置
import { CONSTANTS } from '@/config/constants';
import { CLIENT_TYPES } from '@/config/client-types';
import { REMOTE_CONFIGS } from '@/config/remote-configs';

// 导入Composables
import { useSubscriptionForm, addCustomParam, saveSubUrl as saveSubscriptionUrl } from '@/composables/useSubscriptionForm';
import { useSubscription } from '@/composables/useSubscription';
import { useGeneratedLinks } from '@/composables/useGeneratedLinks';

// 导入工具函数
import { getLocalStorageItem } from '@/utils/storage';
import { copyText } from '@/utils/clipboard';
import { formatErrorMessage } from '@/utils/formatters';

// 导入服务 (移除了短链接相关的 ShortUrlService)
import { BackendService } from '@/services/backendService';
import { ConfigUploadService } from '@/services/configUploadService';

// 导入组件 (移除了 UrlParseDialog)
import ConfigUploadDialog from '@/components/ConfigUploadDialog.vue';

export default {
  name: 'Subconverter',
  components: {
    ConfigUploadDialog,
    MagicStick,
    Plus
  },
  data() {
    const subscriptionForm = useSubscriptionForm();
    // 使用 Vite 环境变量，若未配置则回退到默认本地端口
    const defaultBackend = import.meta.env.VITE_APP_SUBCONVERTER_DEFAULT_BACKEND || "http://127.0.0.1:25500/sub?";

    return {
      // 配置选项
      options: {
        clientTypes: CLIENT_TYPES,
        backendOptions: [{ value: defaultBackend }],
        remoteConfig: REMOTE_CONFIGS
      },

      // 状态
      backendVersion: "",
      generatedLinks: useGeneratedLinks(),
      uploading: false,
      dialogUploadConfigVisible: false,
      uploadConfig: "",
      uploadResultUrl: "",
      subDocAdvanced: CONSTANTS.DOC_ADVANCED,

      // 是否为 PC 端
      isPC: true,

      // 合并表单状态
      ...subscriptionForm
    };
  },
  computed: {
    Connection() { return Connection; },
    Delete() { return Delete; },
    DocumentCopy() { return DocumentCopy; },
    InfoFilled() { return InfoFilled; },
    UploadFilled() { return UploadFilled; },

    buttonStyle() {
      return { width: '140px' };
    },

    canGenerateUrl() {
      return this.form.sourceSubUrl.length > 0 && this.form.clientType;
    },

    canImportClash() {
      return this.customSubUrl.length > 0;
    },

    customSubUrl() {
      return this.generatedLinks.longUrl;
    },

    loading() {
      return this.uploading || this.generatedLinks.pending;
    }
  },
  watch: {
    dialogUploadConfigVisible(visible) {
      if (!visible) {
        this.uploadResultUrl = "";
      }
    }
  },
  created() {
    document.title = "Subscription Converter";
    this.isPC = this.$getOS().isPc;

    if (import.meta.env.VITE_USE_STORAGE === 'true') {
      const cachedUrl = getLocalStorageItem('sourceSubUrl');
      if (cachedUrl) {
        this.form.sourceSubUrl = cachedUrl;
      }
    }
  },
  mounted() {
    this.form.clientType = CONSTANTS.DEFAULT_CLIENT_TYPE;
    // 初始化后端地址为环境变量
    if (!this.form.customBackend) {
      this.form.customBackend = import.meta.env.VITE_APP_SUBCONVERTER_DEFAULT_BACKEND || "http://127.0.0.1:25500/sub?";
    }
    
    this.getBackendVersion();

    this.notifyTimer = setTimeout(() => {
      this.notify();
    }, 1000);
  },
  beforeUnmount() {
    clearTimeout(this.notifyTimer);
  },
  methods: {
    async copyToClipboard(text, successMessage = "Copied!") {
      if (!text) {
        return false;
      }
      const copied = await copyText(text);
      if (copied) {
        this.$message.success(successMessage);
      } else {
        this.$message.error("复制失败，请手动选中链接复制");
      }
      return copied;
    },

    clashInstall() {
      if (this.customSubUrl === "") {
        this.$message.error("请先填写必填项，生成订阅链接");
        return false;
      }
      window.open('clash://install-config?url=' + encodeURIComponent(this.generatedLinks.importUrl));
    },

    makeUrlClick() {
      const url = this.makeUrl(this.form, this.advanced, this.customParams);
      if (url) {
        this.generatedLinks.setLongUrl(url);
        this.copyToClipboard(this.customSubUrl, "定制订阅已复制到剪贴板");
      } else {
        this.$message.error("订阅链接与客户端为必填项");
      }
    },

    async confirmUploadConfig() {
      if (this.uploading) return;
      if (this.uploadConfig.trim() === '') {
        this.$message.warning('远程配置不能为空');
        return false;
      }
      this.uploading = true;
      try {
        const result = await ConfigUploadService.uploadConfig(this.$axios, this.uploadConfig, copyText);
        this.form.remoteConfig = result.url;
        if (result.copied) {
          this.$message.success('远程配置上传成功，配置链接已复制到剪贴板，有效期三个月望知悉');
          this.dialogUploadConfigVisible = false;
          this.uploadConfig = '';
          this.uploadResultUrl = '';
        } else {
          this.$message.error('复制失败，请手动选中链接复制');
          this.uploadResultUrl = result.url;
        }
      } catch (error) {
        this.$message.error('远程配置上传失败: ' + formatErrorMessage(error));
      } finally {
        this.uploading = false;
      }
    },

    handleUploadCancel() {
      if (this.uploading) return;
      this.uploadConfig = "";
      this.uploadResultUrl = "";
      this.dialogUploadConfigVisible = false;
    },

    handleConfigUpload(configContent) {
      if (this.uploading) return;
      this.uploadConfig = configContent;
      return this.confirmUploadConfig();
    },

    backendSearch(queryString, cb) {
      const results = this.backendSearchSuggestions(queryString, this.options.backendOptions);
      cb(results);
    },

    backendSearchSuggestions(queryString, backends) {
      if (queryString) {
        return backends.filter(backend => {
          return backend.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0;
        });
      }
      return backends;
    },

    async getBackendVersion() {
      this.backendVersion = await BackendService.getBackendVersion(this.$axios);
    },

    notify() {
      this.$notify({
        title: "隐私提示",
        type: "success",
        message: h(
          "i",
          { style: "color: teal" },
          "当前已启用完全本地化模式。已移除所有短链接与第三方API调用，确保您的订阅数据绝对安全。"
        )
      });
    },

    saveSubUrl() {
      saveSubscriptionUrl(this.form);
    },

    addCustomParam() {
      addCustomParam(this.customParams);
    },

    ...useSubscription(CONSTANTS.DEFAULT_BACKEND)
  }
};
</script>

<style scoped>
.actions-row :deep(.el-form-item__content) {
  justify-content: center;
}
.options-row {
  width: 100%;
  flex-wrap: nowrap;
}
.options-row :deep(.el-col) {
  flex: 1;
}
</style>

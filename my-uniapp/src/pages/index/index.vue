<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from "vue";

// 本地状态
const deviceToken = ref("");
const initStatus = ref<"initializing" | "initialized" | "failed">("initializing");
const messageLogs = ref<
  Array<{
    time: string;
    type: string;
    summary: string;
    data: any;
  }>
>([]);

// 状态文本
const statusText = computed(() => {
  switch (initStatus.value) {
    case "initializing":
      return "初始化中...";
    case "initialized":
      return "初始化成功";
    case "failed":
      return "初始化失败";
    default:
      return "未知状态";
  }
});

// 状态样式
const statusClass = computed(() => {
  switch (initStatus.value) {
    case "initialized":
      return "status-success";
    case "failed":
      return "status-error";
    default:
      return "status-normal";
  }
});

// 页面加载
onMounted(() => {
  console.log("=== Index 页面加载 ===");
  
  // 监听 device_token 事件
  uni.$on('umeng_device_token', (data: any) => {
    console.log("[index] 收到 device_token:", data);
    if (data.token) {
      deviceToken.value = data.token;
      initStatus.value = "initialized";
      uni.showToast({ title: "初始化成功", icon: "success" });
    }
  });
  
  // 5秒后检查是否收到 device_token
  setTimeout(() => {
    console.log("[index] 检查初始化状态:", deviceToken.value ? "成功" : "超时");
    if (!deviceToken.value && initStatus.value === "initializing") {
      initStatus.value = "failed";
      uni.showToast({ title: "初始化超时", icon: "none" });
    }
  }, 5000);
});

// 页面卸载
onUnmounted(() => {
  uni.$off('umeng_device_token');
});

// 复制 DeviceToken
const copyDeviceToken = () => {
  if (!deviceToken.value) return;
  uni.setClipboardData({
    data: deviceToken.value,
    success: () => {
      uni.showToast({ title: "已复制到剪贴板", icon: "success" });
    },
  });
};

// 发送本地通知
const sendLocalNotification = () => {
  // #ifdef APP-PLUS
  plus.push.createMessage(
    "这是一条本地测试通知消息",
    JSON.stringify({ type: "test", time: new Date().toISOString() }),
    { title: "本地通知测试", icon: "static/logo.png" }
  );
  addLog("本地通知", { action: "send_local" });
  // #endif
  // #ifndef APP-PLUS
  uni.showToast({ title: "仅在 App 环境有效", icon: "none" });
  // #endif
};

// 添加日志
const addLog = (type: string, data: any) => {
  const now = new Date();
  const timeStr = `${now.getHours().toString().padStart(2, "0")}:${now.getMinutes().toString().padStart(2, "0")}:${now.getSeconds().toString().padStart(2, "0")}`;

  let summary = "";
  if (data.payload) {
    const payload = data.payload;
    if (payload.body) {
      summary = payload.body.title || payload.body.text || JSON.stringify(payload.body).substring(0, 50);
    } else {
      summary = JSON.stringify(payload).substring(0, 50);
    }
  } else {
    summary = JSON.stringify(data).substring(0, 50);
  }

  messageLogs.value.unshift({ time: timeStr, type, summary, data });
  if (messageLogs.value.length > 50) {
    messageLogs.value = messageLogs.value.slice(0, 50);
  }
};

// 清空日志
const clearLogs = () => {
  messageLogs.value = [];
};

// 显示日志详情
const showLogDetail = (log: any) => {
  uni.showModal({
    title: `${log.type} - ${log.time}`,
    content: JSON.stringify(log.data, null, 2).substring(0, 500),
    showCancel: false,
  });
};

// 显示调试信息
const showDebugInfo = () => {
  // #ifdef APP-PLUS
  const info = {
    deviceToken: deviceToken.value,
    initStatus: initStatus.value,
    platform: uni.getSystemInfoSync().platform,
  };
  uni.showModal({
    title: "调试信息",
    content: JSON.stringify(info, null, 2),
    showCancel: false,
  });
  // #endif
  // #ifndef APP-PLUS
  uni.showToast({ title: "仅在 App 环境有效", icon: "none" });
  // #endif
};
</script>

<template>
  <view class="content">
    <image class="logo" src="/static/logo.png" />
    <view class="text-area">
      <text class="title">友盟推送演示</text>
    </view>

    <!-- 初始化状态提示 -->
    <view class="status-section">
      <view class="status-item">
        <text class="status-label">初始化状态:</text>
        <text class="status-value" :class="statusClass">{{ statusText }}</text>
      </view>
    </view>

    <!-- 设备信息区 -->
    <view class="info-section">
      <view class="info-item">
        <text class="info-label">设备 Token:</text>
        <text class="info-value token-text" selectable>{{ deviceToken || "未获取" }}</text>
      </view>
    </view>

    <!-- 操作按钮区 -->
    <view class="button-section">
      <button class="action-btn primary" @click="copyDeviceToken" :disabled="!deviceToken">
        复制 DeviceToken
      </button>
      <button class="action-btn" @click="sendLocalNotification">发送本地通知</button>
      <button class="action-btn" @click="showDebugInfo">显示调试信息</button>
    </view>

    <!-- 消息日志区 -->
    <view class="log-section" v-if="messageLogs.length > 0">
      <view class="log-title">消息日志 ({{ messageLogs.length }})</view>
      <scroll-view class="log-list" scroll-y>
        <view class="log-item" v-for="(log, index) in messageLogs" :key="index" @click="showLogDetail(log)">
          <text class="log-time">{{ log.time }}</text>
          <text class="log-type">[{{ log.type }}]</text>
          <text class="log-content">{{ log.summary }}</text>
        </view>
      </scroll-view>
      <button class="clear-btn" @click="clearLogs">清空日志</button>
    </view>

    <!-- 使用说明 -->
    <view class="tips-section">
      <view class="tips-title">使用说明</view>
      <view class="tips-content">
        <text class="tips-item">1. DeviceToken 用于服务端推送定位设备</text>
        <text class="tips-item">2. 支持华为、小米、魅族、vivo、OPPO 离线推送</text>
        <text class="tips-item">3. 点击"复制 DeviceToken"在友盟后台测试推送</text>
      </view>
    </view>
  </view>
</template>

<style>
.content {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40rpx;
  min-height: 100vh;
  background: #f5f7fa;
}

.logo {
  height: 150rpx;
  width: 150rpx;
  margin-top: 40rpx;
  margin-bottom: 30rpx;
}

.title {
  font-size: 40rpx;
  font-weight: bold;
  color: #333;
}

.status-section {
  width: 100%;
  background: #fff;
  border-radius: 16rpx;
  padding: 30rpx;
  margin: 30rpx 0;
  box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.05);
}

.status-item {
  display: flex;
  align-items: center;
}

.status-label {
  font-size: 28rpx;
  color: #666;
  margin-right: 20rpx;
}

.status-value {
  font-size: 28rpx;
  font-weight: bold;
}

.status-success { color: #4caf50; }
.status-error { color: #f44336; }
.status-normal { color: #666; }

.info-section {
  width: 100%;
  background: #fff;
  border-radius: 16rpx;
  padding: 30rpx;
  margin-bottom: 30rpx;
  box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.05);
}

.info-item {
  display: flex;
  align-items: flex-start;
}

.info-label {
  width: 160rpx;
  font-size: 28rpx;
  color: #666;
  flex-shrink: 0;
}

.info-value {
  flex: 1;
  font-size: 26rpx;
  color: #333;
  word-break: break-all;
}

.token-text {
  font-family: monospace;
  background: #f5f5f5;
  padding: 10rpx;
  border-radius: 8rpx;
  font-size: 22rpx;
}

.button-section {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 20rpx;
  margin-bottom: 30rpx;
}

.action-btn {
  width: 100%;
  height: 90rpx;
  background: #fff;
  color: #667eea;
  border-radius: 45rpx;
  font-size: 30rpx;
  font-weight: bold;
  border: 2rpx solid #667eea;
}

.action-btn.primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  border: none;
}

.action-btn[disabled] { opacity: 0.5; }

.log-section {
  width: 100%;
  background: #fff;
  border-radius: 16rpx;
  padding: 30rpx;
  margin-bottom: 30rpx;
  box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.05);
}

.log-title {
  font-size: 30rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 20rpx;
}

.log-list {
  max-height: 400rpx;
  background: #f8f9fa;
  border-radius: 12rpx;
  padding: 20rpx;
}

.log-item {
  padding: 16rpx 0;
  border-bottom: 1rpx solid #eee;
}

.log-item:last-child { border-bottom: none; }

.log-time { font-size: 24rpx; color: #999; margin-right: 10rpx; }
.log-type { font-size: 24rpx; color: #667eea; margin-right: 10rpx; font-weight: bold; }
.log-content { font-size: 26rpx; color: #333; }

.clear-btn {
  margin-top: 20rpx;
  width: 100%;
  height: 70rpx;
  background: #f5f5f5;
  color: #999;
  border-radius: 35rpx;
  font-size: 26rpx;
  border: none;
}

.tips-section {
  width: 100%;
  background: #e3f2fd;
  border-radius: 16rpx;
  padding: 30rpx;
  border: 2rpx solid #2196f3;
}

.tips-title {
  font-size: 30rpx;
  font-weight: bold;
  color: #1976d2;
  margin-bottom: 20rpx;
}

.tips-content {
  display: flex;
  flex-direction: column;
  gap: 12rpx;
}

.tips-item {
  font-size: 26rpx;
  color: #424242;
  line-height: 1.5;
}
</style>

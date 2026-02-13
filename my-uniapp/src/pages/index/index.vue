<template>
  <view class="content">
    <image class="logo" src="/static/logo.png" />
    <view class="text-area">
      <text class="title">{{ title }}</text>
    </view>
    <view class="button-area">
      <button class="notify-btn" @click="sendNotification">发送系统通知</button>
    </view>
    <view class="info-area">
      <text class="info-text">{{ status }}</text>
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

const title = ref('Hello uni-app')
const status = ref('点击按钮发送通知')

// 发送本地通知
const sendNotification = () => {
  // 检查是否在 App 环境
  // #ifdef APP-PLUS
  // 使用 5+ API 发送本地消息通知
  plus.push.createMessage('这是一条测试通知消息', '通知payload', {
    title: '系统通知',
    icon: 'static/logo.png'
  })
  status.value = '已发送本地通知'
  // #endif
  
  // #ifdef H5
  // H5 环境下使用浏览器通知 API
  if ('Notification' in window) {
    Notification.requestPermission().then(permission => {
      if (permission === 'granted') {
        new Notification('系统通知', {
          body: '这是一条测试通知消息',
          icon: '/static/logo.png'
        })
        status.value = '已发送浏览器通知'
      } else {
        status.value = '通知权限被拒绝'
        uni.showToast({
          title: '请允许通知权限',
          icon: 'none'
        })
      }
    })
  } else {
    uni.showToast({
      title: 'H5 不支持系统通知',
      icon: 'none'
    })
  }
  // #endif
  
  // #ifdef MP
  // 小程序环境下使用微信订阅消息
  uni.showModal({
    title: '提示',
    content: '小程序需要通过服务端发送订阅消息',
    showCancel: false
  })
  status.value = '小程序请使用订阅消息'
  // #endif
}

// 页面加载时检查推送权限
onMounted(() => {
  // #ifdef APP-PLUS
  // 检查推送权限
  plus.push.getClientInfoAsync((info) => {
    console.log('推送客户端信息:', info)
  })
  // #endif
})
</script>

<style>
.content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40rpx;
}

.logo {
  height: 200rpx;
  width: 200rpx;
  margin-top: 100rpx;
  margin-left: auto;
  margin-right: auto;
  margin-bottom: 50rpx;
}

.text-area {
  display: flex;
  justify-content: center;
  margin-bottom: 60rpx;
}

.title {
  font-size: 36rpx;
  color: #333;
  font-weight: bold;
}

.button-area {
  width: 100%;
  display: flex;
  justify-content: center;
}

.notify-btn {
  width: 80%;
  height: 90rpx;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  border-radius: 45rpx;
  font-size: 32rpx;
  font-weight: bold;
  border: none;
  box-shadow: 0 8rpx 20rpx rgba(102, 126, 234, 0.4);
  transition: all 0.3s ease;
}

.notify-btn:active {
  transform: scale(0.98);
  box-shadow: 0 4rpx 10rpx rgba(102, 126, 234, 0.3);
}

.info-area {
  margin-top: 40rpx;
}

.info-text {
  font-size: 28rpx;
  color: #999;
}
</style>

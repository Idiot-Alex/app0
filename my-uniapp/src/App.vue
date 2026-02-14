<script setup lang="ts">
import { onLaunch, onShow, onHide } from "@dcloudio/uni-app";

onLaunch(() => {
  console.log("=== App Launch ===");
  
  // #ifdef APP-PLUS
  console.log("APP-PLUS 环境，开始初始化友盟推送");
  
  try {
    // 引入新插件 spx-umpush
    const UmPush = uni.requireNativePlugin("spx-umpush");
    console.log("UmPush 插件对象:", UmPush);
    
    if (!UmPush) {
      console.error("友盟推送插件未找到");
      return;
    }

    // 先监听消息（在初始化之前调用）
    UmPush.onPushMessage((res: any) => {
      console.log("[UmPush] 收到消息:", JSON.stringify(res));
      // 转发到页面
      uni.$emit('umeng_push_message', res);
    });

    // 初始化推送
    console.log("[UmPush] 开始初始化推送");
    UmPush.initUmPush((res: any) => {
      console.log("[UmPush] 初始化回调:", JSON.stringify(res));
      // code: 0 表示成功
      if (res.code === 0 && res.deviceToken) {
        console.log("[UmPush] 获取到 deviceToken:", res.deviceToken);
        uni.$emit('umeng_device_token', { token: res.deviceToken });
      }
    });
    
  } catch (e) {
    console.error("[UmPush] 初始化异常:", e);
  }
  
  // #endif
});

onShow(() => {
  console.log("=== App Show ===");
});

onHide(() => {
  console.log("=== App Hide ===");
});
</script>

<style>
</style>

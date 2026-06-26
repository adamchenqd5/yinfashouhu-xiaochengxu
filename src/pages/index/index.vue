<template>
  <view class="page">
    <view class="container">
      <view class="shield-icon">🛡️</view>
      <text class="title">银发守护者</text>
      <text class="subtitle">老年人防诈骗聊天模拟器</text>

      <view class="rules">
        <text class="rules-title">📋 游戏规则</text>
        <view class="rule-item">
          <text class="rule-bullet">▸</text>
          <text class="rule-text">💰 守护钱包 — 初始100元，一分钱都不能被骗走</text>
        </view>
        <view class="rule-item">
          <text class="rule-bullet">▸</text>
          <text class="rule-text">🔍 识破套路 — 每轮选择最安全的回应方式</text>
        </view>
        <view class="rule-item">
          <text class="rule-bullet">▸</text>
          <text class="rule-text">🏆 十轮对话 — 识破越多越好</text>
        </view>
        <view class="rule-item">
          <text class="rule-bullet">▸</text>
          <text class="rule-text">⚠️ 钱包归零或识破数为0则游戏失败</text>
        </view>
      </view>

      <button class="start-btn" @click="startGame">开始挑战</button>

      <text class="warning">⚠️ 本游戏基于真实诈骗案例改编，仅供反诈教育用途。记住核心原则：不轻信、不贪利、不转账！</text>
    </view>
  </view>
</template>

<script>
export default {
  methods: {
    startGame() {
      uni.navigateTo({ url: '/pages/select/select' })
    }
  }
}
</script>

<style scoped>
.page {
  min-height: 100vh;
  background: linear-gradient(180deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 60rpx 40rpx;
}

.container {
  text-align: center;
  max-width: 660rpx;
  width: 100%;
}

.shield-icon {
  font-size: 140rpx;
  margin-bottom: 40rpx;
}

.title {
  color: #ffffff;
  font-size: 60rpx;
  font-weight: 700;
  display: block;
  margin-bottom: 20rpx;
  letter-spacing: 6rpx;
}

.subtitle {
  color: #FF8C50;
  font-size: 36rpx;
  display: block;
  margin-bottom: 50rpx;
}

.rules {
  background: rgba(255,255,255,0.1);
  border: 2rpx solid rgba(255,255,255,0.15);
  border-radius: 28rpx;
  padding: 36rpx 32rpx;
  text-align: left;
  margin-bottom: 48rpx;
}

.rules-title {
  color: #ffffff;
  font-size: 36rpx;
  font-weight: 700;
  display: block;
  margin-bottom: 26rpx;
}

.rule-item {
  display: flex;
  align-items: flex-start;
  margin-bottom: 18rpx;
}

.rule-bullet {
  color: #07C160;
  font-size: 30rpx;
  margin-right: 16rpx;
  line-height: 1.8;
}

.rule-text {
  color: #dddddd;
  font-size: 32rpx;
  line-height: 1.8;
}

.start-btn {
  background: #07C160;
  color: #ffffff;
  padding: 36rpx;
  border-radius: 16rpx;
  border: none;
  font-size: 44rpx;
  font-weight: 600;
  width: 100%;
  margin-bottom: 36rpx;
}

.start-btn:active {
  background: #06AD56;
}

.warning {
  color: #aaaaaa;
  font-size: 26rpx;
  line-height: 1.8;
  display: block;
}
</style>

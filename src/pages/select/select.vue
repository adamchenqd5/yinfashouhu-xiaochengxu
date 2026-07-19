<template>
  <view class="page">
    <view class="header">
      <text class="header-title">选择诈骗场景</text>
      <text class="header-desc">每个场景都是真实常见的老年人诈骗套路</text>
    </view>

    <view class="scene-list">
      <view
        v-for="scene in scenes"
        :key="scene.id"
        class="scene-card"
        @click="startScene(scene.id)"
      >
        <view class="avatar" :style="{background: scene.bg}">
          <text>{{ scene.emoji }}</text>
        </view>
        <view class="info">
          <text class="name">{{ scene.name }}</text>
          <text class="tag">{{ scene.tag }}</text>
          <view class="danger-row">
            <text class="danger">{{ scene.df }}</text>
            <text class="arrow">›</text>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
import SCENES from '@/data/scenes.js'

export default {
  onShareAppMessage() {
    return {
      title: '银发守护者 - 这些诈骗套路你爸妈能识破吗？',
      path: '/pages/index/index'
    }
  },
  onShareTimeline() {
    return {
      title: '银发守护者 - 这些诈骗套路你爸妈能识破吗？'
    }
  },
  data() {
    return {
      scenes: SCENES
    }
  },
  methods: {
    startScene(cid) {
      uni.navigateTo({
        url: `/pages/chat/chat?cid=${cid}`
      })
    }
  }
}
</script>

<style scoped>
.page {
  min-height: 100vh;
  background: #F5F5F5;
}

.header {
  text-align: center;
  padding: 40rpx 28rpx 28rpx;
  background: #F5F5F5;
  position: sticky;
  top: 0;
  z-index: 10;
}

.header-title {
  color: #111111;
  font-size: 48rpx;
  font-weight: 700;
  display: block;
}

.header-desc {
  color: #666666;
  font-size: 30rpx;
  margin-top: 14rpx;
  display: block;
}

.scene-list {
  padding: 0;
}

.scene-card {
  background: #ffffff;
  border-bottom: 2rpx solid #E0E0E0;
  padding: 32rpx 28rpx;
  display: flex;
  align-items: center;
  gap: 24rpx;
}

.scene-card:active {
  background: #ECECEC;
}

.avatar {
  width: 100rpx;
  height: 100rpx;
  border-radius: 22rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 52rpx;
  flex-shrink: 0;
}

.info {
  flex: 1;
  text-align: left;
}

.name {
  color: #111111;
  font-size: 38rpx;
  font-weight: 600;
  display: block;
}

.tag {
  color: #666666;
  font-size: 28rpx;
  margin-top: 6rpx;
  display: block;
}

.danger-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 10rpx;
}

.danger {
  font-size: 26rpx;
  color: #E07020;
  font-weight: 500;
}

.arrow {
  color: #B0B0B0;
  font-size: 48rpx;
  font-weight: 300;
}
</style>

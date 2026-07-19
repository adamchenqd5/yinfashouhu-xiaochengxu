<template>
  <view class="page">
    <!-- 顶栏 -->
    <view class="top-bar">
      <text class="back-btn" @click="confirmBack">‹</text>
      <view class="top-avatar" :style="{background: scene.bg}">
        <text>{{ scene.emoji }}</text>
      </view>
      <view class="top-info">
        <text class="top-name">{{ scene.name }}</text>
        <text class="top-status">在线</text>
      </view>
      <text class="top-more">⋯</text>
    </view>

    <!-- 状态栏 -->
    <view class="status-bar">
      <view class="status-row">
        <text class="status-icon">💰</text>
        <view class="status-content">
          <view class="status-header">
            <text class="status-label">钱包余额</text>
            <text class="status-value" :class="walletAnim">{{ '¥' + wallet }}</text>
          </view>
          <view class="progress-bg">
            <view class="progress-fill safe" :style="{width: wallet + '%'}"></view>
          </view>
        </view>
      </view>
      <view class="status-row">
        <text class="status-icon">🔍</text>
        <view class="status-content">
          <view class="status-header">
            <text class="status-label">识破次数</text>
            <text class="status-value">{{ correct }} / {{ totalRound }}</text>
          </view>
          <view class="progress-bg">
            <view class="progress-fill detect" :style="{width: (correct/totalRound*100) + '%'}"></view>
          </view>
        </view>
      </view>
    </view>

    <!-- 钱包条 -->
    <view class="wallet-bar">
      <text class="wallet-label">💰 你的钱包</text>
      <text class="wallet-amount" :class="walletAnim">¥{{ wallet }}</text>
      <text class="wallet-round">识破 {{ correct }}/{{ totalRound }}</text>
    </view>

    <!-- 聊天区 -->
    <scroll-view class="chat-area" scroll-y :scroll-into-view="scrollToId" :scroll-with-animation="true">
      <view v-for="(msg, i) in messages" :key="i" :id="'msg-' + i">
        <!-- 对方消息 -->
        <view v-if="msg.type === 'bot'" class="chat-row">
          <view class="bot-avatar" :style="{background: scene.bg}">
            <text>{{ scene.emoji }}</text>
          </view>
          <view class="msg bubble-bot">{{ msg.text }}</view>
        </view>
        <!-- 自己消息 -->
        <view v-if="msg.type === 'player'" class="chat-row me">
          <view class="msg bubble-me">{{ msg.text }}</view>
        </view>
        <!-- 系统消息 -->
        <view v-if="msg.type === 'system'" class="chat-row sys">
          <view class="msg bubble-sys">{{ msg.text }}</view>
        </view>
      </view>
    </scroll-view>

    <!-- 选项区 -->
    <view class="options-area" v-if="currentOpts.length > 0 && !gameEnded">
      <button
        v-for="(opt, i) in currentOpts"
        :key="i"
        class="option-btn"
        :class="{ 'opt-correct': opt._selected === true && opt._correct, 'opt-wrong': opt._selected === true && !opt._correct }"
        :disabled="locked"
        @click="chooseOption(i)"
      >
        {{ opt.text }}
      </button>
    </view>

    <!-- 游戏结束弹窗 -->
    <view class="overlay" v-if="gameEnded" @click.stop>
      <view class="result-card">
        <text class="result-emoji">{{ resultEmoji }}</text>
        <text class="result-title">{{ resultTitle }}</text>
        <text class="result-desc">{{ resultDesc }}</text>

        <view class="result-summary">
          <view class="summary-row">
            <text class="summary-label">💬 对话轮数</text>
            <text class="summary-value">{{ round }} / {{ totalRound }}</text>
          </view>
          <view class="summary-row">
            <text class="summary-label">🔍 识破套路</text>
            <text class="summary-value">{{ correct }} 次</text>
          </view>
          <view class="summary-row">
            <text class="summary-label">💸 被骗金额</text>
            <text class="summary-value" :style="{color: lost > 0 ? '#D32F2F' : '#2E7D32'}">¥{{ lost }}</text>
          </view>
          <view class="summary-row">
            <text class="summary-label">💰 剩余钱包</text>
            <text class="summary-value">¥{{ wallet }}</text>
          </view>
          <view class="summary-row">
            <text class="summary-label">📊 识破效率</text>
            <text class="summary-value">{{ efficiency }}%</text>
          </view>
        </view>

        <view class="tip-box">
          <text class="tip-title">💡 防骗小贴士</text>
          <text class="tip-text">{{ scene.tip }}</text>
        </view>

        <button class="restart-btn" @click="goSelect">换个场景再挑战</button>
      </view>
    </view>
  </view>
</template>

<script>
import SCENES from '@/data/scenes.js'
import ROUNDS from '@/data/rounds.js'

export default {
  onShareAppMessage() {
    return {
      title: '银发守护者 - 我正在挑战「' + (this.scene.name || '防诈骗') + '」场景，快来帮爸妈识破骗局！',
      path: '/pages/index/index'
    }
  },
  onShareTimeline() {
    return {
      title: '银发守护者 - 帮爸妈识破骗局！'
    }
  },
  data() {
    return {
      scene: {},
      cid: '',
      wallet: 100,
      correct: 0,
      round: 0,
      totalRound: 10,
      locked: false,
      gameEnded: false,
      currentOpts: [],
      currentRound: null,
      currentBotMsg: '',
      pendingReply: '',
      messages: [],
      scrollToId: '',
      walletAnim: '',
      resultEmoji: '',
      resultTitle: '',
      resultDesc: '',
      lost: 0,
      efficiency: '0'
    }
  },

  onLoad(options) {
    this.cid = options.cid || 'health'
    const s = SCENES.find(x => x.id === this.cid)
    if (s) {
      this.scene = s
      uni.setNavigationBarTitle({ title: s.name })
    }
    this.$nextTick(() => {
      this.startRound()
    })
  },

  methods: {
    startRound() {
      const rounds = ROUNDS[this.cid]
      if (!rounds || this.round >= rounds.length) {
        this.finishGame()
        return
      }

      this.currentRound = rounds[this.round]
      this.currentBotMsg = this.currentRound.bot
      this.addMessage('bot', this.currentRound.bot)

      setTimeout(() => {
        this.currentOpts = this.currentRound.opts.map(o => ({
          ...o,
          _selected: false,
          _correct: o.c > 0
        }))
      }, 600)
    },

    chooseOption(index) {
      if (this.locked || this.gameEnded) return
      this.locked = true

      const opt = this.currentOpts[index]

      this.currentOpts[index]._selected = true
      this.addMessage('player', opt.text)

      if (opt.m < 0) {
        this.wallet += opt.m
        if (this.wallet < 0) this.wallet = 0
        this.walletAnim = 'down'
        setTimeout(() => { this.walletAnim = '' }, 800)
      }

      if (opt.c > 0) {
        this.correct += opt.c
      }

      setTimeout(() => {
        this.addMessage('system', opt.fb)

        if (opt.reply) {
          setTimeout(() => {
            this.addMessage('bot', opt.reply)

            setTimeout(() => {
              this.round++
              this.locked = false
              this.currentOpts = []

              if (this.wallet <= 0) {
                this.finishGame(true, '钱包归零', '你的钱包已经被骗空了！记住：遇到涉及钱的事，一定要先跟子女商量！')
                return
              }
              if (this.correct <= 0 && this.round > 0) {
                this.finishGame(true, '防骗意识薄弱', '你没有识破任何套路，说明还需要加强反诈学习。可以多看几遍防骗小贴士。')
                return
              }
              if (this.round >= this.totalRound || this.round >= (ROUNDS[this.cid] || []).length) {
                this.finishGame()
                return
              }

              this.startRound()
            }, 700)
          }, 800)
        }
      }, 500)
    },

    addMessage(type, text) {
      this.messages.push({ type, text })
      this.$nextTick(() => {
        this.scrollToId = 'msg-' + (this.messages.length - 1)
      })
    },

    finishGame(forced, forcedTitle, forcedDesc) {
      this.gameEnded = true
      this.locked = true
      this.currentOpts = []

      this.lost = 100 - this.wallet
      if (this.lost < 0) this.lost = 0
      this.efficiency = this.round > 0 ? Math.round(this.correct / this.round * 100) : 0

      if (forced) {
        this.resultEmoji = '💔'
        this.resultTitle = '游戏结束 — ' + (forcedTitle || '')
        this.resultDesc = forcedDesc || ''
      } else {
        if (this.wallet >= 100 && this.correct >= 8) {
          this.resultEmoji = '🏆'
          this.resultTitle = '完美防御！防骗达人！'
          this.resultDesc = '你成功识破了「' + this.scene.name + '」的所有套路，一分钱没花！你是银发守护者！'
        } else if (this.wallet >= 80 && this.correct >= 5) {
          this.resultEmoji = '✅'
          this.resultTitle = '防御成功！'
          this.resultDesc = '你在「' + this.scene.name + '」中保持了警惕，识破了' + this.correct + '次套路。做得不错！'
        } else if (this.wallet >= 50) {
          this.resultEmoji = '⚠️'
          this.resultTitle = '部分失守'
          this.resultDesc = '你被骗了¥' + this.lost + '，识破了' + this.correct + '次。还需要提高警惕！'
        } else {
          this.resultEmoji = '💔'
          this.resultTitle = '损失严重'
          this.resultDesc = '你被骗了¥' + this.lost + '。记住：不轻信、不贪利、不转账！'
        }
      }
    },

    confirmBack() {
      if (this.round > 0 && !this.gameEnded) {
        uni.showModal({
          title: '确认退出',
          content: '确定要退出当前游戏吗？进度将丢失。',
          confirmText: '确定退出',
          cancelText: '继续玩',
          success: (res) => {
            if (res.confirm) this.goSelect()
          }
        })
      } else {
        this.goSelect()
      }
    },

    goSelect() {
      uni.navigateBack({ delta: 1 })
    }
  }
}
</script>

<style scoped>
.page {
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #EDEDED;
}

/* ===== 顶栏 ===== */
.top-bar {
  background: #EDEDED;
  padding: 0 24rpx;
  display: flex;
  align-items: center;
  gap: 16rpx;
  height: 100rpx;
  border-bottom: 2rpx solid #D0D0D0;
  flex-shrink: 0;
}

.back-btn {
  font-size: 56rpx;
  color: #07C160;
  padding: 0 12rpx;
  font-weight: 300;
  line-height: 1;
}

.top-avatar {
  width: 64rpx;
  height: 64rpx;
  border-radius: 16rpx;
  font-size: 34rpx;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.top-info {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.top-name {
  font-size: 36rpx;
  font-weight: 600;
  color: #111111;
}

.top-status {
  font-size: 24rpx;
  color: #666666;
  font-weight: 400;
}

.top-more {
  font-size: 48rpx;
  color: #666666;
  padding: 0 12rpx;
}

/* ===== 状态栏 ===== */
.status-bar {
  background: #ffffff;
  padding: 20rpx 28rpx;
  display: flex;
  flex-direction: column;
  gap: 16rpx;
  border-bottom: 2rpx solid #D0D0D0;
  flex-shrink: 0;
}

.status-row {
  display: flex;
  align-items: center;
  gap: 16rpx;
}

.status-icon {
  font-size: 32rpx;
  width: 44rpx;
  text-align: center;
  flex-shrink: 0;
}

.status-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 6rpx;
}

.status-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.status-label {
  font-size: 26rpx;
  color: #555555;
}

.status-value {
  font-size: 30rpx;
  font-weight: 700;
  color: #111111;
  transition: all 0.3s;
}

.status-value.down {
  transform: scale(1.2);
  color: #D32F2F;
}

.progress-bg {
  height: 12rpx;
  border-radius: 6rpx;
  background: #E0E0E0;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  border-radius: 6rpx;
  transition: width 0.8s ease;
}

.progress-fill.safe { background: #07C160; }
.progress-fill.detect { background: #FA9D3B; }

/* ===== 钱包条 ===== */
.wallet-bar {
  background: #ffffff;
  padding: 16rpx 28rpx;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 2rpx solid #D0D0D0;
  flex-shrink: 0;
}

.wallet-label {
  font-size: 26rpx;
  color: #555555;
}

.wallet-amount {
  font-size: 38rpx;
  font-weight: 700;
  color: #111111;
  transition: all 0.3s;
}

.wallet-amount.down {
  color: #D32F2F;
  transform: scale(1.1);
}

.wallet-round {
  font-size: 24rpx;
  color: #555555;
  background: #F0F0F0;
  padding: 6rpx 16rpx;
  border-radius: 8rpx;
}

/* ===== 聊天区 ===== */
.chat-area {
  flex: 1;
  padding: 18rpx 24rpx;
  display: flex;
  flex-direction: column;
  gap: 20rpx;
  overflow-y: auto;
  background: #EDEDED;
}

.chat-row {
  display: flex;
  gap: 14rpx;
  align-items: flex-start;
  margin-bottom: 16rpx;
}

.chat-row.me {
  justify-content: flex-end;
}

.chat-row.sys {
  justify-content: center;
}

.bot-avatar {
  width: 72rpx;
  height: 72rpx;
  border-radius: 16rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 38rpx;
  flex-shrink: 0;
}

.msg {
  max-width: 72%;
  padding: 20rpx 26rpx;
  border-radius: 10rpx;
  font-size: 34rpx;
  line-height: 1.8;
  word-break: break-word;
  color: #111111;
}

.bubble-bot {
  background: #ffffff;
  align-self: flex-start;
}

.bubble-me {
  background: #95EC69;
  align-self: flex-end;
  color: #111111;
}

.bubble-sys {
  background: rgba(0,0,0,0.1);
  color: #444444;
  font-size: 28rpx;
  padding: 10rpx 22rpx;
  border-radius: 8rpx;
  text-align: center;
  max-width: 85%;
}

/* ===== 选项区 ===== */
.options-area {
  background: #F7F7F7;
  padding: 16rpx 24rpx;
  display: flex;
  flex-direction: column;
  gap: 12rpx;
  border-top: 2rpx solid #D0D0D0;
  max-height: 420rpx;
  overflow-y: auto;
  flex-shrink: 0;
}

.option-btn {
  background: #ffffff;
  border: 2rpx solid #E0E0E0;
  border-radius: 12rpx;
  padding: 24rpx 28rpx;
  font-size: 34rpx;
  color: #111111;
  text-align: left;
  line-height: 1.7;
  font-family: inherit;
  min-height: 88rpx;
}

.option-btn:active {
  background: #ECECEC;
}

.option-btn[disabled] {
  opacity: 0.5;
}

.opt-correct {
  background: #C8E6C9 !important;
  border: 3rpx solid #2E7D32 !important;
  color: #1B5E20 !important;
}

.opt-wrong {
  background: #FFCDD2 !important;
  border: 3rpx solid #D32F2F !important;
  color: #B71C1C !important;
}

/* ===== 结束弹窗 ===== */
.overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
}

.result-card {
  background: #ffffff;
  border-radius: 24rpx;
  padding: 48rpx 36rpx;
  text-align: center;
  max-width: 640rpx;
  width: 88%;
}

.result-emoji {
  font-size: 100rpx;
  display: block;
  margin-bottom: 24rpx;
}

.result-title {
  font-size: 40rpx;
  font-weight: 700;
  color: #111111;
  display: block;
  margin-bottom: 18rpx;
}

.result-desc {
  font-size: 32rpx;
  color: #444444;
  line-height: 1.8;
  display: block;
  margin-bottom: 32rpx;
}

.result-summary {
  background: #F5F5F5;
  border-radius: 16rpx;
  padding: 24rpx;
  margin-bottom: 32rpx;
  text-align: left;
}

.summary-row {
  display: flex;
  justify-content: space-between;
  padding: 10rpx 0;
  font-size: 30rpx;
}

.summary-label {
  color: #555555;
}

.summary-value {
  color: #111111;
  font-weight: 600;
}

.tip-box {
  background: #FFF8E1;
  border: 2rpx solid #FFC107;
  border-radius: 16rpx;
  padding: 24rpx;
  margin-bottom: 32rpx;
  text-align: left;
}

.tip-title {
  font-size: 30rpx;
  color: #E65100;
  font-weight: 700;
  display: block;
  margin-bottom: 12rpx;
}

.tip-text {
  font-size: 30rpx;
  color: #444444;
  line-height: 1.8;
}

.restart-btn {
  background: #07C160;
  color: #ffffff;
  padding: 32rpx;
  border-radius: 16rpx;
  border: none;
  font-size: 38rpx;
  font-weight: 600;
  width: 100%;
}

.restart-btn:active {
  background: #06AD56;
}
</style>

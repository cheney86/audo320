<template>
  <view class="content">
   <!-- <image class="logo" src="/static/logo.png"></image> -->
    <view class="text-area">
      <text class="title">{{title}}</text>
    </view>
    
    <view class="audio-player">
      <!-- 进度条和时间显示 -->
      <view class="progress-section">
        <view class="skip-container">
          <button class="skip-btn" @tap="toggleSkipMenu('backward')">
           <!-- <text class="skip-icon">↺</text> -->
            <text class="skip-text">{{ currentSkipTime }}s</text>
          </button>
          <!-- 快退时间选择菜单 -->
          <view v-if="skipMenuVisible && skipMenuDirection === 'backward'" class="skip-menu">
            <button class="skip-menu-item" @tap="setSkipTime(10, 'backward')">10s</button>
            <button class="skip-menu-item" @tap="setSkipTime(15, 'backward')">15s</button>
            <button class="skip-menu-item" @tap="setSkipTime(20, 'backward')">20s</button>
            <button class="skip-menu-item" @tap="setSkipTime(30, 'backward')">30s</button>
          </view>
        </view>
        
        <view class="progress-container">
          <text class="time-text">{{ formatTime(currentTime) }}</text>
          <view 
            class="progress-bar" 
            @click="seek"
            @touchstart="handleTouchStart"
            @touchmove="handleTouchMove"
            @touchend="handleTouchEnd"
            @mousedown="handleMouseDown"
            @mousemove="handleMouseMove"
            @mouseup="handleMouseUp"
            @mouseleave="handleMouseUp"
          >
            <view class="progress-track"></view>
            <view class="progress-fill" :style="{ width: progress + '%' }"></view>
            <view 
              class="progress-handle" 
              :style="{ left: progress + '%' }"
              @touchstart.stop="handleTouchStart"
              @mousedown.stop="handleMouseDown"
            ></view>
          </view>
          <text class="time-text">{{ formatTime(duration) }}</text>
        </view>
        
        <view class="skip-container">
          <button class="skip-btn" @tap="toggleSkipMenu('forward')">
          <!--  <text class="skip-icon">↻</text> -->
            <text class="skip-text">{{ currentSkipTime }}s</text>
          </button>
          <!-- 快进时间选择菜单 -->
          <view v-if="skipMenuVisible && skipMenuDirection === 'forward'" class="skip-menu">
            <button class="skip-menu-item" @tap="setSkipTime(10, 'forward')">10s</button>
            <button class="skip-menu-item" @tap="setSkipTime(15, 'forward')">15s</button>
            <button class="skip-menu-item" @tap="setSkipTime(20, 'forward')">20s</button>
            <button class="skip-menu-item" @tap="setSkipTime(30, 'forward')">30s</button>
          </view>
        </view>
      </view>
      
      <!-- 播放控制按钮 -->
      <view class="control-section">
      <!--  <button class="control-btn" @tap="prevTrack">
          <text class="control-icon">⏮</text>
        </button> -->
        
       <button class="play-btn" @tap="togglePlay">
          <text class="play-icon">{{ isPlaying ? '⏸' : '▶' }}</text>
        </button>
        
      <!--  <button class="control-btn" @tap="nextTrack">
          <text class="control-icon">▶</text>
        </button> -->
        
      <!--  <button class="control-btn loop-btn" :class="{ 'active': isLoop }" @tap="toggleLoop">
          <text class="control-icon">🔁</text>
        </button> -->
      </view>
    </view>
  </view>
</template>

<script>
import xixiAudio from '@/uni_modules/xixi-audio';

export default {
  components: {
    xixiAudio
  },
  data() {
    return {
      title: '语音播放插件测试',
      audioContext: null,
      isPlaying: false,
      isLoop: false,
      currentTime: 0,
      duration: 0,
      progress: 0,
      isDragging: false,
      progressBarWidth: 0,
      progressBarLeft: 0,
      updateInterval: null,
      audioUrl: 'https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3',
      currentSkipTime: 15,
      skipMenuVisible: false,
      skipMenuDirection: 'backward'
    };
  },
  mounted() {
    this.initAudio();
  },
  beforeUnmount() {
    if (this.audioContext) {
      this.audioContext.stop();
      this.audioContext.destroy();
    }
    if (this.updateInterval) {
      clearInterval(this.updateInterval);
    }
  },
  methods: {
    initAudio() {
      this.audioContext = uni.createInnerAudioContext();
      this.audioContext.src = this.audioUrl;
      
      this.audioContext.onPlay(() => {
        this.isPlaying = true;
        this.startProgressUpdate();
      });
      
      this.audioContext.onPause(() => {
        this.isPlaying = false;
      });
      
      this.audioContext.onStop(() => {
        this.isPlaying = false;
        this.currentTime = 0;
        this.progress = 0;
      });
      
      this.audioContext.onEnded(() => {
        this.isPlaying = false;
        this.currentTime = this.duration;
        this.progress = 100;
        if (this.isLoop) {
          this.audioContext.seek(0);
          this.audioContext.play();
        }
      });
      
      this.audioContext.onError((res) => {
        console.error('音频播放错误:', res || '未知错误');
      });
      
      this.audioContext.onCanplay(() => {
        this.duration = this.audioContext.duration || 0;
      });
    },
    startProgressUpdate() {
      this.updateInterval = setInterval(() => {
        if (this.audioContext && !this.isDragging) {
          this.currentTime = this.audioContext.currentTime || 0;
          this.duration = this.audioContext.duration || 0;
          if (this.duration > 0) {
            this.progress = Math.round((this.currentTime / this.duration) * 100);
          }
        }
      }, 1000);
    },
    togglePlay() {
      if (this.isPlaying) {
        this.audioContext.pause();
      } else {
        this.audioContext.play();
      }
    },
    skipBackward() {
      const newTime = Math.max(0, this.currentTime - this.currentSkipTime);
      this.audioContext.seek(newTime);
      this.currentTime = newTime;
      this.progress = Math.round((newTime / this.duration) * 100);
    },
    skipForward() {
      const newTime = Math.min(this.duration, this.currentTime + this.currentSkipTime);
      this.audioContext.seek(newTime);
      this.currentTime = newTime;
      this.progress = Math.round((newTime / this.duration) * 100);
    },
    toggleSkipMenu(direction) {
      if (this.skipMenuVisible && this.skipMenuDirection === direction) {
        this.skipMenuVisible = false;
      } else {
        this.skipMenuDirection = direction;
        this.skipMenuVisible = true;
      }
    },
    setSkipTime(time, direction) {
      this.currentSkipTime = time;
      this.skipMenuVisible = false;
      if (direction === 'backward') {
        this.skipBackward();
      } else {
        this.skipForward();
      }
    },
    prevTrack() {
      // 模拟上一曲功能
      uni.showToast({
        title: '上一曲',
        duration: 1000
      });
    },
    nextTrack() {
      // 模拟下一曲功能
      uni.showToast({
        title: '下一曲',
        duration: 1000
      });
    },
    toggleLoop() {
      this.isLoop = !this.isLoop;
      this.audioContext.loop = this.isLoop;
    },
    formatTime(seconds) {
      if (!seconds || isNaN(seconds)) return '00:00';
      const mins = Math.floor(seconds / 60);
      const secs = Math.floor(seconds % 60);
      return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
    },
    seek(e) {
      if (!this.audioContext || !this.duration || this.isDragging) return;
      
      // 使用小程序兼容的方式获取点击位置
      this.calculateProgress(e, true);
    },
    handleTouchStart(e) {
      this.isDragging = true;
      // 获取进度条宽度和左侧位置，确保拖拽过程中位置一致
      try {
        const rect = e.currentTarget.getBoundingClientRect();
        this.progressBarWidth = rect.width;
        this.progressBarLeft = rect.left;
      } catch (error) {
        // 小程序环境下的兼容处理
        this.progressBarWidth = 300;
        this.progressBarLeft = 0;
      }
      this.calculateProgress(e);
    },
    handleTouchMove(e) {
      if (!this.isDragging) return;
      this.calculateProgress(e);
    },
    handleTouchEnd(e) {
      if (!this.isDragging) return;
      this.isDragging = false;
      this.calculateProgress(e, true);
    },
    handleMouseDown(e) {
      this.isDragging = true;
      // 获取进度条宽度和左侧位置，确保拖拽过程中位置一致
      try {
        const rect = e.currentTarget.getBoundingClientRect();
        this.progressBarWidth = rect.width;
        this.progressBarLeft = rect.left;
      } catch (error) {
        // 小程序环境下的兼容处理
        this.progressBarWidth = 300;
        this.progressBarLeft = 0;
      }
      this.calculateProgress(e);
    },
    handleMouseMove(e) {
      if (!this.isDragging) return;
      this.calculateProgress(e);
    },
    handleMouseUp(e) {
      if (!this.isDragging) return;
      this.isDragging = false;
      this.calculateProgress(e, true);
    },
    calculateProgress(e, seek = false) {
      if (!this.duration) return;
      
      let clientX;
      if (e.touches && e.touches[0]) {
        clientX = e.touches[0].clientX || e.touches[0].pageX;
      } else {
        clientX = e.clientX || e.pageX;
      }
      
      // 尝试使用getBoundingClientRect，如果不支持则使用其他方式
      let rect;
      try {
        rect = e.currentTarget.getBoundingClientRect();
      } catch (error) {
        // 小程序环境下的兼容处理
        // 这里我们使用一个估算值，实际项目中可能需要更精确的计算
        rect = { left: 0, width: 300 };
      }
      
      // 使用之前获取的进度条宽度，但使用当前的左侧位置
      const width = this.progressBarWidth || rect.width;
      let offsetX = clientX - rect.left;
      
      // 限制在进度条范围内
      offsetX = Math.max(0, Math.min(offsetX, width));
      
      const percent = offsetX / width;
      const seekTime = percent * this.duration;
      
      // 更新UI
      this.currentTime = seekTime;
      this.progress = Math.round(percent * 100);
      
      // 如果是拖拽结束，则执行跳转
      if (seek && this.audioContext) {
        this.audioContext.seek(seekTime);
        // 如果之前是播放状态，立即恢复播放
        if (this.isPlaying) {
          this.audioContext.play();
        }
      }
    }
  }
};
</script>

<style>
.content {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40rpx;
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.logo {
  height: 200rpx;
  width: 200rpx;
  margin-top: 100rpx;
  margin-bottom: 50rpx;
  border-radius: 50%;
  box-shadow: 0 10rpx 30rpx rgba(0, 0, 0, 0.3);
}

.text-area {
  margin-bottom: 80rpx;
}

.title {
  font-size: 42rpx;
  font-weight: bold;
  color: #fff;
  text-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.3);
}

.audio-player {
  width: 100%;
  max-width: 650rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 60rpx;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 30rpx;
  padding: 50rpx 40rpx;
  box-shadow: 0 20rpx 60rpx rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(10px);
}

.progress-section {
  width: 100%;
  display: flex;
  align-items: center;
  gap: 20rpx;
}

.skip-btn {
  width: 80rpx;
  height: 80rpx;
  border: none;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 50%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 0;
  box-shadow: 0 6rpx 20rpx rgba(102, 126, 234, 0.4);
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.skip-btn::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(255, 255, 255, 0.3) 0%, transparent 70%);
  opacity: 0;
  transition: opacity 0.3s ease;
}

.skip-btn:active::before {
  opacity: 1;
}

.skip-btn:active {
  transform: scale(0.9);
  box-shadow: 0 3rpx 10rpx rgba(102, 126, 234, 0.6);
}

.skip-icon-wrapper {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.skip-icon {
  font-size: 36rpx;
  color: #fff;
  font-weight: bold;
  line-height: 1;
  text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.2);
}

.skip-text {
  font-size: 16rpx;
  color: #fff;
  font-weight: bold;
  text-shadow: 0 1rpx 2rpx rgba(0, 0, 0, 0.3);
  margin-top: 2rpx;
}

.skip-number {
  position: absolute;
  font-size: 16rpx;
  color: #fff;
  font-weight: bold;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-shadow: 0 1rpx 2rpx rgba(0, 0, 0, 0.3);
}

.skip-container {
  position: relative;
}

.skip-menu {
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  margin-top: 10rpx;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 10rpx;
  box-shadow: 0 10rpx 30rpx rgba(0, 0, 0, 0.2);
  padding: 10rpx;
  z-index: 100;
  backdrop-filter: blur(10px);
  display: flex;
  flex-direction: column;
  gap: 5rpx;
  min-width: 100rpx;
}

.skip-menu-item {
  border: none;
  background: transparent;
  padding: 12rpx 20rpx;
  border-radius: 8rpx;
  font-size: 24rpx;
  font-weight: 500;
  color: #333;
  text-align: center;
  transition: all 0.2s ease;
}

.skip-menu-item:hover {
  background: rgba(102, 126, 234, 0.1);
}

.skip-menu-item:active {
  background: rgba(102, 126, 234, 0.2);
  transform: scale(0.95);
}

.progress-container {
  flex: 1;
  display: flex;
  align-items: center;
  gap: 15rpx;
}

.time-text {
  font-size: 24rpx;
  color: #666;
  min-width: 80rpx;
  text-align: center;
  font-weight: 500;
}

.progress-bar {
  flex: 1;
  position: relative;
  height: 12rpx;
  background: linear-gradient(90deg, #e0e0e0 0%, #f5f5f5 100%);
  border-radius: 6rpx;
  cursor: pointer;
  overflow: hidden;
}

.progress-track {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border-radius: 6rpx;
}

.progress-fill {
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
  border-radius: 6rpx;
  transition: width 0.2s ease;
  box-shadow: 0 0 10rpx rgba(102, 126, 234, 0.5);
}

.progress-handle {
  position: absolute;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 28rpx;
  height: 28rpx;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 50%;
  transition: all 0.2s ease;
  box-shadow: 0 4rpx 15rpx rgba(102, 126, 234, 0.6);
  border: 3rpx solid #fff;
}

.progress-handle:active {
  transform: translate(-50%, -50%) scale(1.2);
}

.control-section {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 40rpx;
}

.control-btn {
  width: 80rpx;
  height: 80rpx;
  border: none;
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  border-radius: 50%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 0;
  box-shadow: 0 8rpx 25rpx rgba(240, 147, 251, 0.4);
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.control-btn::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(255, 255, 255, 0.3) 0%, transparent 70%);
  opacity: 0;
  transition: opacity 0.3s ease;
}

.control-btn:active::before {
  opacity: 1;
}

.control-btn:active {
  transform: scale(0.9);
  box-shadow: 0 4rpx 15rpx rgba(240, 147, 251, 0.6);
}

.control-icon {
  font-size: 40rpx;
  color: #fff;
  font-weight: bold;
  z-index: 1;
}

.loop-btn {
  position: relative;
}

.loop-btn.active {
  background: linear-gradient(135deg, #FFD700 0%, #FFA500 100%);
  box-shadow: 0 8rpx 25rpx rgba(255, 215, 0, 0.5);
}

.loop-btn.active .control-icon {
  color: #fff;
  text-shadow: 0 0 10rpx rgba(255, 255, 255, 0.8);
}

.play-btn {
  width: 140rpx;
  height: 140rpx;
  border: none;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
  box-shadow: 0 15rpx 40rpx rgba(102, 126, 234, 0.5);
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.play-btn::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(255, 255, 255, 0.3) 0%, transparent 70%);
  opacity: 0;
  transition: opacity 0.3s ease;
}

.play-btn:active::before {
  opacity: 1;
}

.play-btn:active {
  transform: scale(0.9);
  box-shadow: 0 8rpx 25rpx rgba(102, 126, 234, 0.7);
}

.play-icon {
  font-size: 60rpx;
  color: white;
  font-weight: bold;
  text-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.3);
  z-index: 1;
}

/* 添加动画效果 */
@keyframes pulse {
  0% {
    box-shadow: 0 15rpx 40rpx rgba(102, 126, 234, 0.5);
  }
  50% {
    box-shadow: 0 15rpx 50rpx rgba(102, 126, 234, 0.8);
  }
  100% {
    box-shadow: 0 15rpx 40rpx rgba(102, 126, 234, 0.5);
  }
}

.play-btn {
  animation: pulse 2s infinite;
}

/* 添加渐变动画 */
@keyframes gradientShift {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

.audio-player {
  background-size: 200% 200%;
  animation: gradientShift 8s ease infinite;
}
</style>
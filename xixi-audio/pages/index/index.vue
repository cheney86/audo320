<template>
  <view class="content">
    <image class="logo" src="/static/logo.png"></image>
    <view class="text-area">
      <text class="title">{{title}}</text>
    </view>
    
    <view class="audio-player">
      <!-- 进度条和时间显示 -->
      <view class="progress-section">
        <button class="skip-btn" @tap="skipBackward">
          <text class="skip-icon">↺</text>
          <text class="skip-text">15</text>
        </button>
        
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
        
        <button class="skip-btn" @tap="skipForward">
          <text class="skip-text">15</text>
          <text class="skip-icon">↻</text>
        </button>
      </view>
      
      <!-- 播放控制按钮 -->
      <view class="control-section">
        <button class="control-btn" @tap="prevTrack">
          <text class="control-icon">⏮</text>
        </button>
        
        <button class="play-btn" @tap="togglePlay">
          <text class="play-icon">{{ isPlaying ? '⏸' : '▶' }}</text>
        </button>
        
        <button class="control-btn" @tap="nextTrack">
          <text class="control-icon">⏭</text>
        </button>
        
        <button class="control-btn" @tap="toggleLoop">
          <text class="control-icon" :class="{ 'active': isLoop }">🔁</text>
          <text class="loop-text">循环</text>
        </button>
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
      updateInterval: null,
      audioUrl: 'https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3'
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
      
      this.audioContext.onLoadedMetadata(() => {
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
      const newTime = Math.max(0, this.currentTime - 15);
      this.audioContext.seek(newTime);
      this.currentTime = newTime;
      this.progress = Math.round((newTime / this.duration) * 100);
    },
    skipForward() {
      const newTime = Math.min(this.duration, this.currentTime + 15);
      this.audioContext.seek(newTime);
      this.currentTime = newTime;
      this.progress = Math.round((newTime / this.duration) * 100);
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
      
      let offsetX = clientX - rect.left;
      
      // 限制在进度条范围内
      offsetX = Math.max(0, Math.min(offsetX, rect.width));
      
      const percent = offsetX / rect.width;
      const seekTime = percent * this.duration;
      
      // 更新UI
      this.currentTime = seekTime;
      this.progress = Math.round(percent * 100);
      
      // 如果是拖拽结束，则执行跳转
      if (seek && this.audioContext) {
        this.audioContext.seek(seekTime);
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
}

.logo {
  height: 200rpx;
  width: 200rpx;
  margin-top: 100rpx;
  margin-bottom: 50rpx;
}

.text-area {
  margin-bottom: 80rpx;
}

.title {
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
}

.audio-player {
  width: 100%;
  max-width: 600rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 60rpx;
}

.progress-section {
  width: 100%;
  display: flex;
  align-items: center;
  gap: 20rpx;
}

.skip-btn {
  width: 60rpx;
  height: 60rpx;
  border: none;
  background: none;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 0;
}

.skip-icon {
  font-size: 24rpx;
  color: #666;
}

.skip-text {
  font-size: 20rpx;
  color: #666;
  margin-top: 2rpx;
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
}

.progress-bar {
  flex: 1;
  position: relative;
  height: 8rpx;
  background-color: #e0e0e0;
  border-radius: 4rpx;
  cursor: pointer;
}

.progress-track {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border-radius: 4rpx;
}

.progress-fill {
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  background-color: #4CAF50;
  border-radius: 4rpx;
  transition: width 0.2s ease;
}

.progress-handle {
  position: absolute;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 20rpx;
  height: 20rpx;
  background-color: #4CAF50;
  border-radius: 50%;
  transition: left 0.2s ease;
  box-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.2);
}

.control-section {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 40rpx;
}

.control-btn {
  width: 60rpx;
  height: 60rpx;
  border: none;
  background: none;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 0;
}

.control-icon {
  font-size: 32rpx;
  color: #666;
}

.control-icon.active {
  color: #4CAF50;
}

.loop-text {
  font-size: 20rpx;
  color: #666;
  margin-top: 2rpx;
}

.play-btn {
  width: 120rpx;
  height: 120rpx;
  border: none;
  background-color: #4CAF50;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
  box-shadow: 0 4rpx 8rpx rgba(0, 0, 0, 0.2);
}

.play-icon {
  font-size: 48rpx;
  color: white;
}

/* 按钮点击反馈 */
button:active {
  opacity: 0.8;
  transform: scale(0.95);
}

.play-btn:active {
  transform: scale(0.95);
}
</style>
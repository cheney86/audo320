<template>
  <view class="xixi-audio">
    <view class="audio-controls">
      <button @tap="play" class="control-btn">播放</button>
      <button @tap="pause" class="control-btn">暂停</button>
      <button @tap="stop" class="control-btn">停止</button>
    </view>
    <view class="audio-progress">
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
    <view class="audio-info">
      <text>当前状态: {{ status }}</text>
    </view>
  </view>
</template>

<script>
export default {
  name: 'xixi-audio',
  props: {
    src: {
      type: String,
      default: ''
    },
    autoplay: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      audioContext: null,
      status: '未初始化',
      progress: 0,
      currentTime: 0,
      duration: 0,
      updateInterval: null,
      isDragging: false,
      progressBarWidth: 0
    };
  },
  mounted() {
    this.initAudio();
  },
  beforeUnmount() {
    this.stop();
    if (this.updateInterval) {
      clearInterval(this.updateInterval);
    }
  },
  methods: {
    initAudio() {
      if (this.src) {
        this.audioContext = uni.createInnerAudioContext();
        this.audioContext.src = this.src;
        this.audioContext.autoplay = this.autoplay;
        
        this.audioContext.onPlay(() => {
          this.status = '播放中';
          this.startProgressUpdate();
        });
        
        this.audioContext.onPause(() => {
          this.status = '已暂停';
          this.stopProgressUpdate();
        });
        
        this.audioContext.onStop(() => {
          this.status = '已停止';
          this.progress = 0;
          this.stopProgressUpdate();
        });
        
        this.audioContext.onEnded(() => {
          this.status = '播放完成';
          this.progress = 100;
          this.stopProgressUpdate();
        });
        
        this.audioContext.onError((res) => {
          this.status = '播放错误';
          console.error('音频播放错误:', res || '未知错误');
        });
        
        this.status = '已初始化';
      }
    },
    play() {
      if (this.audioContext) {
        this.audioContext.play();
      } else {
        console.error('音频未初始化');
      }
    },
    pause() {
      if (this.audioContext) {
        this.audioContext.pause();
      }
    },
    stop() {
      if (this.audioContext) {
        this.audioContext.stop();
      }
    },
    startProgressUpdate() {
      this.updateInterval = setInterval(() => {
        if (this.audioContext) {
          this.currentTime = this.audioContext.currentTime || 0;
          this.duration = this.audioContext.duration || 0;
          if (this.duration > 0) {
            this.progress = Math.round((this.currentTime / this.duration) * 100);
          }
        }
      }, 1000);
    },
    stopProgressUpdate() {
      if (this.updateInterval) {
        clearInterval(this.updateInterval);
        this.updateInterval = null;
      }
    },
    formatTime(seconds) {
      if (!seconds || isNaN(seconds)) return '00:00';
      const mins = Math.floor(seconds / 60);
      const secs = Math.floor(seconds % 60);
      return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
    },
    seek(e) {
      if (!this.audioContext || !this.duration || this.isDragging) return;
      
      const rect = e.currentTarget.getBoundingClientRect();
      const offsetX = e.clientX - rect.left;
      const percent = offsetX / rect.width;
      const seekTime = percent * this.duration;
      
      this.audioContext.seek(seekTime);
      this.currentTime = seekTime;
      this.progress = Math.round(percent * 100);
    },
    handleTouchStart(e) {
      this.isDragging = true;
      this.progressBarWidth = e.currentTarget.getBoundingClientRect().width;
      this.updateProgressFromEvent(e);
    },
    handleTouchMove(e) {
      if (!this.isDragging) return;
      this.updateProgressFromEvent(e);
    },
    handleTouchEnd(e) {
      if (!this.isDragging) return;
      this.isDragging = false;
      this.updateProgressFromEvent(e, true);
    },
    handleMouseDown(e) {
      this.isDragging = true;
      this.progressBarWidth = e.currentTarget.getBoundingClientRect().width;
      this.updateProgressFromEvent(e);
    },
    handleMouseMove(e) {
      if (!this.isDragging) return;
      this.updateProgressFromEvent(e);
    },
    handleMouseUp(e) {
      if (!this.isDragging) return;
      this.isDragging = false;
      this.updateProgressFromEvent(e, true);
    },
    updateProgressFromEvent(e, seek = false) {
      if (!this.duration) return;
      
      let clientX;
      if (e.touches && e.touches[0]) {
        clientX = e.touches[0].clientX;
      } else {
        clientX = e.clientX;
      }
      
      const rect = e.currentTarget.getBoundingClientRect();
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

<style scoped>
.xixi-audio {
  padding: 20rpx;
  background-color: #f5f5f5;
  border-radius: 10rpx;
  margin: 20rpx 0;
}

.audio-controls {
  display: flex;
  gap: 20rpx;
  margin-bottom: 20rpx;
}

.control-btn {
  flex: 1;
  padding: 15rpx;
  border: 1rpx solid #ddd;
  border-radius: 5rpx;
  background-color: #fff;
  font-size: 28rpx;
}

.audio-progress {
  display: flex;
  align-items: center;
  gap: 15rpx;
  margin-bottom: 20rpx;
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
  height: 10rpx;
  background-color: #e0e0e0;
  border-radius: 5rpx;
  cursor: pointer;
}

.progress-track {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border-radius: 5rpx;
}

.progress-fill {
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  background-color: #4CAF50;
  border-radius: 5rpx;
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

.audio-info {
  font-size: 28rpx;
  color: #666;
  line-height: 1.5;
}
</style>

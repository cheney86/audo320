<template>
  <view class="xixi-audio">
    <view class="audio-controls">
      <image @tap="toggleLoop" class="control-icon" :src="loopIcon" alt="循环模式" />
      <image @tap="skipPrevious" class="control-icon" src="/static/prev.svg" alt="上一首" />
      <image @tap="togglePlayPause" class="control-icon play-icon" :src="playPauseIcon" alt="播放/暂停" />
      <image @tap="skipNext" class="control-icon" src="/static/next.svg" alt="下一首" />
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
      progressBarWidth: 0,
      progressBarLeft: 0,
      isLooping: false
    };
  },
  computed: {
    playPauseIcon() {
      return this.status === '播放中' ? '/static/pause.svg' : '/static/play.svg';
    },
    loopIcon() {
      return '/static/loop.svg';
    }
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
    togglePlayPause() {
      if (!this.audioContext) {
        console.error('音频未初始化');
        return;
      }
      if (this.status === '播放中') {
        this.pause();
      } else {
        this.play();
      }
    },
    toggleLoop() {
      if (this.audioContext) {
        this.isLooping = !this.isLooping;
        this.audioContext.loop = this.isLooping;
      }
    },
    skipPrevious() {
      // 上一首逻辑，这里只是示例，实际需要根据具体业务逻辑实现
      console.log('上一首');
    },
    skipNext() {
      // 下一首逻辑，这里只是示例，实际需要根据具体业务逻辑实现
      console.log('下一首');
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
      const rect = e.currentTarget.getBoundingClientRect();
      this.progressBarWidth = rect.width;
      this.progressBarLeft = rect.left;
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
      const rect = e.currentTarget.getBoundingClientRect();
      this.progressBarWidth = rect.width;
      this.progressBarLeft = rect.left;
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
      
      // 获取当前进度条的位置和宽度
      const rect = e.currentTarget.getBoundingClientRect();
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
        if (this.status === '播放中') {
          this.audioContext.play();
        }
      }
    }
  }
};
</script>

<style scoped>
.xixi-audio {
  width: 100%;
  height: 80px;
  background-color: #f5f5f5;
  border-radius: 10px;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.audio-controls {
  width: 100%;
  height: 48px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 20px;
}

.control-icon {
  width: 32px;
  height: 32px;
  cursor: pointer;
}

.play-icon {
  width: 48px;
  height: 48px;
}

.audio-progress {
  width: 100%;
  height: 20px;
  display: flex;
  align-items: center;
  gap: 12px;
}

.time-text {
  font-size: 12px;
  color: #666;
  min-width: 60px;
  text-align: center;
}

.progress-bar {
  flex: 1;
  position: relative;
  height: 4px;
  background-color: #e0e0e0;
  border-radius: 2px;
  cursor: pointer;
}

.progress-track {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border-radius: 2px;
}

.progress-fill {
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  background-color: #4CAF50;
  border-radius: 2px;
  transition: width 0.2s ease;
}

.progress-handle {
  position: absolute;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 20px;
  height: 20px;
  background-color: #4CAF50;
  border-radius: 50%;
  transition: left 0.2s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}
</style>

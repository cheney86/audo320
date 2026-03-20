## 1.0.0（2026-03-18）
# xixi-audio 语音播放插件

## 简介
xixi-audio是一个基于uni-app的语音播放插件，支持音频的播放、暂停、停止等基本操作，并提供状态和进度显示。

## 安装方式
在HBuilderX中，通过uni_modules安装本插件。

## 使用方法

### 方式一：直接在页面中引入组件

```vue
<template>
  <view>
    <xixi-audio 
      src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3" 
      :autoplay="false"
    />
  </view>
</template>

<script>
import xixiAudio from '@/uni_modules/xixi-audio/components/xixi-audio/xixi-audio.vue';

export default {
  components: {
    xixiAudio
  }
}
</script>
```

### 方式二：通过导入方式使用

```vue
<template>
  <view>
    <xixi-audio 
      src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3" 
      :autoplay="false"
    />
  </view>
</template>

<script>
import xixiAudio from '@/uni_modules/xixi-audio';

export default {
  components: {
    xixiAudio
  }
}
</script>
```

## 参数说明

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| src | String | '' | 音频文件的URL |
| autoplay | Boolean | false | 是否自动播放 |

## 功能说明

- 播放：点击播放按钮开始播放音频
- 暂停：点击暂停按钮暂停播放
- 停止：点击停止按钮停止播放
- 状态显示：显示当前音频播放状态
- 进度显示：显示当前音频播放进度百分比

## 注意事项

- 请确保提供有效的音频文件URL
- 在实际使用时，建议替换示例中的音频URL为真实的音频文件地址
- 插件会自动处理音频的加载、播放和错误处理


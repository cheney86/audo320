<template>
	<view class="content">
		<image class="logo" src="/static/logo.png"></image>
		<view class="text-area">
			<text class="title">{{title}}</text>
		</view>
		<xixi-audio 
			src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3" 
			:autoplay="false"
		/>
		<view class="test-info">
			<text>测试说明：</text>
			<text>1. 点击播放按钮开始播放音频</text>
			<text>2. 点击暂停按钮暂停播放</text>
			<text>3. 点击停止按钮停止播放</text>
			<text>4. 观察状态和进度显示</text>
		</view>
	</view>
</template>

<script>
	import xixiAudio from '@/uni_modules/xixi-audio/components/xixi-audio/xixi-audio.vue';
		export default {
		components: {
			xixiAudio
		},
		data() {
			return {
				title: '语音播放插件测试'
			}
		},
		onLoad() {

		},
		methods: {

		}
	}
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
		margin-bottom: 40rpx;
	}

	.title {
		font-size: 36rpx;
		font-weight: bold;
		color: #333;
	}

	.test-info {
		margin-top: 40rpx;
		padding: 20rpx;
		background-color: #f9f9f9;
		border-radius: 10rpx;
		font-size: 28rpx;
		color: #666;
		line-height: 1.6;
		width: 100%;
	}
</style>

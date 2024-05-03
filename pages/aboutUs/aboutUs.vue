<template>
	<view>
		<!-- 图标 --> 
		<view class="up">
			<image src="/static/images/profile-photo.png" class="logo"></image>
		</view>
		
		<!-- 内容 -->
		<view class="info">
			
			<!-- 左边标题 -->
			<view class="item">
				<text class="biaoti">软件名称</text>
				<text class="content">{{systemName}}</text>
			</view>
			
			<!-- 右边内容 -->
			<view class="item">
				<text class="biaoti">版本信息</text>
				<text class="content">{{version}}</text>
			</view>
			
		</view>
		
	</view>
</template>

<script>
	export default {
		data() {
			return {
				systemName: "智慧烟草种植管理系统",
				version: "v2.0"
			}
		}
	}
</script>

<style>
	.up {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}
	
	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
		border-radius: 50%;
	}
	
	.info {
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-top: 35px;
	}
	
	.item {
		display: flex;
		justify-content: space-between;
		width: 90%; 
		margin-top: 15px;
		background-color: white;
		border-radius: 10px;
	}
	
	.biaoti {
		padding: 10px;
		padding-right: 20px; 
		font-weight: 550;
		font-size: 15px;
	}
	
	.content {
		padding: 10px;
		color: #a4a4a4;
		font-weight: 550;
		font-size: 15px;
	}
</style>

<template>
	<view class="container">
		
		<!-- 意见反馈内容部分 -->
		<view class="card">
			<view class="card-title">问题和意见（200字以内）<text class="card-must">*</text></view>
			<view class="card-body">
				<textarea placeholder="请填写您的功能建议,感谢您的支持" maxlength="200" v-model="opinion" ></textarea>
			</view>
		</view>
		
		<!-- 联系方式部分 -->
		<view class="card">
			<view class="card-title">联系方式</view>
			<view class="card-body">
				<input placeholder="留下联系方式,可能更快的解决问题~" v-model="phoneNumber" type="number" maxlength="11"/>
			</view>
		</view>
		
		<!-- 提交按钮 -->
		<view class="button" @click="submit">提交</view>
		
	</view>
</template>

<script>
	export default {
		data() {
			return {
				opinion:"",  //反馈意见和问题
				phoneNumber:""  //手机号
			}
		},
		
		onLoad() {

		},
		
		methods: {
			//提交反馈意见
			submit(){
				if (this.opinion == "" || this.opinion == null) {  //对反馈意见输入框判空，如果空，则toast提示
					uni.showToast({
						title: "问题和意见不能为空",
						icon: "none",
						duration: 2000
					})
					return;
				}else if(this.phoneNumber == "" || this.phoneNumber == null){
					this.sendOpinions();
				}else{  //意见反馈和手机号都填写的情况
					if(!(/^1(3|4|5|6|7|8|9)\d{9}$/.test(this.phoneNumber))){  //对输入的手机号进行校验，是否为一个手机号，如果不是，则提示用户
						uni.showToast({
							title: "请填写正确手机号码",
							icon: "none",
							duration: 2000
						});
						return;
					}else{
						this.sendOpinions();
					}
				}
			},
			
			//发送反馈意见
			sendOpinions(){
				const token=uni.getStorageSync("login_user_token");  //获取用户的token
				const url=`http://139.155.150.150:8070/secure/opinion?phone_number=${this.phoneNumber}&description=${this.opinion}`;
				uni.request({
					url:url,
					method:"POST",
					header:{
						"Authorization": token //自定义请求头信息
					},
					success: (res) => {
						if(res.data.code*1==200){
							uni.showToast({
								title: "意见反馈成功,请等候处理",
								icon: "none",
								duration: 2000
							});
							this.opinion="";
							this.phoneNumber="";
						}else{
							this.$u.toast("意见反馈失败，请重试");
						}
					},
					fail: (err) => {
						console.error("意见反馈接口请求失败",err);
					}
				});
			}
		}
	}
</script>

<style lang="scss">
	page{
		/* 设置背景图片 */  
		background-image: url("../../static/images/background-image.jpg");  
		/* 背景图片覆盖整个容器 */  
		background-size: cover;  
		/* 根据需要设置背景位置 */  
		background-position: center; 
	}
	
	.container {
		min-height: 100vh;
		width: 100vw;
		
		.card{
			width: 710rpx;
			margin: 30rpx auto;
			
			.card-title{
				font-size: 15px;
				color: #555;
				padding: 15rpx;
				
				.card-must{
					color: red;
				}
			}
			
			.card-body{
				background: #fff;
				border-radius: 12rpx;
				padding: 15rpx;
				margin-top: 5px;
				
				textarea{
					height: 300rpx;
					font-size: 14px;
				}
				
				input{
					font-size: 14px;
					height: 70rpx;
				}
				
			}
		}
		
		.button{
			width: 500rpx;
			margin: 20rpx auto;
			border-radius: 40rpx;
			height: 80rpx;
			font-size: 16px;
			font-weight: bold;
			background-color: #55aaff;
			text-align: center;
			line-height: 80rpx;
			color: #fff;
			margin-top: 20px;
			&:active{
				background-color: #3d7bb8;
			}
		}
	}
</style>
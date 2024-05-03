<template>
	<view>
		<view>
			<!-- 头部用户信息及用户个人信息展示页面跳转 -->
			<view class="u-flex user-box u-p-l-30 u-p-r-20 u-p-b-30" @click="modify()">
				<!-- 头像 -->
				<view class="u-m-r-10 profilr-photo">
					<u-avatar :src="profilePhoto" size="160" mode="circle"></u-avatar>
				</view>
				<!-- 昵称 -->
				<view class="u-flex-1">
					<view class="u-font-18 u-p-b-20 nickname" v-if="user.nickname">{{user.nickname}}</view>
				</view>
				<!-- 右边箭头图标 -->
				<view class="u-m-l-10 u-p-10">
					<u-icon name="arrow-right" color="#969799" size="28"></u-icon>
				</view>
			</view>
			
			<!-- 中部功能部分 -->
			<view class="u-m-t-20 middle">
				<u-cell-group>
					<u-cell-item icon="edit-pen" title="修改密码" :title-style="{'fontSize': '35rpx'}" @click="toPasswordModify"></u-cell-item>
					<u-cell-item icon="info-circle" title="关于" :title-style="{'fontSize': '35rpx'}" @click="toAboutUs"></u-cell-item>
					<u-cell-item icon="file-text" title="意见反馈" :title-style="{'fontSize': '35rpx'}" @click="toFeedback"></u-cell-item>
				</u-cell-group>
			</view>
			
			<!-- 底部退出账号功能 -->
			<view class="u-m-t-20 bottom">
				<u-cell-group>
					<u-cell-item icon="man-delete" title="退出" @click="quit" :arrow="false" :title-style="{'fontSize': '35rpx'}"></u-cell-item>
				</u-cell-group>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				user: {  //用户信息
					nickname:"",  //昵称
					phoneNumber:"",  //手机号
					email:"",  //邮箱
					address:""  //地址
				},
				profilePhoto:"../../static/images/profile-photo.png" //头像（固定）
			}
		},
		
		onLoad() {
		},
		
		onShow() {
			this.user.phoneNumber=uni.getStorageSync("login_user_phoneNumber"); //从本地内存获取用户手机号
			this.getUserInfo(); //调用接口获取用户信息
		},
		
		methods: {
			// 跳转到个人信息页面
			modify(){
				uni.navigateTo({
					url: "/pages/myInfo/myInfo"
				});
			},
			
			// 跳转到修改密码页面
			toPasswordModify(){
				uni.navigateTo({
					url:"/pages/passwordModify/passwordModify"
				});
			},
			
			// 跳转到关于我们页面
			toAboutUs(){
				uni.navigateTo({
					url:"/pages/aboutUs/aboutUs"
				});
			},
			
			// 跳转到意见反馈页面
			toFeedback(){
				uni.navigateTo({
					url:"/pages/feedback/feedback"
				});
			},
			
			//退出账户
			quit(){
				//清空用户信息
				try{
					uni.setStorageSync("login_user","");
					uni.setStorageSync("login_user_phoneNumber","");
					uni.setStorageSync("login_user_token","");
				}catch(e){
					console.log("清空用户信息失败",e);
				}
				//界面切换
				uni.reLaunch({  //关闭所有页面，打开应用内的某个页面
					url:"/pages/signIn/signIn"
				});
			},
			
			// 调用接口获取用户信息
			getUserInfo(){
				const token=uni.getStorageSync("login_user_token");  //获取用户的token
				uni.request({
					url:"http://139.155.150.150:8070/secure/GetUserInfo",
					method:"GET",
					header:{
						"Authorization": token //自定义请求头信息
					},
					success: (res) => {
						if(res.data.code*1==200){  //如果返回成功json数据
							//如果用户昵称为空，即第一次注册的情况，默认用用户手机号作为昵称
							if(res.data.data.nickname==null){ 
								res.data.data.nickname=this.user.phoneNumber
							}
							//将json数据中的null处理为空字符串，方便显示
							if(res.data.data.email==null){  
								res.data.data.email=""  
							}
							if(res.data.data.address==null){  
								res.data.data.address=""  
							}
							try{
								uni.setStorageSync("login_user",res.data.data)  //将获取的用户信息存在本地内存，方便后续获取
							}catch(err){
								console.log("用户信息存储失败",err)
							}
							this.user=res.data.data  
						}else{
							console.log("用户信息获取失败")
						}
					},
					fail: (err) => {  //调用接口失败
						console.error("获取用户信息接口请求失败",err);
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
	
	.user-box{
		/* 头部用户信息部分背景颜色 */ 
		background-color: #fff; 
		opacity: 0.8;
	}
	
	.profilr-photo {
		margin-top: 15px; 
		margin-left: 10px;
	}
	
	.nickname {
		margin-left: 22px;
		font-size: 17px;
	}
	
	.middle {
		opacity: 0.8; 
		margin-top: 50px;
	}
	
	.bottom {
		opacity: 0.8;
	}
</style>

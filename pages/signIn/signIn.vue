<template>
	<view class="container">
		<!-- 背景图 -->
		<view class="top">
			<image src="../../static/images/sign-background.jpeg" mode="aspectFill"></image>
			<text class="title">智 慧 烟 草 种 植 管 理 系 统</text>
		</view>
		
		<!-- 登录注册页面切换栏 -->
		<view class="box">
			<view class="main">
				<view class="box2" >
					<view :class="navId=='login'?'active':'unactive'" @click="itemClick('login')">登录
						<view class="line" v-show="navId=='login'">
							<view class="y" ></view>
						</view>
					</view>
					
					<view :class="navId=='register'?'active':'unactive'" @click="itemClick('register')">注册
						<view class="line" v-show="navId=='register'">
							<view class="y"></view>
						</view>
					</view>
				</view>
		
				<!-- 登录 -->
				<view class="box3" v-show="navId=='login'">
					<view class="form" style="margin-top: 30px;">
						<u-form   ref="uForm" >
							
							<!-- 手机号 -->
							<u-form-item left-icon="phone" label-width="200" label="手机号" prop="phoneNumber">
								<u-input  placeholder="请输入手机号" type="number" v-model="login_user.phoneNumber" maxlength="11" :clearable="false"></u-input>
							</u-form-item>
							
							<!-- 密码 -->
							<u-form-item   left-icon="lock" label-width="180"  label="密码" prop="password">
								<u-input :password-icon="true"  type="password"  placeholder="请输入密码" v-model="login_user.password" :clearable="false"></u-input>
							</u-form-item>
						</u-form>
					</view>
					
					<!-- 登录按钮 -->
					<view class="" style="position: relative;left: 0;top: 0;">
						<u-button class="button" @click="doLogin()" style="margin-top: 25px;">确认</u-button>
					</view>
				</view>
				
				<!-- 注册 -->
				<view class="box3" v-show="navId=='register'" >
					<view class="form">
						<u-form  ref="uForm">
							
							<!-- 手机号 -->
							<u-form-item  label-width="150"  label="手机号" prop="phoneNumber/">
								<u-input type="number" placeholder="请输入手机号" v-model="register_user.phoneNumber" maxlength="11" :clearable="false" ></u-input>
							</u-form-item>
							
							<!-- 验证码 -->
							<u-form-item  label-width="150"  label="验证码" prop="code/">
								<u-input type="text" placeholder="请输入验证码" v-model="register_user.code" :clearable="false"></u-input>
								
								<!-- 提示弹窗 -->
								<u-toast ref="uToast"></u-toast>
								
								<!-- 发送验证码按钮 -->
								<u-button @tap="getCode" size="mini" :disabled="getCodeisWaiting">{{getCodeText}}</u-button>
							</u-form-item>
							
							<!-- 密码 -->
							<u-form-item  label-width="150"  label="密码" prop="password">
								<u-input :password-icon="true"   type="password"  placeholder="请输入密码" v-model="register_user.password" @input="checkPassword" :clearable="false"></u-input>
							</u-form-item>
							
							<!-- 确认密码 -->
							<u-form-item  label-width="150"  label="确认密码" prop="repassword">
								<u-input :password-icon="true"  type="password" placeholder="再输一次密码" v-model="register_user.repassword" @input="checkRepassword" :clearable="false" :disabled="disabled"></u-input>
							</u-form-item>
						</u-form>
						
						<!-- 输入状态的提示信息 -->
						<view v-if="startInput" class="check">
							<!-- A为密码输入的判断  B是通过  C是确认密码输入的判断 -->
							<image v-if="error==='A'" src="../../static/images/fault.png" class="status-icon" mode="widthFix" /> 
							<image v-if="error==='B'" src="../../static/images/pass.png" class="status-icon" mode="widthFix" />
							<text v-if="error==='A'" class="error-text" style="color: red;">密码必须包含数字、字母和特殊字符，且长度为8-16位</text>
							<text v-if="error==='B'" class="error-text" style="color: #36ab60;">输入正确</text>
							<image v-if="error==='C'" src="../../static/images/fault.png" class="status-icon" mode="widthFix" />
							<text v-if="error==='C'" class="error-text" style="color: red;">两次输入的密码不一致!</text>
						</view>
					</view>
					
					<!-- 注册按钮 -->
					<view class="" style="position: relative;left: 0;top: 0;">
						<u-button class="button" @click="doRegister()" style="margin-top: 10px;">确认</u-button>
					</view>
				
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				navId: "login",  //登录和注册页面切换状态
				getCodeText: "获取验证码",  //获取验证码按钮的文字
				getCodeisWaiting: false,  //获取验证码是否需要等待
				error:"",  //输入错误类型
				disabled: true,  //确认密码输入框的禁用状态
				startInput: false,  //输入错误信息是否显示的状态
				login_user:{  //用户登录的信息
					phoneNumber:"",  //手机号
					password:"",  //密码
					token:""  //登录成功后获得的认证信息
				},
				register_user:{  //用户注册的信息
					phoneNumber:"",  //手机号
					code: "",  //验证码
					password:"",  //密码
					repassword:""  //确认密码
				}
			};
		},
		methods: {
			// 发送验证码
			getCode() {
				uni.hideKeyboard(); //隐藏已经显示的软键盘，如果软键盘没有显示则不做任何操作。
				if (this.getCodeisWaiting) {  //如果验证码处于等待状态,则不能发送验证码,即在发送后60秒内不能再次发送验证码
					return;
				}
				if (!(/^1(3|4|5|6|7|8|9)\d{9}$/.test(this.register_user.phoneNumber))) { //校验手机号码是否有误
					uni.showToast({
						title: '请填写正确手机号码',
						icon: "none"
					});
					return false;
				}
				this.getCodeText = "发送中..."; //发送验证码
				this.getCodeisWaiting = true;
				let url=`http://139.155.150.150:8070/api/msm/RegisterSend?phoneNumber=${this.register_user.phoneNumber}`;
				uni.request({
					url:url,
					method:"GET",
					success: (res) => {
						if(res.data.code*1==200){
							this.setTimer(); //调用定时器方法
							uni.showToast({
								title: '验证码已发送',
								icon: "none"
							}); 
						}else if(res.data.code*1==400){  //已注册用户不能再注册
							uni.showToast({
								title: '该用户已注册',
								icon: "none"
							});
							this.getCodeisWaiting=false;  //验证码发送失败后恢复验证码发送按钮的功能
							this.getCodeText = "获取验证码";
						}else{
							uni.showToast({
								title: '验证码发送失败',
								icon: "none"
							}); 
							console.log('验证码发送失败',err);
							this.getCodeisWaiting=false;  //验证码发送失败后恢复验证码发送按钮的功能
							this.getCodeText = "获取验证码";
						}
					},
					fail: function(err){ 
						console.log('验证码发送接口请求失败',err);
						this.getCodeisWaiting=false;   //验证码发送失败后恢复验证码发送按钮的功能
						this.getCodeText = "获取验证码";
						uni.showToast({
							title: '验证码发送失败',
							icon: "none"
						}); 
					}
				});
			},
			
			//定时器方法
			setTimer() {
				let holdTime = 60; //定义变量并赋值
				this.getCodeText = "重新获取(60)";
				this.Timer = setInterval(() => {
					if (holdTime <= 0) {
						this.getCodeisWaiting = false;
						this.getCodeText = "获取验证码";
						clearInterval(this.Timer); //清除该函数
						return; //返回前面
					}
					this.getCodeText = "重新获取(" + holdTime + ")";
						holdTime--;
				}, 1000)
			},

			//监测注册密码输入
			checkPassword(){
				this.startInput=true;  //开启错误信息提示区域
				const regex = /^(?=.*[0-9])(?=.*[a-zA-Z])(?=.*[!@#$%^&*?~`()<>.,/;'":]).{8,16}$/;  //校验密码的正则表达式
				// 对不同的错误情况赋以不同的错误类型
				if((this.register_user.password==''||this.register_user.password==null) && (this.register_user.repassword==''||this.register_user.repassword==null)){
					this.startInput=false;
				}else if(!regex.test(this.register_user.password)){
					this.error = 'A';
				}else{
					this.error = 'B';
					this.disabled=false;  //密码不符合要求时无法填写确认密码
				}
			},
			
			//监测注册确认密码输入
			checkRepassword(){
				this.startInput=true;  //开启错误信息提示区域
				// 对不同的错误情况赋以不同的错误类型
				if((this.register_user.password==''||this.register_user.password==null) && (this.register_user.repassword==''||this.register_user.repassword==null)){
					this.startInput=false;
				}else if(this.register_user.password == this.register_user.repassword){
					this.error = 'B';
				}else{
					this.error = 'C';
				}
			},
			
			//切换登录or注册
			itemClick(value) {
				this.navId = value;
			},
			
			//登录
			doLogin() {
				// 对密码进行判空
				if (this.login_user.phoneNumber == '' || this.login_user.phoneNumber == null) {
					uni.showToast({
						title: '请输入手机号',
						icon: 'none',
						duration: 2000
					});
					return;
				// 对手机号进行校验
				} else if(!(/^1(3|4|5|6|7|8|9)\d{9}$/.test(this.login_user.phoneNumber))) { //校验手机号码是否有误
					uni.showToast({
						title: '请填写正确手机号码',
						icon: "none",
						duration: 2000
					});
					return;
				// 对密码进行判空
				}else if (this.login_user.password == '' || this.login_user.password == null) {
					uni.showToast({
						title: '请输入密码',
						icon: 'none',
						duration: 2000
					});
					return;
				// 对密码进行校验
				} else if(!(/^(?=.*[0-9])(?=.*[a-zA-Z])(?=.*[!@#$%^&*?~`()<>.,/;'":]).{8,16}$/.test(this.login_user.password))) { 
					uni.showToast({
						title: '请填写正确的密码',
						icon: "none",
						duration: 2000
					});
					return;
				// 调用接口登录
				}else {
					uni.request({
						url:"http://139.155.150.150:8070/login",
						method:"POST",
						data:{
							phoneNumber:this.login_user.phoneNumber,
							password:this.login_user.password
						},
						success: (res) => {
							// 登录成功，处理成功逻辑  
							if(res.data.code*1==200){
								//存储后端返回的用户信息数据
								uni.setStorageSync("login_user",res.data.data.user);
								// 存储用户的手机号
								uni.setStorageSync('login_user_phoneNumber',res.data.data.user.phoneNumber);
								// 存储用户的token
								uni.setStorageSync("login_user_token",res.data.data.token);
								//登录成功跳转到首页,因为是tarbar的页面,因此用switchTab
								uni.switchTab({
									url: '/pages/index/index'
								});
							}else{
								this.$u.toast("登录失败，请检查手机号/密码,如果没有注册请注册账号")
								console.log("登录失败");
							}
						},
						fail: (err) => {
							console.log("登录接口请求失败",err);
						}
					});
				}
			},
			
			// 注册
			doRegister() {
				uni.hideKeyboard(); //隐藏已经显示的软键盘，如果软键盘没有显示则不做任何操作。
				// 对注册手机号进行判空
				if (this.register_user.phoneNumber == '' || this.register_user.phoneNumber == null) {
					uni.showToast({
						title: '手机号不能为空',
						icon: 'none',
						duration: 2000
					});
					return;
				// 对验证码进行判空
				} else if (this.register_user.code == '' || this.register_user.code == null) {
					uni.showToast({
						title: '验证码不能为空',
						icon: 'none',
						duration: 2000
					});
					return;
				// 对密码进行判空
				} else if (this.register_user.password == '' || this.register_user.password == null) {
					uni.showToast({
						title: '密码不能为空',
						icon: 'none',
						duration: 2000
					});
					return;
				// 对确认密码进行判空
				} else if (this.register_user.repassword == '' || this.register_user.repassword == null) {
					uni.showToast({
						title: '确认密码不能为空',
						icon: 'none',
						duration: 2000
					});
					return;
				// 调用接口注册
				} else {
					uni.request({
						url:'http://139.155.150.150:8070/register',
						method:"POST",
						data:{
							phoneNumber:this.register_user.phoneNumber,
							password:this.register_user.password,
							code:this.register_user.code
						},
						success: (res) => {
							// 注册成功，成功处理逻辑  
							if(res.data.code*1==200){
								this.navId="login"  //注册成功就切换到登录页面
							}else{
								this.$u.toast("注册失败，请检测验证码");
								console.log("注册失败");
							}
						},
						fail: (err) => {
							console.log("注册接口请求失败",err);
						}
					});
				}
			}
		}
	}
</script>

<style lang="scss">
	.container {
		font-family: PingFangSC, PingFang SC;
	}
 
	.top {
		position: relative;
		left: 0;
		top: 0;
 
		.title {
			display: inline-block;
			width: 700rpx;
			padding: 3rpx;
			color: #fff;
			font-weight: bold;
			font-size: 50rpx;
			line-height: 62rpx;
			position: absolute;
			left: 50%;
			top: 50%;
			transform: translate(-50%, -50%);
			text-align: center;
		}
		
		image {
			width: 100%;
			border-bottom-left-radius: 40rpx;
			border-bottom-right-radius: 40rpx;
		}
	}
 
	.box {
		width: 100%;
		display: flex;
		justify-content: center;
		height: 300px;
 
		.main {
			position: relative;
			width: 670rpx;
			height: 770rpx;
			background-color: #ffffff;
			box-shadow: 0px 0px 9px 0px rgba(10, 53, 74, 0.19);
			border-radius: 40rpx;
			margin-top: -150rpx;
		}
	}
 
	.box2 {
		width: 100%;
		padding: 39rpx;
		display: flex;
		align-items: center;
 
		view {
			margin-right: 50rpx;
			font-size: 28rpx;
			color: #999999;
			line-height: 40rpx;
			font-weight: 600;
		}
 
		.active {
			font-size: 46rpx;
			font-weight: 600;
			color: #333333;
			line-height: 65rpx;
		}
 
		.line {
			width: 100%;
			height: 6rpx;
			background: linear-gradient(122deg, #2AB1E7 0%, #51B1FF 100%);
			border-radius: 3rpx;
			position: relative;
			left: 0;
			top: 0;
	 
			.y {
				width: 30rpx;
				height: 30rpx;
				background: rgba(0, 145, 255, 0.2);
				border-radius: 15rpx;
				position: absolute;
				right: -15rpx;
				top: -15rpx;
				margin-right: 0;
			}
		}
	}
 
	.box3 {
			width: 100%;
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: center;
 
		.items {
			width: 550rpx;
			height: 92rpx;
			background: #F8F8F8;
			border-radius: 10rpx;
			display: flex;
			align-items: center;
			padding: 0 30rpx;
		}
	 
		.uni-input-placeholder {
			color: #C7D4DD;
		}
		
		.u-form-item {  
			margin-bottom: 10rpx; /* 或者其他你想要的间距值 */  
		}
	 
		.uni-input {
			flex: 1;
			font-size: 26rpx;
			font-weight: 400;
			color: #333333;
			line-height: 37rpx;
		}
	 
		.input2 {
			width: 200rpx;
			flex: none;
		}
	}
	
	.check{
		display: flex;
		margin-top: 5px;
		width: 100%;
	  
		.status-icon {
			width: 12px;   
			height: 12px; 
		}  
		
		.error-text {  
			font-size: 9px;
			margin-left: 5px;
		}
	}
	
	.button {
		color: #fff;
		width: 200px;
		background-color: #1C85C8;
	}
</style>
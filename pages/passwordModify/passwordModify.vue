<template>
	<view id="contianer">
		<view class="pagetitle">密码修改</view>
			<!-- 手机号 -->
			<view class="password">
				<view class="contentuser">
					<view class="user" style="color: #b4b4b4;">
						手机号
					</view>
					<view class="contentuserText">
						<input v-model="modify_user.phoneNumber" type="number" maxlength="11" class="input" placeholder="请输入手机号" disabled="true"/>
						<u-toast ref="uToast"></u-toast>
						<view class="button-wrap">
							<button @click="getCode" size="mini" :disabled="getCodeisWaiting" >{{getCodeText}}</button>
						</view>
					</view>
				</view>
				<view class="border"></view>
			</view>
			
			<!-- 验证码 -->
			<view class="password">
				<view class="contentuser">
					<view class="user">
						验证码
					</view>
					<view class="contentuserText">
						<input v-model="modify_user.code" type="number" class="input" placeholder="请输入验证码"/>
					</view>
				</view>
				<view class="border"></view>
			</view>
			
			<!-- 新密码 -->
			<view class="password">
				<view class="contentuser">
					<view class="user">
						新密码
					</view>
					<view class="contentuserText">
						<input v-model="modify_user.password" type="password" v-show="eyeVisible[0]"  class="input" placeholder="请输入密码" @input="checkPasswd()"/>
						<input v-model="modify_user.password" type="text" v-show="!eyeVisible[0]" class="input" placeholder="请输入密码" @input="checkPasswd()"/>
						<u-icon name="eye-fill" v-if="!eyeVisible[0]" size="28"
						@click="togglePasswordVisibility(0)"></u-icon>
						<u-icon name="eye-off" size="28" v-if="eyeVisible[0]"
							@click="togglePasswordVisibility(0)"></u-icon>
					</view>
				</view>
				<view class="border"></view>
			</view>
			
			<!-- 确认密码 -->
			<view class="password">
				<view class="contentuser">
					<view class="user">
						确认密码
					</view>
					<view class="contentuserText">
						<input v-model="modify_user.repassword" type="password" v-show="eyeVisible[1]" class="input" placeholder="请再次输入新密码" @input="checkRepasswd()" :disabled="disabled"/>
						<input v-model="modify_user.repassword" type="text" v-show="!eyeVisible[1]" class="input" placeholder="请再次输入新密码" @input="checkRepasswd()" :disabled="disabled"/>
						<u-icon name="eye-fill" v-if="!eyeVisible[1]" size="28"
							@click="togglePasswordVisibility(1)"></u-icon>
						<u-icon name="eye-off" size="28" v-if="eyeVisible[1]"
							@click="togglePasswordVisibility(1)"></u-icon>
					</view>
				</view>
				<view class="border"></view>
			</view>
			
			<!-- 输入错误提示区 -->
			<view class="password">
				<view  v-if="startInput" class="check">
					<image v-if="error==='A'" src="../../static/images/fault.png" class="status-icon" mode="widthFix"/> 
					<image v-if="error==='B'" src="../../static/images/pass.png" class="status-icon" mode="widthFix"/>
					<text v-if="error==='A'" class="error-text" style="color: red;">密码必须包含数字、字母和特殊字符，且长度为8-16位</text>
					<text v-if="error==='B'" class="error-text" style="color: #36ab60;">输入正确</text>
					<image v-if="error==='C'" src="../../static/images/fault.png" class="status-icon" mode="widthFix"/>
					<text v-if="error==='C'" class="error-text" style="color: red;">两次输入的密码不一致!</text>
				</view>
			</view>
		<view style="margin-top: 60rpx;"><button class="button" @tap="signPut()">提交</button></view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				eyeVisible: [true, true], // 控制眼睛图标显示状态的数组
				getCodeText: "获取验证码",  //获取验证码按钮的文字
				getCodeisWaiting: false,  //获取验证码是否要等待
				disabled:true,  //确认密码输入框是否禁用
				error:"",  //错误类型
				startInput:false,  //错误信息显示区域是否开启
				modify_user:{  //用户修改的信息
					phoneNumber:"",  //手机号
					code: "",  //验证码
					password:"",  //新密码
					repassword:""  //确认密码
				}
			};
		},
		
		onShow() {
			const value=uni.getStorageSync("login_user_phoneNumber");  //获取内存中用户的电话
			if(value){  //手机号自动显示,并且不能修改
				this.modify_user.phoneNumber=value;
			}else{
				console.log("从数据缓存获取用户手机号失败");
			}
		},	
			
		methods: {
			getCode() {
				const token=uni.getStorageSync("login_user_token");  //获取用户token
				uni.hideKeyboard(); //隐藏已经显示的软键盘，如果软键盘没有显示则不做任何操作。
				if (this.getCodeisWaiting) {  //如果获取验证码需要等待,则不能发验证码
					return;
				}
				this.getCodeText = "发送中..."; //发送验证码
				this.getCodeisWaiting = true;  //发送验证码后修改按钮状态为禁用
				let url=`http://139.155.150.150:8070/api/msm/UpdateSend?phoneNumber=${this.modify_user.phoneNumber}`;
				uni.request({
					url:url,
					method:"GET",
					header:{
						"Authorization": token
					},
					success: (res) => {
						if(res.data.code*1==200){
							this.setTimer(); //调用定时器方法
							uni.showToast({
								title: "验证码已发送",
								icon: "none"
							}); 
						}else{
							console.log("验证码发送失败");
							uni.showToast({
								title: "验证码发送失败",
								icon: "none"
							}); 
							this.getCodeisWaiting = false;
							this.getCodeText = "获取验证码";
						}
					},
					fail: function(err){
						uni.showToast({
							title: "验证码发送失败",
							icon: "none"
						}); 
						this.getCodeisWaiting = false;
						this.getCodeText = "获取验证码";
						console.log("验证码发送接口请求失败",err);
					}
				})
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
					this.getCodeText = "重新获取(" + holdTime + ")"
						holdTime--;
				}, 1000)
			},
			
			//监测新密码输入框
			checkPasswd(){
				this.startInput=true;  //开启错误信息显示区域
				const regex = /^(?=.*[0-9])(?=.*[a-zA-Z])(?=.*[!@#$%^&*?~`()<>.,/;"":]).{8,16}$/;  //密码校验的正则表达式
				// 对不同的错误信息显示不同的错误类型
				if((this.modify_user.password==""||this.modify_user.password==null) && (this.modify_user.repassword==""||this.modify_user.repassword==null)){
					this.startInput=false;
				}else if(this.modify_user.password==""||this.modify_user.password==null){
					this.startInput=false;
				}else if(!regex.test(this.modify_user.password)){
					this.error = "A";
				}else{
					this.error = "B";
					this.disabled=false;
				}
			},
			
			//监测确认密码输入框
			checkRepasswd(){
				this.startInput=true;  //开启错误信息显示区域
				// 对不同的错误信息显示不同的错误类型
				if((this.modify_user.password==""||this.modify_user.password==null) && (this.modify_user.repassword==""||this.modify_user.repassword==null)){
					this.startInput=false;
				}else if(this.modify_user.repassword==""||this.modify_user.repassword==null){
					this.startInput=false;
				}else if(this.modify_user.password == this.modify_user.repassword){
					this.error = "B";
				}else{
					this.error = "C";
				}
			},
			
			//改变密码可视状态
			togglePasswordVisibility(index) {
				// 使用 $set手动更新
				this.$set(this.eyeVisible, index, !this.eyeVisible[index]); //控制密码框是否可视密码
			},
			
			// 修改密码
			signPut() {
				const token=uni.getStorageSync("login_user_token");
				uni.hideKeyboard(); //隐藏已经显示的软键盘，如果软键盘没有显示则不做任何操作。
				// 对验证码进行判空
				if (this.modify_user.code == "" || this.modify_user.code == null) {
					uni.showToast({
						title: "验证码不能为空",
						icon: "none",
						duration: 2000
					})
					return;
				// 对密码进行判空
				} else if (this.modify_user.password == "" || this.modify_user.password == null) {
					uni.showToast({
						title: "新密码不能为空",
						icon: "none",
						duration: 2000
					})
					return;
				// 对确认密码进行判空
				} else if (this.modify_user.repassword == "" || this.modify_user.repassword == null) {
					uni.showToast({
						title: "确认密码不能为空",
						icon: "none",
						duration: 2000
					})
					return;
				// 对密码进行校验
				} else if(!(/^(?=.*[0-9])(?=.*[a-zA-Z])(?=.*[!@#$%^&*?~`()<>.,/;"":]).{8,16}$/.test(this.modify_user.password))) { 
					uni.showToast({
						title: "请填写正确的密码",
						icon: "none",
						duration: 2000
					});
					return;
				// 调用后端接口修改密码
				}else {
					let url=`http://139.155.150.150:8070/secure/UpdatePassword?phoneNumber=${this.modify_user.phoneNumber}&NewPassword=${this.modify_user.password}&code=${this.modify_user.code}`;
					uni.request({
						url:url,
						method:"POST",
						header:{
							"Authorization": token //自定义请求头信息
						},
						success: (res) => {
							//如果修改成功,需要登出重新登录
							if(res.data.code*1==200){
								uni.showToast({
									title: "密码修改成功,请重新登录",
									icon: "none",
									duration: 2000
								});
								// 清空用户信息
								try{
									uni.setStorageSync("login_user","");
									uni.setStorageSync("login_user_token","");
									uni.setStorageSync("login_user_phoneNumber","");
								}catch(e){
									console.log("用户信息清空失败",e);
								}
								// 关闭所有页面,打开登录页面
								uni.reLaunch({
									url:"/pages/signIn/signIn"
								});
							}else{
								this.$u.toast("密码修改失败，请检测验证码");
							}
						},
						fail: (err) => {
							console.error("修改密码接口请求失败",err);
						}
					})
				}
			}
		}
	}
</script>



<style scoped>
	page{
		/* 设置背景图片 */  
		background-image: url("../../static/images/background-image.jpg"); 
		/* 背景图片覆盖整个容器 */  
		background-size: cover;  
		/* 根据需要设置背景位置 */  
		background-position: center; 
	}
	
	#contianer {
		padding-top: 1rpx;
		position: relative;
		height: 100%;
	}
	
	.pagetitle{
		padding-top: 150rpx;
		display: flex;
		justify-content: flex-start;
		margin-left: 50rpx;
		font-weight: 500;
		font-size: 50rpx;
		color: #011108;
		letter-spacing: 5rpx;
		margin-bottom: 50rpx;
	}
	
	.button-wrap {
		width:92%; 
		margin-top:10px; 
		font-size: 12px; 
		height: 100%; 
		text-align: right;
	}
	
	.input{
		height: 100rpx;
	}
	
	.user {
		font-size: 32rpx;
		font-family: Inter, Inter;
		font-weight: 400;
		color: #333333;
		padding-left: 5rpx;
	}
 
	.contentuser {
		display: flex;
		padding-right: 20rpx;
		box-sizing: border-box;
		justify-content: space-between;
		align-items: center;
		height: 116rpx;
	}
 
	.contentuserText {
		display: flex;
		width: 500rpx;
		font-size: 28rpx;
		font-family: Inter, Inter;
		padding-right: 15rpx;
		font-weight: 400;
		color: #666666;
	}

	.border {
		width: 656rpx;
		height: 1rpx;
		opacity: 1;
		border-bottom: 0.5rpx solid #D2D2D2;
 
	}
 
	.password {
		box-sizing: border-box;
		padding-left: 40rpx;
	}

	.button {
		width: 90%;
		font-size: 32rpx;
		height: 80rpx;
		color: #fff;
		border-radius: 68rpx;
		margin-top: 82rpx;
		text-align: center;
		line-height: 80rpx;
		background-color: #00bfff;
		margin-left: auto;
		margin-right: auto;

	}
	
	.check{
		  display: flex;
		  margin-top: 5px;
		  width: 100%;
	}
	
	.status-icon {
		 width: 17px; 
		 height: 10px;  
	}  
	
	.error-text {  
		font-size: 13px;
		margin-left: 7px;
	}
</style>

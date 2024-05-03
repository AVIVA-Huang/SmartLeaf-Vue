<template>
	<view>
		<view class="u-m-t-20" style="opacity: 0.8;margin-top: 50px;">
			<!-- 循环生成用户信息 -->
			<u-cell-group v-for="(item, index) in list" :key="index">  
			    <u-cell-item :title="item.title" @click="editItem(index)" :title-style="{'fontSize': '35rpx'}" :required="item.required" :arrow="item.arrow">  
			        <view v-if="!isEditing || isEditing !== index">  
			          {{ item.value }}
			        </view>  
			        <u-input v-else v-model="list[index].value" placeholder="请输入新值" input-align="right" :clearable="false" @input="monitor(index) "></u-input>  
			    </u-cell-item> 
			</u-cell-group> 
			 
			<!-- 修改信息时输入不满足调节时的错误提示区 A为昵称错误,B通过,C为邮箱错误-->
			<view v-if="startInput" class="check">
				<image v-if="error==='A'" src="../../static/images/fault.png" class="status-icon" mode="widthFix"/> 
				<image v-if="error==='B'" src="../../static/images/pass.png" class="status-icon" mode="widthFix"/>
				<text v-if="error==='A'" style="color: red;">昵称为2~12位中文、英文或数字</text>
				<text v-if="error==='B'" style="color: #36ab60;">输入正确</text>
				<image v-if="error==='C'" src="../../static/images/fault.png" class="status-icon" mode="widthFix"/>
				<text v-if="error==='C'" style="color: red;">邮箱格式不正确</text>
			</view>
			
			<view class="button-wrap">
				
				<!-- 修改信息按钮 -->
			 	<view style="margin-right: 80px;">
			 		<button @click="saveItem()" size="mini">保存</button>  
			 	</view>
				
				<!-- 取消修改信息,显示原信息 -->
				<view>
			 		<button @click="cancelEdit" size="mini">取消</button>  
			 	</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				list:[  //用户信息的属性
					{
						title: "手机号",  //用户信息标题
						value: "",  //用户信息
						required:true,  //是否必填
						arrow:false  //右侧是否有箭头
					},
					{
						title: "昵称",
						value: "",
						required:true,
						arrow:true
					},
					{
						title: "邮箱",
						value: "",
						required:false,
						arrow:true
					},
					{
						title: "地址",
						value: "",
						required:false,
						arrow:true
					}
				],
				isEditing: null ,// 当前正在编辑的cell的索引  
				error:"",  //错误类型
				startInput:false  //错误信息区域是否显示
			}
		},
		
		onShow(){
			const value=uni.getStorageSync("login_user");  //获取内存用户信息
			this.list[0].value=uni.getStorageSync("login_user_phoneNumber");  //获取内存中用户手机号
			if(value.nickname==null||value.nickname==""){  //将获取的用户信息进行赋值，若为null处理为空字符串
				this.list[1].value="";
			}else{
				this.list[1].value=value.nickname;
			}
			if(value.email==null||value.email==""){
				this.list[2].value="";
			}else{
				this.list[2].value=value.email;
			}
			if(value.address==null||value.address==""){
				this.list[3].value="";
			}else{
				this.list[3].value=value.address;
			}
		},
		
		methods: {
			// 获取正在编辑的cell的索引,如果是手机号cell,提示手机号不能修改
			editItem(index) {    
				if(index==0){
					uni.showToast({
						title:"手机号不能修改",
						icon:"none",
						duration:2000
					})
				};
			    this.isEditing = index;  
			},  
			
			// 监测每个cell的输入情况
			monitor(index){
				switch(index){
					// 对昵称输入进行校验
					case 1:
						this.startInput=true;
						if(this.list[1].value==""||this.list[1].value==null){
							this.startInput=false;
						}else if(!(/^[\u4e00-\u9fa5a-zA-Z0-9]{2,12}$/.test(this.list[1].value))){
							this.error="A";
						}else{
							this.error="B";
						}
						break;
						// 对邮箱格式进行校验
					case 2:
						this.startInput=true;
						if(this.list[2].value==""||this.list[2].value==null){
							this.startInput=false;
						}else if(!(/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/.test(this.list[2].value))){
							this.error="C";
						}else{
							this.error="B";
						}
						break;
					default:
						return;
				}
			},
			
			// 修改用户信息
			saveItem() {  
				const token=uni.getStorageSync("login_user_token");
				// 对昵称进行判空
				if(this.list[1].value==""||this.list[1].value==null){
					uni.showToast({
						title: "昵称不能为空",
						icon: "none",
						duration: 2000
					});
					return;
				// 对昵称进行校验
				}else if(!(/^[\u4e00-\u9fa5a-zA-Z0-9]{2,12}$/.test(this.list[1].value))){
					uni.showToast({
						title: "请填写正确的昵称",
						icon: "none",
						duration: 2000
					});
					return;
				// 当邮箱为空时,直接调用接口修改用户信息
				}else if(this.list[2].value=="" || this.list[2].value==null){
					uni.request({
						url:"http://139.155.150.150:8070/secure/SetUserInfo",
						method:"POST",
						header:{
							"Authorization": token
						},
						data:{
							nickname:this.list[1].value,
							email:"",
							address:this.list[3].value
						},
						success: (res) => {
							if(res.data.code*1==200){
								uni.showToast({
									title: "修改成功",
									icon: "none",
									duration: 2000
								});
								uni.setStorageSync("login_user",res.data.data);
								const login_user=uni.getStorageSync("login_user");
							}else{
								console.error("个人信息修改失败",err);
								this.$u.toast("个人信息修改失败");
							}
						},
						fail: (err) => {
							console.error("修改个人信息接口请求失败",err);
						}
					})
					// 当邮箱不为空时,校验邮箱格式,再调用接口修改用户信息
				}else{
					if(!(/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/.test(this.list[2].value))){
						uni.showToast({
							title: "请填写正确的邮箱格式",
							icon: "none",
							duration: 2000
						});
						return;
					}
					uni.request({
						url:"http://139.155.150.150:8070/secure/SetUserInfo",
						method:"POST",
						header:{
							"Authorization": token
						},
						data:{
							nickname:this.list[1].value,
							email:this.list[2].value,
							address:this.list[3].value
						},
						success: (res) => {
							if(res.data.code*1==200){
								uni.showToast({
									title: "修改成功",
									icon: "none",
									duration: 2000
								});
								uni.setStorageSync("login_user",res.data.data);
								const login_user=uni.getStorageSync("login_user");
							}else{
								console.error("个人信息修改失败",err);
								this.$u.toast("个人信息修改失败");
							}
						},
						fail: (err) => {
							console.error("修改个人信息接口请求失败",err);
						}
					})
				}
			},  
			
			// 取消编辑，所有信息还原
			cancelEdit() {  
				const value=uni.getStorageSync("login_user");
				this.list[1].value=value.nickname;
				this.list[2].value=value.email;
				this.list[3].value=value.address;
			    this.isEditing = null;
				this.startInput=false;
			}
		}
	}
</script>

<style>
	page{
		/* 设置背景图片 */  
		background-image: url("../../static/images/background-image.jpg");  
		/* 背景图片覆盖整个容器 */  
		background-size: cover;  
		/* 根据需要设置背景位置 */  
		background-position: center; 
	}
	
	.check{
		display: flex;
		margin-top: 5px;
		width: 100%;
		margin-left: 10px;
		
		image {
			width: 16px;
		}
	}
	
	.button-wrap {
		width: 100%;
		height: 100%;
		margin-left: 20%;
		margin-top:10px; 
		display: flex;
	}
</style>

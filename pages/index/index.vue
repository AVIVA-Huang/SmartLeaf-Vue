<template>
	<view>
		<!-- 欢迎提示框 -->
		<view class="u-flex u-p-l-30 u-p-r-20 u-p-b-30">
			<!-- 固定头像 -->
			<view class="u-m-r-10" style="margin-top: 20px;">
				<u-avatar :src="profilePhoto" size="140" mode="circle"></u-avatar>
			</view>
			<view class="u-flex-1" style="margin-top: 20px;">
				<!-- 欢迎语 -->
				<view class="u-font-18 u-p-b-20 welcome">欢迎来到智慧烟草种植基地</view>
				<!-- 当天日期 -->
				<view class="u-font-14 u-tips-color" style="margin-left: 20px; font-size: 13px;">{{ currentDate }}</view>
			</view>
		</view>
		
		<!-- 实时天气信息 调用了第三方API（和风天气）-->
		<view class="weather-card" v-if="now">  
			<image class="weather-icon" :src="`/static/icons/${now.icon}.svg`" alt="QWeather Icons"></image>
			<view class="weather-info">  
				<text class="weather-text">{{now.text}} | </text>  
				<text class="temp-text"> 气温{{now.temp}}°C | </text>  
				<text class="wind-text"> {{now.windDir}}{{now.windScale}}级 | </text>  
				<text class="humidity-text"> 湿度{{now.humidity}}%</text> 
			</view>    
		</view>
		
		<!-- 轮播图 -->
		<view class="wrap" style="margin-top: 15px;">
			<u-swiper :list="list"></u-swiper>
		</view>
		
		<!-- 实时数据 -->
		<view style="opacity: 0.8;">
			<u-card title="实时数据" title-size="titleSize">
				<view class="" slot="body">
					<view class="u-body-item u-flex u-border-bottom u-col-between u-p-t-0">
						
						<!-- 光照强度 正常情况为normal1=true 预警时normal1=false -->
						<view class="data">
							<image v-if="normal1" src="../../static/images/light.png"></image>
							<image v-else src="../../static/images/light-alert.png"></image>
							<view v-if="normal1" class="inTime-normal">光照强度</view>
							<view v-else class="inTime-abnormal">光照强度</view>
							<view v-if="normal1" class="inTime-normal">{{lightIntensity}}μmol·m<sup>-2</sup>·s<sup>-1</sup></view>
							<view v-else class="inTime-abnormal">{{lightIntensity}}μmol·m<sup>-2</sup>·s<sup>-1</sup></view>
						</view>
						
						<!-- 温度 正常情况为normal2=true 预警时normal2=false -->
						<view class="data">
							<image v-if="normal2" src="../../static/images/temp.png"></image>
							<image v-else src="../../static/images/temp-alert.png"></image>
							<view v-if="normal2" class="inTime-normal">温度</view>
							<view v-else class="inTime-abnormal">温度</view>
							<view v-if="normal2" class="inTime-normal">{{temperature}}℃</view>
							<view v-else class="inTime-abnormal">{{temperature}}℃</view>
						</view>
						
						<!-- 湿度 正常情况为normal3=true 预警时normal3=false -->
						<view class="data">
							<image v-if="normal3" src="../../static/images/wet.png"></image>
							<image v-else src="../../static/images/wet-alert.png"></image>
							<view v-if="normal3" class="inTime-normal">湿度</view>
							<view v-else class="inTime-abnormal">湿度</view>
							<view v-if="normal3" class="inTime-normal">{{humidity}}%</view>
							<view v-else class="inTime-abnormal">{{humidity}}%</view>
						</view>
						
					</view>
				</view>
			</u-card>
		</view>
		
		<!-- 传感器状态显示 -->
		<view style="opacity: 0.8;">
			<u-card title="状态显示" title-size="titleSize">
				<view class="" slot="body">
					<view class="u-body-item u-flex u-border-bottom u-col-between u-p-t-0">
						
						<!-- LED灯状态 -->
						<view class="data" style="margin-left: 0px; margin-right: 5px;">
							<image src="../../static/images/lamp.png" style="width: 60rpx; height: 60rpx; margin-top:10px;"></image>
							<view class="status1">
								<view class="status1-tip">自动化状态:</view>
								<view class="status">
									<image v-if="isClosed1" src="../../static/images/close.png"></image>
									<image v-else src="../../static/images/open.png" ></image>
								</view>
							</view>
							<view class="status1">
								<view class="status1-tip">LED灯状态:</view>
								<view class="status">
									<image v-if="isClosed2" src="../../static/images/close.png" ></image>
									<image v-else src="../../static/images/open.png" ></image>
								</view>
							</view>
							<view class="status1">
								<view class="status1-tip" >当前挡位:  {{light.lightLevel}}</view>
							</view>
						</view>
						
						<!-- 风扇状态 -->
						<view class="data" style="margin-right: 5px;">
							<image src="../../static/images/fan.png" style="width: 60rpx; height: 60rpx; margin-top:10px;"></image>
							<view class="status1">
								<view class="status1-tip">自动化状态:</view>
								<view class="status">
									<image v-if="isClosed3" src="../../static/images/close.png" ></image>
									<image v-else src="../../static/images/open.png" ></image>
								</view>
							</view>
							<view class="status1">
								<view class="status1-tip">风扇状态:</view>
								<view class="status">
									<image v-if="isClosed4" src="../../static/images/close.png" ></image>
									<image v-else src="../../static/images/open.png" ></image>
								</view>
							</view>
							<view class="status1">
								<view class="status1-tip">当前挡位:  {{fan.fanLevel}}</view>
							</view>
						</view>
						
						<!-- 水泵状态 -->
						<view class="data" style="margin-right: 0px;">
							<image src="../../static/images/pump.png" style="width: 60rpx; height: 60rpx; margin-top:10px;"></image>
							<view class="status1">
								<view class="status1-tip">自动化状态:</view>
								<view class="status">
									<image v-if="isClosed5" src="../../static/images/close.png" ></image>
									<image v-else src="../../static/images/open.png"></image>
								</view>
							</view>
							<view class="status1">
								<view class="status1-tip">水泵状态:</view>
								<view class="status">
									<image v-if="isClosed6" src="../../static/images/close.png"></image>
									<image v-else src="../../static/images/open.png"></image>
								</view>
							</view>
							
							<!-- 水泵没有挡位状态，此处做占位的作用，放置排版错乱 -->
							<view class="status1">
								<view class="status1-tip"></view>
							</view>
							
						</view>
					</view>
				</view>
			</u-card>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				lightIntensity:"",  //实时光照强度
				temperature:"",  //实时温度
				humidity:"",  //实时湿度
				profilePhoto:"../../static/images/profile-photo.png",  //头像
				currentDate: "",  //当天时间
				titleSize:33,  //u-card的title字体大小
				isClosed1: true,  //LED灯自动化状态是否关闭
				isClosed2: true,  //LED灯手动开关状态是否关闭
				isClosed3: true,  //风扇自动化状态是否关闭
				isClosed4: true,  //风扇手动开关状态是否关闭
				isClosed5: true,  //水泵自动化状态是否关闭
				isClosed6: true,  //水泵手动开关状态是否关闭
				normal1:true,  //实时光照强度是否正常
				normal2:true,  //实时温度是否正常
				normal3:true,  //实时湿度是否正常
				timer: null, // 用于存储定时器的引用 
				MAX_LIGHT:600,  //光照最高异常值
				MIN_LIGHT:300,  //光照最低异常值
				MAX_TEMPERATURE:30,  //温度最高异常值
				MIN_TEMPERATURE:20,  //温度最低异常值
				MAX_HUMIDITY:70,  //湿度最高异常值
				MIN_HUMIDITY:60,  //湿度最低异常值
				list: [{  //轮播图
						image: "../../static/images/tobacco1.jpeg",
						title: ""
					},
					{
						image: "../../static/images/tobacco2.jpeg",
						title: ""
					}
				],
				now:{  //实时天气信息
					icon:"",  //天气图标
					temp:"",  //温度值
					text:"",  //天气情况描述
					windDir:"",  //风向
					windScale:"",  //风的等级
					humidity:""  //当前空气湿度
				},
				//LED灯状态
				light:{
					lightMode:"",  //LED灯手动开关状态 1为开,0为关
					lightLevel:"",  //LED灯挡位值 0-100
					LightStatus:""  //LED灯自动化开关状态 1为开,0为关
				},
				fan:{
					fanMode:"",  //风扇手动开关状态 1为开,0为关
					fanLevel:"",  //风扇挡位值 20-100
					FenStatus:""  //风扇自动化开关状态 1为开,0为关
				},
				pump:{
					pumpCtrlState:"",  //水泵手动开关状态 1为开,0为关
					pumpPowerState:"",  //水泵没有挡位值
					PumpStatus:""   //水泵自动化开关状态 1为开,0为关
				}
			}
		},
		
		onLoad() {
			this.getWeather();  //获取天气信息
			this.updateCurrentDate();  //获取当前时间
		},
		
		onShow(){
			this.startFetchingData(); // 页面展示时开始获取数据  
		},
		
		methods: {
			startFetchingData() {  
			    // 设置一个定时器，每1秒请求一次数据   模拟获取实时数据
			    this.timer = setInterval(() => {  
			        this.getLightIntensity();  //获取实时光照
					this.getTemperature();  //获取实时温度
					this.getHumidity();  //获取实时湿度
					this.getState();  //获取传感器状态
			    }, 1000);  
			},  
			
			// 停止获取数据
			stopFetchingData() {  
			    // 清除定时器  
			    clearInterval(this.timer);  
			    this.timer = null;  
			},  
			
			// 获取天气信息接口,使用了第三方API(和风天气)
			/*
			 *因谷歌浏览器不能获取地址信息
			 * 所以这里为方便演示，将获取地理位置的方法注释，设定经纬度为官方文档所给的重庆市渝北区的经纬度
			 * 若设备支持且有需要可去掉注释，获取所在地的实时天气
			 */
			getWeather(){
				const that=this;  //用that存储this,以防后续this指代的不是我们需要的对象
				//先获取位置信息,在获取天气信息
				// uni.getLocation({
				// 	type: "wgs84",  //返回gps坐标
				// 	success: (res) =>{
				// 		let longitude = res.longitude.toFixed(2);
				// 		let latitude = res.latitude.toFixed(2);
						uni.request({
							url: `https://devapi.qweather.com/v7/weather/now?location=106.51,29.60&key=448cf88b3f704b5daecf846a16463c19`,
							method: "GET",
							dataType: "json",
							success: (res) =>{  //获取成功后赋值
								that.now.icon=res.data.now.icon;
								that.now.temp=res.data.now.temp;
								that.now.text=res.data.now.text;
								that.now.windDir=res.data.now.windDir;
								that.now.windScale=res.data.now.windScale;
								that.now.humidity=res.data.now.humidity;
							},
							fail: (err) => {
								console.log("天气信息获取失败",err);  //后台打印失败信息
							}
						})
				// 	}
				// })
			},
			
			//获取当前时间
			updateCurrentDate() {  
			    const now = new Date();  
			    const year = now.getFullYear();  
			    const month = String(now.getMonth() + 1).padStart(2, "0"); // 月份是从0开始的，所以要加1，并用padStart补零  
			    const date = String(now.getDate()).padStart(2, "0"); // 用padStart补零  
			    const days = ["日", "一", "二", "三", "四", "五", "六"];  
			    const dayOfWeek = days[now.getDay()];  
			    this.currentDate = `${year}年${month}月${date}日  星期${dayOfWeek}`;  
			},
			
			// 获取实时温度
			getTemperature(){
				const token=uni.getStorageSync("login_user_token");  //获取用户token
				uni.request({
					url:"http://139.155.150.150:8070/secure/temperature",
					method:"GET",
					header:{
						"Authorization": token //自定义请求头信息
					},
					success: (res) => {
						if(res.data.code*1==200){
							this.temperature=res.data.data.temperature;
							if(this.temperature>this.MAX_TEMPERATURE || this.temperature<this.MIN_TEMPERATURE){  //对实时温度进行判断,若不是适宜温度,则变红,形成预警提醒用户
								this.normal2=false;
							}else{
								this.normal2=true;
							}
						}else{
							console.error("获取实时温度失败");
						}
					},
					fail: (err) => {
						console.error("获取实时温度接口请求失败",err);
					}
				})
			},
			
			// 获取实时光照
			getLightIntensity(){
				const token=uni.getStorageSync("login_user_token");
				uni.request({
					url:"http://139.155.150.150:8070/secure/light-intensity",
					method:"GET",
					header:{
						"Authorization": token 
					},
					success: (res) => {
						if(res.data.code*1==200){
							this.lightIntensity=res.data.data.lightIntensity;
							if(this.lightIntensity>this.MAX_LIGHT || this.lightIntensity<this.MIN_LIGHT){ //对实时光照强度进行判断,若不是适宜温度,则变红,形成预警提醒用户
								this.normal1=false;
							}else{
								this.normal1=true;
							}
						}else{
							console.error("获取实时光照强度失败");
						}
					},
					fail: (err) => {
						console.error("获取实时光照强度接口请求失败",err);
					}
				})
			},
			
			// 获取实时湿度
			getHumidity(){
				const token=uni.getStorageSync("login_user_token");
				uni.request({
					url:"http://139.155.150.150:8070/secure/humidity",
					method:"GET",
					header:{
						"Authorization":token  //自定义请求头信息
					},
					success: (res) => {
						if(res.data.code*1==200){
							this.humidity=res.data.data.humidity;
							if(this.humidity>this.MAX_HUMIDITY||this.humidity<this.MIN_HUMIDITY){  //对实时湿度进行判断,若不是适宜温度,则变红,形成预警提醒用户
								this.normal3=false;
							}else{
								this.normal3=true;
							}
						}else{
							console.error("获取实时湿度失败");
						}
					},
					fail: (err) => {
						console.error("获取实时湿度接口请求失败",err);
					}
				})
			},
			
			// 获取传感器开关及挡位的状态
			getState(){
				const token=uni.getStorageSync("login_user_token");
				uni.request({
					url:"http://139.155.150.150:8070/secure/get_device_state",
					method:"GET",
					header:{
						"Authorization":token  //自定义请求头信息
					},
					success: (res) => {
						if(res.data.code*1==200){
							this.light.lightMode=res.data.data.lightMode;
							this.light.lightLevel=res.data.data.light_level;
							this.fan.fanMode=res.data.data.fanMode;
							this.fan.fanLevel=res.data.data.fanLevel;
							this.pump.pumpCtrlState=res.data.data.pumpCtrlState;
							this.pump.pumpPowerState=res.data.data.pumpPowerState;
							this.light.LightStatus=res.data.data.LightStatus;
							this.fan.FenStatus=res.data.data.FenStatus;
							this.pump.PumpStatus=res.data.data.PumpStatus;
							 //对json数据中的0-1映射为true和false
							if(this.light.lightMode==1){ 
								this.isClosed2=false;
							}else if(this.light.lightMode==0){
								this.isClosed2=true;
							}
							if(this.fan.fanMode==1){
								this.isClosed4=false;
							}else if(this.fan.fanMode==0){
								this.isClosed4=true;
							}
							if(this.pump.pumpCtrlState==1){
								this.isClosed6=false;
							}else if(this.pump.pumpCtrlState==0){
								this.isClosed6=true;
							}
							if(this.light.LightStatus==1){
								this.isClosed1=false;
							}else if(this.light.LightStatus==0){
								this.isClosed1=true;
							}
							if(this.fan.FenStatus==1){
								this.isClosed3=false;
							}else if(this.fan.FenStatus==0){
								this.isClosed3=true;
							}
							if(this.pump.PumpStatus==1){
								this.isClosed5=false;
							}else if(this.pump.PumpStatus==0){
								this.isClosed5=true;
							}
						}else{
							console.log("传感器状态获取失败");
						}
					},
					fail: (err) => {
						console.error("获取传感器状态接口请求失败",err);
					}
				})
			}
		},
		
		onUnload() {
		    this.stopFetchingData(); // 页面卸载时停止获取数据  
		},
		
		onHide(){
			this.stopFetchingData(); // 页面隐藏时停止获取数据
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

	.welcome {
		margin-left: 20px; 
		font-weight:bold; 
		font-size: 17px;
	}
	
	.weather-card {  
	  display: flex;  
	  align-items: center;  
	  padding: 20rpx;  
	  border-radius: 20rpx;  
	  background-color: #fff;  
	  width: 92%;
	  margin: 0 auto;
	  opacity: 0.8;
	}  
	  
	.weather-icon {  
	  width: 70rpx;  
	  height: 70rpx;  
	  margin-right: 40rpx;  
	  margin-left: 20rpx;
	}  
	  
	.weather-info {  
	  flex: 1;  
	  white-space: normal; 
	}  
	  
	.weather-text,  
	.temp-text,  
	.wind-text,  
	.humidity-text {  
	  margin-bottom: 10rpx;  
	  font-weight: bold;
	  margin-left: 5px;
	} 
	 
	.u-card-wrap { 
		background-color: $u-bg-color;
		padding: 1px;
	}
		
	.u-body-item {
		font-size: 32rpx;
		color: #333;
		padding: 20rpx 10rpx;
	}
			
	.u-body-item image {
		width: 120rpx;
		flex: 0 0 120rpx;
		height: 120rpx;
		border-radius: 8rpx;
		margin-left: 12rpx;
	}
	
	.data{
		width: 100px;
		border-radius: 10px;
		flex: 1;
		text-align:center;
		
		image{
			width: 60rpx;
			height: 60rpx;
			margin-top: 10px;
		}
	}
	
	.inTime-normal{
		text-align: center;
		font-size: 10px;
		margin-top: 3px;
	}
	
	.inTime-abnormal{
		text-align: center;
		font-size: 10px;
		margin-top: 3px;
		color: #d81e06;
	}
	
	.status{
		height: 14px;  
		width: 33px;
		text-align: left;
			
		image {
			width: 10px;
			height: 10px;
			margin-top: -1px;
		}
	}
	
	.status1{
		display: flex;
		flex: 1;
		text-align: center;
		height: 14px;
		width: 100%;
		margin-top: 4px;
	}
	
	.status1-tip{
		text-align: center;
		font-size: 10px;
		margin-top: 3px;
		display: flex;
		padding-left: 10px;
		width: 80px;
	}
	
	.wrap {
		padding: 20rpx;
		border-radius: 20rpx;
		background-color: #fff;  
		width: 92%;
		height: 100%;
		margin: 0 auto;
		opacity: 0.8;
	}
</style>

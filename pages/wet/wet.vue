<template>
	<view>
		<!-- 近7日湿度趋势折线图 使用echarts -->
		<view style="opacity: 0.8;">
			<u-card :title="title" :title-size="titleSize">
				<view class="" slot="body">
					<view class="main"> 
						<div id="echarts" ref="dom" ></div>
					</view>
				</view>
			</u-card>
		</view>
		
		<!-- 当天截止现在湿度数据统计 -->
		<view class="stastics">
			<!-- 最小值 -->
			<view class="block1">
				<view class="light-value" style="color: deepskyblue;">{{stat.min}}%</view>
				<view class="value-lable">最低值</view>
			</view>
			
			<!-- 平均值 -->
			<view class="block1">
				<view class="light-value" style="color: forestgreen;">{{stat.avg}}%</view>
				<view class="value-lable">平均值</view>
			</view>
			
			<!-- 最大值 -->
			<view class="block2">
				<view class="light-value" style="color: red;">{{stat.max}}%</view>
				<view class="value-lable">最大值</view>
			</view>
		</view>
		
		<!-- 预警信息 -->
		<view style="opacity: 0.8;">
			<u-card :title="title1" :title-size="titleSize">
				<view class="" slot="body">
					<view class="u-body-item u-flex u-border-bottom u-col-between u-p-t-0" style="display: flex;">
						<view class="img">
							<image src="../../static/images/wet.png"></image>
						</view>
						<view class="yujing">
							<view style="margin-top: 0px;">土壤湿度： {{alert.alertData}}%</view>
							<view style="margin-top: 10px;">预警： {{alert.alertMessage}}</view>
							<view style="margin-top: 10px;">异常时间： {{alert.alertTime}}</view>
						</view>
					</view>
				</view>
			</u-card>
		</view>
		
		<!-- 水泵设备控制 -->
		<view style="opacity: 0.8;">
			<u-card :title="title2" :title-size="titleSize">
				<view class="" slot="body">
					<view class="u-body-item u-flex u-border-bottom u-col-between u-p-t-0">
						<view style="display: flex; margin-top: 5px; flex: 1;">
							
							<!-- 自动化开关 -->
							<view class="switch_wrap">
								<view class="switch">水泵自动化开关 </view>
								<view style="margin-left: 20px;">
									<u-switch v-model="checked" active-color="green" size="30" @change="changeAuto"></u-switch>
								</view>
							</view>
							
							<!-- 手动开关 -->
							<view class="switch_wrap">
								<view class="switch">水泵手动开关 </view>
								<view style="margin-left: 20px;">
									<u-switch v-model="checked1"  active-color="green" size="30" @change="changeSwitch"></u-switch>
								</view>
							</view>
							
						</view>
					</view>
					<view style="width: 100%; height: 8px;"></view>  
				</view>
			</u-card>
		</view>
		
		<!-- 为了使页面底部有点空余，不让最后一个卡片底部与页面底部贴合 -->
		<view style="width: 100%; height: 8px;"></view>  
	</view>
</template>

<script>
	import * as echarts from "echarts";
	export default {
		onLoad() {   
			// this.getTrends(); // 页面加载时开始获取近7日日均湿度数据
		}, 
		data() {
			return {
				stat:{  //当天截止现在温度统计数据
					min:"",  //最小值
					max:"",  //最大值
					avg:""  //平均值
				},
				title:"过去7日 日均湿度趋势图",  //趋势图标题
				title1:"预警",  //预警标题
				title2:"水泵控制区",  //水泵设备控制区标题
				checked: false,  //水泵自动化开关状态
				checked1: false,  //水泵手动开关状态
				trend:["","","","","","",""],  //近7日日均湿度数据
				timer: null, // 用于存储定时器的引用 
				alert:{  //预警信息
					alertData:"",  //预警数据
					alertMessage:"",  //预警信息说明
					alertTime:""  //上次预警时间
				},
				pump:{  //水泵开关与挡位状态
					pumpCtrlState:"",  //手动开关状态 1为开,0为关
					pumpPowerState:"",  //挡位
					autoStatus:""  //自动化开关状态 1为开,0为关
				},
				value: 0,  //挡位调节时滑动选择器的值
				titleSize: 33,  //u-card标题文字大小
				myCharts: {},   //echarts图表对象
				//折线图的数据设置
				option: {
					// x轴设置
				  xAxis: {
				       type: "category",
				       name:"距离今天的天数",
				       nameLocation: "middle",
				       nameGap: "30",
				       data: ["7日", "6日", "5日", "4日","3日","2日","1日"],
				       axisTick: {
				           show: true,
				           alignWithLabel: true
				       },
				       axisLabel: {
				           show: true,
				       },
				       axisLine: {
				            show: true,
				            lineStyle: {
				               color: "#456980 "
							},
				  		}
				   },
				   //y轴设置
				   yAxis: {
				       name:"日均湿度",
				       axisTick: {
				           alignWithLabel: true,
				       },
				       axisLine: {
				           show: true,
				           lineStyle: {
				               color: "#456980",
				           }
				       },
				       splitLine: {
				           show: true,
				           lineStyle: {
				               color: "#456980",
				           }
				       },
				       axisLabel: {
				           show: true,
				       }
				   },
				// 数据设置
				  series: [
				      {
							type: "line",
				            data: [63.21,68.91,73.22,66.89,58.21,64.22,69.51],
				  			label: {
				  				normal: {
				  					show: true,
				  					position: "top",
				  					formatter: "{c}",
				  					textStyle: {
				  						color: "#000",
				  						fontWeight: "normal",
				  						fontSize: "10",
				  					},
				  				}
				  			},
							// 阈值线设置
				  			markLine: {  
				  				silent:true,
				  				Symbol:["circle","arrow"],
				  		        data: [  
				  					{
										yAxis: 60,
				  						lineStyle: {
											width: 2,
											color: "#00ff00"
				  						},
				  						label: {
				  							show: true,
				  							color: "#00ff00",
				  							fontSize: "14",
				  							position: "end", //将警示值放在哪个位置，三个值“start”,"middle","end"  开始  中点 结束
				  							formatter: "{c}",
				  						}
				  					}, {
											yAxis: 70,
											lineStyle: {
												width: 2,
												color: "#ff0000"
											},
											label: {
												show: true,
												color: "#ff0000",
												fontSize: "14",
												position: "end", //将警示值放在哪个位置，三个值“start”,"middle","end"  开始  中点 结束
												formatter: "{c}",
											}
				  						}
				  				],  
								//线型设置
				  				lineStyle: {  
				  					type: "dashed" // 设置线型为虚线  
				  				}  
				  			},
						}
					],
					// 不同范围的线的颜色设置
				    visualMap: {
						type: "piecewise",
				        show: false,
				        seriesIndex: 0,
				        pieces: [{
				            gt: 70,
				            color: "#F2761D"
				        }, {
				            lt: 60,
				            color: "#605D5A"
				        }],
				        outOfRange: { // 在选中范围外 的视觉元素，这里设置在正常范围内的图形颜色
				            color: "#1890FF"
				        }
				    }
				}
			}
		},
		
		onShow(){
			this.startFetchingData(); // 页面展示时开始获取数据  
			this.setECharts();
		},
		
		methods:{
			startFetchingData() {
			    // 设置一个定时器，每1秒请求一次数据  模拟获取实时数据 
			    this.timer = setInterval(() => {  
			       this.getHumidityMath();
			       this.getPumpState();
			       this.getAlerts();
			    }, 1000);  
			},  
			
			stopFetchingData() {  
			    // 清除定时器  
			    clearInterval(this.timer);  
			    this.timer = null;  
			},
			
			//初始化echarts图表
			setECharts() {
			    //获取dom元素
			    let dom = this.$refs.dom;
			    this.myCharts = echarts.init(dom, null, {
					width: 300,   //初始设置画布宽度
					height: 235  //初始设置画布高度
				});
			    //设置echarts展示到对应的dom元素中
			    this.myCharts.setOption(this.option);
			},
			
			//获取当日截止当时的湿度数据统计值
			getHumidityMath(){
				const token=uni.getStorageSync("login_user_token");
				uni.request({
					url:"http://139.155.150.150:8070/secure/humidity-math",
					method:"GET",
					header:{
						"Authorization": token //自定义请求头信息
					},
					success: (res) => {
						if(res.data.code*1==200){
							//保留两位小数
							this.stat.min=res.data.data.min.toFixed(2);
							this.stat.avg=res.data.data.avg.toFixed(2);
							this.stat.max=res.data.data.max.toFixed(2);
						}else{
							console.log("获取最大、最小、平均值失败");
						}
					},
					fail: (err) => {
						console.error("获取最大、最小、平均值接口请求失败",err);
					}
				})
			},
			
			//获取水泵开关状态
			getPumpState(){
				const token=uni.getStorageSync("login_user_token");
				uni.request({
					url:"http://139.155.150.150:8070/secure/getPumpStatus",
					method:"GET",
					header:{
						"Authorization":token  //自定义请求头信息
					},
					success: (res) => {
						if(res.data.code*1==200){
							this.pump.pumpCtrlState=res.data.data.pumpCtrlState;
							this.pump.pumpPowerState=res.data.data.pumpPowerState;
							this.pump.autoStatus=res.data.data.autoStatus;
							// 将json数据中的0,1映射为true和false
							if(this.pump.pumpCtrlState==1){
								this.checked1=true;
							}else if(this.pump.pumpCtrlState==0){
								this.checked1=false;
							}
							if(this.pump.autoStatus==1){
								this.checked=true;
							}else if(this.pump.autoStatus==0){
								this.checked=false;
							}
							this.value=this.pump.pumpPowerState;
						}else{
							console.log("获取水泵状态失败");
						}
					},
					fail: (err) => {
						console.error("获取水泵状态接口请求失败",err);
					}
				})
			},
			
			//异步获取近7日日均温度
			async getTrends(){
				const token=uni.getStorageSync("login_user_token");
				try{
					const res=await uni.request({
						url:"http://139.155.150.150:8070/secure/humidity-week",
						method:"GET",
						header:{
							"Authorization":token  //自定义请求头信息
						}
					});
					if(res[1].data && res[1].data.code==200){
						this.trend[0]=res[1].data.data[6].soilHumidity.toFixed(2);
						this.trend[1]=res[1].data.data[5].soilHumidity.toFixed(2);
						this.trend[2]=res[1].data.data[4].soilHumidity.toFixed(2);
						this.trend[3]=res[1].data.data[3].soilHumidity.toFixed(2);
						this.trend[4]=res[1].data.data[2].soilHumidity.toFixed(2);
						this.trend[5]=res[1].data.data[1].soilHumidity.toFixed(2);
						this.trend[6]=res[1].data.data[0].soilHumidity.toFixed(2);
						this.option.series[0].data=this.trend;
						this.setECharts();
					}else {  
						// 处理请求失败的情况  
						uni.showToast({ title: "获取数据失败", icon: "none" });  
						console.log("获取7日日均湿度数据失败");
					}  
				}catch(error){
					console.error("获取数据出错：", error); 
					uni.showToast({ title: "网络错误", icon: "none" });  
				}
			},
			
			// 获取预警信息
			getAlerts(){
				const token=uni.getStorageSync("login_user_token");
				uni.request({
					url:"http://139.155.150.150:8070/secure/alert-info",
					method:"GET",
					header:{
						"Authorization":token  //自定义请求头信息
					},
					data:{
						alertType:"2"
					},
					success: (res) => {
						if(res.data.code*1==200){
							this.alert.alertData=res.data.data.alertData;
							this.alert.alertMessage=res.data.data.alertMessage;
							this.alert.alertTime=res.data.data.alertTime;
							//对获取的时间进行处理,变为一般时间格式
							const transedTime=this.formatISO8601Date(this.alert.alertTime);
							this.alert.alertTime=transedTime;
						}else{
							console.log("获取预警信息失败");
						}
					},
					fail: (err) => {
						console.error("获取预警信息接口请求失败",err);
					}
					
				})
			},
			
			//监测水泵手动开关按钮行为,改变时调用接口上传状态
			changeSwitch(e){
				this.checked1=e;
				if(this.checked1==false){
					this.checked=false;
					this.updateSwitchStatus(0);
				}else{
					this.checked=false;
					this.updateSwitchStatus(1);
				}
			},
			
			//监测水泵自动化按钮行为,改变时调用接口上传状态
			changeAuto(e){
				this.checked=e;
				this.updateAutoSwitchStatus();
			},
			
			// 更新水泵手动开关状态
			updateSwitchStatus(status){
				const token=uni.getStorageSync('login_user_token');
				let url=`http://139.155.150.150:8070/secure/Pumpctl?p_c_state=${status}`;
				uni.request({
					url:url,
					method:"GET",
					header:{
						'Authorization':token  //自定义请求头信息
					},
					success: (res) => {
						if(res.data.code*1==200){
							console.log('水泵手动开关状态改变的状态发送成功');
						}else{
							console.log('水泵手动开关状态改变的状态发送失败');
						}
					},
					fail: (err) => {
						console.error('更新水泵手动开关接口请求失败',err);
					},
				})
			},
			
			// 更新水泵自动化开关状态
			updateAutoSwitchStatus(){
				const token=uni.getStorageSync("login_user_token");
				let url=`http://139.155.150.150:8070/secure/Get_Pump_auto`;//修改
				uni.request({
					url:url,
					method:"GET",
					header:{
						"Authorization":token  //自定义请求头信息
					},
					success: (res) => {
						if(res.data.code*1==200){
							console.log("改变的水泵自动化状态发送成功");
						}else{
							console.log("改变的水泵自动化状态发送失败");
						}
					},
					fail: (err) => {
						console.error("提交水泵自动化状态改变情况接口请求失败",err);
					},
				})
			},
			
			//处理ISO8601时间格式
			formatISO8601Date(isoString) {
			  // 使用Date对象解析ISO 8601格式的字符串  
			  const date = new Date(isoString);  
			  
			  // 获取年、月、日、时、分、秒  
			  const year = date.getFullYear();  
			  const month = String(date.getMonth() + 1).padStart(2, "0"); // 月份从0开始，所以需要+1  
			  const day = String(date.getDate()).padStart(2, "0");  
			  const hours = String(date.getHours()).padStart(2, "0");  
			  const minutes = String(date.getMinutes()).padStart(2, "0");  
			  const seconds = String(date.getSeconds()).padStart(2, "0");  
			  
			  // 拼接成新的日期时间字符串  
			  const formattedDate = `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;  
			  return formattedDate;  
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

<style>
	page{
		/* 设置背景图片 */  
		background-image: url("../../static/images/background-image.jpg"); 
		/* 背景图片覆盖整个容器 */  
		background-size: cover;  
		/* 根据需要设置背景位置 */  
		background-position: center; 
	}

	#echarts {  
		justify-content: center;
		align-items: center;
		height: 200px;
		width:100% ;
		margin-top: -40px; 
	} 
	
	.stastics {
		width: 92%;
		height: 100px;
		margin: 0 auto;
		border-radius: 20rpx;
		display: flex;
		flex:1;
		background-color:white;
		opacity:0.8;		
	}
	
	.block1 {
		width: 33.3%;
		height: 75%;
		border-right: solid;
		text-align: center;
		margin-top: 15px;
		border-color: gainsboro;
	}
	
	.block2 {
		width: 33.3%;
		height: 75%;
		text-align: center;
		margin-top: 15px;
	}
	
	.light-value {
		font-size: 21px;
	}
	
	.value-lable {
		font-size: 13px;
		margin-top:18px;
		font-weight: bold;
	}
	
	.u-body-item {
		padding: 20rpx 10rpx;
	}
		
	.u-body-item image {
		width: 120rpx;
		flex: 0 0 120rpx;
		height: 100rpx;
		border-radius: 8rpx;
		margin-left: 12rpx;
	}
	
	.img {
		width: 25%;
		height: 100%;
		text-align: center;
		
		image{
			width: 100rpx;
			height: 100rpx;
			margin-top: 0px;
			margin-left: 2px;
		}
	}
	
	.yujing {
		margin-left: 20px;
		text-align: left;
		width: 75%;
		font-size: 12px;
	}
	
	.switch_wrap{
		display: flex;
		width: 50%;
		text-align: center;
	}
	
	.switch {
		font-size: 12px;
	}
</style>

 <!-- 点击定位跳转到用户当前位置，点击停车将会根据地图中心点来就近获取停车场，点击列表将定位到停车场（这时候再点击停车会得到当前停车场距离为0，看似睿智实际影响不大，因为目的地必须离停车场较近） -->
<template>
 	<view style="display: flex; flex-direction: column;">
 		<view style="display: flex; flex-direction: row; flex: 1;">
			<image src='../../static/R-C.jpeg' class="avatar" @click="toInfo"></image>
			<text >{{username}}</text>
 			<button class="button" @click="update_location">定位</button>
			<navigator url="../order/order"><button class="button">查看订单</button></navigator>
 			<button class="button" @click="update_park2">停车</button>
 		</view>
 		
 		<view class="datetime-line" style="flex: 1;">
 			<view class="datetime-ceil">
 				<picker mode="date" :value="date1" :start="date1_start" :end="date1_end"
 					@change="ChangeDate1">
 					<view class="uni-input">{{date1}}</view>
 				</picker>
 				
 				<picker mode="time" :value="time1" 
 					@change="ChangeTime1">
 					<view class="uni-input">{{time1}}</view>
 				</picker>
 			</view>
 			
 			<view style="display: flex; flex: 1; justify-content: center;"><text>至</text></view>
 			
 			<view class="datetime-ceil">
 				<picker mode="date" :value="date2" :start="date2_start" :end="date2_end"
 					@change="ChangeDate2">
 					<view class="uni-input">{{date2}}</view>
 				</picker>
 				<picker mode="time" :value="time2" 
 					@change="ChangeTime2">
 					<view class="uni-input">{{time2}}</view>
 				</picker>
 			</view>
 		</view>
 		
 		<map id="Mymap"
 			style="width: 750rpx; height: 750rpx;"
 			scale="16" 
 			
 			show-compass="true"
 			enable-rotate="true"
 			enable-overlooking="true"
 			enable-traffic="true"
 			enable-poi="true"
 			enable-building="true"
 			show-location="true"
 			
 			:markers="parklot" 
 			:circles="circle"
 			
 			@markertap="markertap">
 			
 		</map>
 		
 		<scroll-view scroll-y="true" style="height: 420rpx;">
 			<view class="cell" v-for="(item,index) in parklot" @click="handle(item)" :key="index" >
 				<text>
 					{{item.description}}
 					{{item.category}}
 					剩余车位：{{item.space_count}}辆
 					距我：{{item.distance}}km
 				</text>
 			</view>
 		</scroll-view>
 		
 	</view>
 </template>
 
 <script>
 	function initDate(type) {//返回格式化Date
 		const date = new Date();
 		let year = date.getFullYear();
 		let month = date.getMonth() + 1;
 		let day = date.getDate();
 	
 		if (type === 'end') {//最多提前一年预约
 			year = year + 1;
 		}
 		month = month > 9 ? month : '0' + month;;
 		day = day > 9 ? day : '0' + day;
 	
 		return `${year}-${month}-${day}`;
 	};
 	function initTime() {//返回格式化Date
 		const date = new Date();
 		let hour = date.getHours();
 		let minute = date.getMinutes()
 
 		hour = hour > 9 ? hour : '0' + hour;;
 		minute = minute > 9 ? minute : '0' + minute;
 	
 		return `${hour}:${minute}`;
 	};
 	export default {
 		data() {
 			return {
 				date1:initDate(),
 				time1:initTime(),
 				date2:initDate(),
 				time2:initTime(),
 				date1_start:initDate(),
 				date1_end:initDate('end'),
 				date2_start:initDate(),
 				date2_end:initDate('end'),
 				
 				token:'',
 				parkingLot:[],//停车场全信息
 				parklot:[],//停车场展示信息（按距离排序）     id等价于该停车场在parkingLot中的下标
 				circle:[],//这是一个圆
 				begTime:'',//时间段
 				endTime:'',
				username:''
 			}
 		},
 		onShow() {//定位+定时
 			var that=this;
 			uni.getStorage({
 				key:'token',
 				success(res) {
 					that.token=res.data;
 					//console.log(that.token);
 					that.update_location();
 				},
 			})
 			uni.getStorage({
 				key:'userId',
 				success(res) {
 					that.userId=JSON.stringify(res.data);
 					//console.log(that.userId);
 				},
 			})
 			that.update_datetime();
			uni.getStorage({
				key:'username',
				success(res){
					that.username=JSON.parse(res.data)
				}
			})
 		},
 		methods: {
 			handle(item) {//跳转到停车场位置
 				console.log(item.id);
				console.log(item.description)
 				var idx=item.id-1;
 				var that=this;
 				// if(that.parkingLot[idx].space_count==0){
 				// 	uni.showToast({
 				// 		title:'无空位',
 				// 		icon:'error'
 				// 	})
 				// 	return;
 				// }
 				
 				var X=that.parkingLot[idx].latitude;
 				var Y=that.parkingLot[idx].longitude;
 				
 				that._map=uni.createMapContext("Mymap",that);
 				that._map.moveToLocation({
 					latitude:X,
 					longitude:Y,
 					success(r) {
 						//console.log('获取位置成功');
 					},
 					fail(){
 						console.log('定位位置失败');
 					},
 				})
 			},
 			update_datetime(){//更新时间段
 				this.begTime = new Date(this.date1+" "+this.time1+":00");
 				this.endTime = new Date(this.date2+" "+this.time2+":00");
 				// console.log(this.begTime);
 				// console.log(this.endTime);
 			},
 			ChangeDate1(e){
 				this.date1=e.detail.value;
 				this.update_datetime();
 			},
 			ChangeDate2(e){
 				this.date2=e.detail.value;
 				this.update_datetime();
 			},
 			ChangeTime1(e){
 				this.time1=e.detail.value;
 				this.update_datetime();
 			},
 			ChangeTime2(e){
 				this.time2=e.detail.value;
 				this.update_datetime();
 			},
			
			
 			
 			markertap(e){//点击停车场图标弹出子窗口
 				var that=this;
 				var idx=e.detail.markerId-1;
 				console.log(that.parkingLot)
				console.log(e)
 				if(that.parkingLot[idx].space_count==0){
 					uni.showToast({
 						title:'无空位',
 						icon:'error'
 					})
 					return;
 				}
 				
 				uni.$emit('page-popup', {
 					title: '(id:'+(idx+1)+")"+that.parkingLot[idx].description,
 					content: '距我：'+that.parkingLot[idx].distance+'km',
 					begTime: that.begTime,
 					endTime: that.endTime,
 					parking_lot_id: that.parkingLot[idx].parking_lot_id,
 					token: that.token,
 					userId: that.userId,
 				});
 				const subNVue = uni.getSubNVueById("sub_park")
 				subNVue.show('slide-in-top', 250,()=>{
 					console.log('子窗体打开成功');
 				})
 			},
 			
 			Rad(d) {
 				return d * Math.PI / 180.0; //经纬度转换成三角函数中度分表形式。
 			},
 			getMapDistance(lat1, lng1, lat2, lng2) {
 				/*
 				 计算距离，参数分别为第一点的纬度，经度；第二点的纬度，经度
 				 默认单位km
 				*/
 				var radLat1 = this.Rad(lat1);
 				var radLat2 = this.Rad(lat2);
 				var a = radLat1 - radLat2;
 				var b = this.Rad(lng1) - this.Rad(lng2);
 				var s = 2 * Math.asin(Math.sqrt(Math.pow(Math.sin(a / 2), 2) +
 					Math.cos(radLat1) * Math.cos(radLat2) * Math.pow(Math.sin(b / 2), 2)));
 				s = s * 6378.137; // EARTH_RADIUS;
 				s = Math.round(s * 10000) / 10000; //输出为公里
 				//s=s.toFixed(2);
 				return s;
 			},
 			update_location(){//定位到当前位置
 				var that=this;
 				uni.getLocation({
 					success(res) {
 						var X=res.latitude;
 						var Y=res.longitude;
 						that._map=uni.createMapContext("Mymap",that);
					
 						that._map.moveToLocation({
 							latitude:X,
 							longitude:Y,
 							success(r) {
 								console.log('获取位置成功');
 								
 							},
 							fail(){
 								console.log('定位位置失败');
 							},
 							complete(){
 								// that.circle=[];
 								// that.circle[0]={
 								// 					"latitude":X,
 								// 					"longitude":Y,
 								// 					"radius":300,
 								// 					"color":'#00aa00',
 								// 					"fillColor":'#ffffff'
 								// 				};
 							}
 						})
 						
 						// that.control[0]={
 						// 					"position":{
 						// 									"left":0,
 						// 									"top":0,
 						// 								},
 						// 					"iconPath":'/static/point.png',
 						// 				};
 					},
 					fail() {
 						console.log('获取位置失败');
 						uni.showToast({
 							title:'请重试',
 							icon:'loading',
 						})
 					}
 				})
 			},
 			/* update_park(){//返回停车场
 				var that=this;
 				that._map=uni.createMapContext("Mymap",that);
 				that._map.getCenterLocation({
 					success(r) {
 						//console.log(r);
 						var X=r.latitude;
 						var Y=r.longitude;
 						// that.circle=[];
 						// that.circle[0]={
 						// 					"latitude":X,
 						// 					"longitude":Y,
 						// 					"radius":300,
 						// 					"color":'#00aa00',
 						// 				};
 						uni.request({
 							url:'http://47.97.90.35:8080/findAllParkingLot',
 							method:'GET',
 							header:{token:JSON.parse(that.token)},//无语
 							success(res) {
 								console.log(res);
 								if(res.data.code!="200"){
 									console.log("停车场信息获取失败！");
 									return;
 								}
 								console.log("停车场信息获取成功！");
 								that.parkingLot=res.data.data;
 								//console.log(that.parkingLot);
 								//that.description=res.data.data[0].description
 								that.parklot=[];
 								for(var i=0;i<that.parkingLot.length;++i)
 								{
 									let p={}
 									let key='latitude'
 									let value=that.parkingLot[i].latitude
 									p[key] = value
 									key = 'longitude'
 									value = that.parkingLot[i].longitude
 									p[key]=value
 									key='description'
 									value=that.parkingLot[i].description
 									p[key]=value
 									key='callout'
 									value={
 												content: 'id:'+i+1,
 												borderRadius: 5,
 												display: "ALWAYS",
 												padding: 7,
 												bgColor: "#FFFFFF",
 											}
 									p[key]=value
 									key='distance'
 									value=that.getMapDistance(
 										X,Y,that.parkingLot[i].latitude,that.parkingLot[i].longitude)
 									p[key]=value
 									
 									that.parkingLot[i][key]=value
 									
 									key = 'iconPath'
 									value = '/static/point.png'
 									p[key]=value
 									key='id'
 									value=i
 									p[key]=value
 									
 									that.parklot.push(p)
 									//console.log(p)
 								}
 								that.parklot.sort(
 								(a,b)=>{
 									return (a.distance<b.distance) ? -1 : (a.distance>b.distance) ? 1 :0
 								}
 								)
 							},
 							fail() {
 								console.log('fail')
 							}
 						})
 					},
 					fail(){
 						console.log('定位失败');
 					},
 				})
 				
 			}, */
 			update_park2(){//根据时间段返回停车场
 				var that=this;
 				console.log(that.begTime);
 				console.log(that.endTime);
 				that.parklot=[];
 				that.parkingLot=[];
 				if(!that.begTime||!that.endTime){
 					uni.showToast({
 						title:'请选择停车时段',
 						icon:'error',
 					})
 					return;
 				}
 				if(that.begTime>=that.endTime){
 					uni.showToast({
 						title:'停车时段非法',
 						icon:'error',
 					})
 					return;
 				}
 				
 				var cur_time = new Date().getTime();
 				if(that.begTime.getTime()-cur_time<30*60*1000){
 					uni.showToast({
 						title:'预约时间过早',
 						icon:'error',
 					})
 					return;
 				}
 				
 				that._map=uni.createMapContext("Mymap",that);
 				that._map.getCenterLocation({
 					success(r) {
 						var X=r.latitude;
 						var Y=r.longitude;
 						// that.circle=[];
 						// that.circle[0]={
 						// 					"latitude":X,
 						// 					"longitude":Y,
 						// 					"radius":300,
 						// 					"color":'#00aa00',
 						// 				};
 						var d={'beginTime':that.begTime,'endTime':that.endTime};
 						uni.request({
 							url:'http://47.97.90.35:8080/parkingLot/checkAllAvailableParkingLotDuringPeriod',
 							method:'POST',
 							header:{token:JSON.parse(that.token)},//无语
 							data:{'beginTime':that.begTime,'endTime':that.endTime},
							complete(res) {
								console.log(res)
							},
 							success(res) {
 								console.log(res);
 								if(res.data.code!="200"){
 									console.log("停车场信息获取失败！");
 									return;
 								}
 								console.log("停车场信息获取成功！");
 								that.parkingLot=res.data.data;
 								console.log(that.parkingLot.length);
 								for(let i=0;i<that.parkingLot.length;i++)
 								{
									console.log(that.parkingLot[i].parking_lot_id)
 									let p={}
 									let key='latitude'
 									let value=that.parkingLot[i].latitude
 									p[key] = value
 									key = 'longitude'
 									value = that.parkingLot[i].longitude
 									p[key]=value
 									
 									key='space_count'
 									value=that.parkingLot[i].space_count
 									p[key]=value
 									
 									key='category'
 									value=that.parkingLot[i].category
 									p[key]=value
 									
 									key='description'
 									value=that.parkingLot[i].description
 									p[key]=value
 									
 									key='callout'
 									value={
 												content: ''+that.parkingLot[i].description,
 												borderRadius: 5,
 												display: "ALWAYS",
 												padding: 7,
 												bgColor: "#FFFFFF",
 											}
 									p[key]=value
 									
 									key='distance'
 									value=that.getMapDistance(
 										X,Y,that.parkingLot[i].latitude,that.parkingLot[i].longitude)
 									p[key]=value
 									
 									that.parkingLot[i][key]=value//停车场扩展信息
 									
 									key = 'iconPath'
 									value = '/static/parking.png'
 									p[key]=value
 									
 									key='id'
 									value=that.parkingLot[i].parking_lot_id
 									p[key]=value
 									
 									that.parklot.push(p)
 								}
 								
 								that.parklot.sort(
 								(a,b)=>{
 									if(a.space_count==0){
 										if(b.space_count==0) return (a.distance<b.distance) ? -1 : (a.distance>b.distance) ? 1 :0;
 										else return 1;
 									}
 									else if(b.space_count==0) return -1;
 									else return (a.distance<b.distance) ? -1 : (a.distance>b.distance) ? 1 :0;
 								}
 								)
								console.log(that.parklot)
								console.log('排序成功')
 								
 							},
 							fail() {
 								console.log('fail')
 							}
 						})
 					},
 					fail(){
 						console.log('定位失败');
 					},
 				})
 				
 			},
			toInfo(){
				uni.navigateTo({
					url:'../User/User'
				})
			}
 		}
 	}
 		
 	
 </script>
 
 <style>
	 
 	.coverview {
 	    position: absolute;
 	    left: 0;
 	    right: 0;
 	    top: 0rpx;
 	    height: 150rpx;
 	    border-width: 10rpx;
 	    border-color: #4CD964;
 	}
 	
 	.example-body {
 		background-color: #fff;
 		padding: 10px;
 	}
 	
 	.datetime-ceil{
 		display: flex; 
 		flex-direction: row;
 		flex: 2;
 		justify-content: space-between;
 	}
 	
 	.datetime-line{
 		display: flex; 
 		flex-direction: row;
 		justify-content: space-around;
 		padding: 35rpx;
 	}
 	
 	.cell {
 		margin: 10rpx;
 		padding: 20rpx 0;
 		top: 10rpx;
 		align-items: center;
 		justify-content: center;
 		border-radius: 10rpx;
 		background-color: #ebebeb;
 	}
	.avatar{
		padding: 25rpx;
		width: 50rpx;
		height: 50rpx;
	}
	.button{
		padding: 0rpx;
		width: 250rpx;
		height: 100rpx;
		font-size: medium;
	}
 </style>
 

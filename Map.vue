<template>
	<view>
		<map id="Mymap"  style="width:750rpx ; height: 750rpx;"latitude="32" longitude="118.7" scale="11" show-location=true :markers="parklot" @callouttap="callouttap"></map>
		<view v-for="(item,index) in parklot" :key="index" >
			<button @click="openOrder" :data-id="item.id">id:{{item.id}}{{item.description}}{{item.distance}}km</button>
			
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				id:0,
				token:'',
				latitude:28,
				longitude:104,
				parkingLot:[],//停车场经纬度信息
				parklot:[],//停车场完整信息（按距离排序）
				description:'',
				distance:'',
				spare:'',
				charge:[]
			}
		},
		methods: {
			callouttap(e){
				console.log(e)
			},
			openOrder(i){
				console.log(i)
				var id=i.currentTarget.dataset.id
				uni.redirectTo({
					url:'../order/order?id='+id
				})
			},
			Rad(d) {
								return d * Math.PI / 180.0; //经纬度转换成三角函数中度分表形式。
							},
							/*
							 计算距离，参数分别为第一点的纬度，经度；第二点的纬度，经度
							 默认单位km
							*/
							getMapDistance(lat1, lng1, lat2, lng2) {
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
			
		},
		onLoad(e) {	
			var that=this;
			console.log(e);
			let k=JSON.parse(e.data);
			this.latitude=k.latitude;
			this.longitude=k.longitude;
			uni.getStorage({
				key:'token',
				success(r) {
					that.token=r.data
					uni.request({
						url:'http://47.97.90.35:8080/findAllParkingLot',
						method:'GET',
						header:{token:JSON.parse(that.token)},
						success(res) {
							console.log(res.data.data)
							that.parkingLot=res.data.data
							that.description=res.data.data[0].description
							for(var i=0;i<that.parkingLot.length;i++)
							{
								let p={}
								let key='latitude'
								let value=that.parkingLot[i].latitude
								p[key] = value
								key = 'longitude'
								value = that.parkingLot[i].longitude
								p[key]=value
								key = 'iconPath'
								value = '/static/point.png'
								p[key]=value
								key='id'
								value=i
								p[key]=value
								key='callout'
								value={content: 'id:'+i,
											borderRadius: 5,
											display: "ALWAYS",
											padding: 7,
											bgColor: "#FFFFFF"}
								p[key]=value
								key='distance'
								value=that.getMapDistance(that.latitude,that.longitude,that.parkingLot[i].latitude,that.parkingLot[i].longitude)
								p[key]=value
								key='description'
								value=res.data.data[i].description
								p[key]=value
								that.parklot.push(p)
								
								console.log(p)
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
				complete() {
					console.log('complete')
				}
				
			})
		},
		}
		
	
</script>

<style>

</style>

	<template>
		<view>
			<!-- <text >{{description}}\n{{distance}}km</text>
			<text>\n计费规则：\n<text v-for="item in chargeRule" :key="item.id">
				停{{ item.val }}天: {{item.unit_price}} 元/天\n
			</text></text>
			<button @click="navigation">去这里</button>
			
				<uni-datetime-picker type="datetime" return-type="date" @change="setBegTime">选择停车开始时间\n{{begTime}}</uni-datetime-picker>
				<uni-datetime-picker type="datetime" @change="setEndTime" return-type="date">选择停车结束时间\n{{endTime}}</uni-datetime-picker>			
			<button @click="makeOrder">下单</button> -->
			<button @click="orderView">查看订单</button>
			<div v-for="item in currentOrder" :key="item.id">
			{{ item.beginTime }}至{{item.endTime}} {{parkingLot[item.parkingLotId].description}} {{status[item.cancelFlag]}}
			<button @click="orderOperate" :data-info='item'>{{operate[item.cancelFlag]}}</button>
			</div>
			
		</view>
	</template>

	<script>
		export default {
			data() {
				return {
					id:'',
					token:'',
					chargeRule:[],
					distance:'',
					description:'',
					latitude:'',
					longitude:'',
					begTime:'',
					endTime:'',
					currentOrder:[],
					parkingLot:[],
					status:['待使用','已取消'],
					operate:['取消','恢复'],
					
					
					
				}
			},
			onLoad(e) {
				var that = this
				console.log(e)
				let k = JSON.parse(e.parkInfo)
				that.id=k.id
				that.distance=k.distance
				that.description=k.description
				that.latitude=k.latitude
				that.longitude=k.longitude
				uni.getStorage({
					key:'token',
					success:function(r)
					{
						that.token = r.data
						uni.request({
							url:'http://47.97.90.35:8080/findChargeRuleByParkingLotId/'+k.id,
							header:{token:JSON.parse(that.token)},
							method:'GET',
							success:function(res){
								console.log(res.data.data)
								that.chargeRule=res.data.data
								
							}
						})
						console.log('success')
					}
				})
				
			},
			methods: {
			/* 	navigation:function()
				{
					uni.openLocation({
						latitude:this.latitude,
						longitude:this.longitude,
						success() {
							console.log('success')
						}
					})
				},
				setBegTime:function(e){
					this.begTime=e
					console.log(this.begTime)
					console.log(this.id)
				},
				setEndTime:function(e){
					this.endTime=e
					console.log(this.endTime)
				},
				makeOrder:function(){
					uni.request({
						url:'http://47.97.90.35:8080/makeOneOrder',
						header:{token:JSON.parse(this.token)},
						method:'POST',
						data:{'parkingLotId':this.id,'beginTime':this.begTime,'endTime':this.endTime},
						success(res) {
							console.log(res)
							uni.showToast({
								title:'预订成功！'
							})
						
						},
						fail() {
							console.log('fail')
						}
						
						
					})
				}, */
				orderView:function(){
					var that = this
					uni.request({
						url:'http://47.97.90.35:8080/findAllParkingLot',
						method:'GET',
						header:{token:JSON.parse(that.token)},
						success(res) {
							that.parkingLot=res.data.data
						}
					}),
					uni.request({
						url:'http://47.97.90.35:8080/findOrdersByUserId/9',
						method:'GET',
						header:{token:JSON.parse(that.token)},
						success(res) {
							console.log(res)
							that.currentOrder=res.data.data
						}
						
					})
				},
				orderOperate:function(e){
					console.log(e)
					var that = this
					var orderId = e.currentTarget.dataset.info.orderId
					if(!e.currentTarget.dataset.info.cancelFlag)
					{
						
						uni.request({
							url:'http://47.97.90.35:8080/cancelOneOrderById/'+orderId,
							method:'GET',
							header:{token:JSON.parse(that.token)},
							success(res) {
								console.log(res)
								if(res.data.code==200)
								{
									uni.showToast({
										title:'取消成功'
									}),
									uni.request({
										url:'http://47.97.90.35:8080/findOrdersByUserId/9',
										method:'GET',
										header:{token:JSON.parse(that.token)},
										success(res) {
											console.log(res)
											that.currentOrder=res.data.data
										}
										
									})
									
								}
							},
							fail() {
								uni.showToast({
									title:'网络错误！',
									icon:'error',
								})
		
							}
							
							
						})
					}
					else
					{
						uni.request({
							url:'http://47.97.90.35:8080/restoreOneOrderById/'+orderId,
							header:{token:JSON.parse(that.token)},
							method:'GET',
							success(res) {
								console.log(res)
								if(res.data.code==200)
								{
									uni.showToast({
										title:'恢复成功'
									}),
									uni.request({
										url:'http://47.97.90.35:8080/findOrdersByUserId/9',
										method:'GET',
										header:{token:JSON.parse(that.token)},
										success(res) {
											that.currentOrder=res.data.data
										}
										
									})
								}
								
							},
							fail() {
								uni.showToast({
									title:'网络错误！',
									icon:'error',
								})
									
							}
							
						})
					}
					
				}
				
				
			}
		}
	</script>

	<style>

	</style>

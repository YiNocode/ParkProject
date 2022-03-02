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
			{{ item.beginTime }}至{{item.endTime}} {{parkingLot[item.parkingLotId-1].description}} 
				<text class="stautes"> {{status[item.cancelFlag]}}</text>
			<button class="button" @click="orderOperate" :data-info='item'>{{operate[item.cancelFlag]}}</button>
			</div>
			
		</view>
	</template>

	<script>
		export default {
			data() {
				return {
					id:'',
					token:'',
					description:'',
					begTime:'',
					endTime:'',
					currentOrder:[],
					parkingLot:[],
					status:['待使用','已取消'],
					operate:['取消','删除']
					
				}
			},
			onLoad(e) {
				var that = this
				uni.getStorage({
					key:'token',
					success:function(r)
					{
						that.token = r.data
						console.log('success')
					}
				})
				uni.getStorage({
					key:'userId',
					success(r) {
						that.id=r.data
						console.log(that.id)
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
						url:'http://47.97.90.35:8080/parkingLot/findAllParkingLot',
						method:'GET',
						header:{token:JSON.parse(that.token)},
						success(res) {
							that.parkingLot=res.data.data
							that.parkingLot.sort(function(a,b){
								return (a.parking_lot_id<b.parking_lot_id) ? -1 : (a.parking_lot_id>b.parking_lot_id) ? 1 :0;
							})
							console.log(that.parkingLot)
						}
					}),
					uni.request({
						url:'http://47.97.90.35:8080/order/findOrdersByUserId/'+that.id,
						method:'GET',
						header:{token:JSON.parse(that.token)},
						success(res) {
							console.log(res)
							that.currentOrder=res.data.data
							if(that.currentOrder.length==0)
							{
								uni.showToast({
									title:'您当前暂无订单',
									icon:'none'
								})
							}
						}
						
					})
				},
				orderOperate:function(e){
					console.log(e)
					var that = this
					var orderId = e.currentTarget.dataset.info.orderId
					if(!e.currentTarget.dataset.info.cancelFlag)
					{
						uni.showModal({
								content:'确认要取消此订单吗',
								success(res)
								{
									if(res.confirm)
									{
										uni.request({
											url:'http://47.97.90.35:8080/order/cancelOneOrderById/'+orderId,
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
														url:'http://47.97.90.35:8080/order/findOrdersByUserId/'+that.id,
														method:'GET',
														header:{token:JSON.parse(that.token)},
														success(res) {
															console.log(res)
															that.currentOrder=res.data.data
															console.log('已更新')
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
								
						})
						
						
					}
					else
					{
						uni.showModal({
							content:'确认删除此订单信息吗？',
							success(res){
								if(res.confirm)
								{
									uni.request({
										url:'http://47.97.90.35:8080/order/deleteOrderById/'+orderId,
										header:{token:JSON.parse(that.token)},
										method:'DELETE',
										success(res)
										{
											uni.showToast({
												title:'删除成功'
											}),
											uni.request({
												url:'http://47.97.90.35:8080/order/findOrdersByUserId/'+that.id,
												method:'GET',
												header:{token:JSON.parse(that.token)},
												success(res) {
													console.log(res)
													that.currentOrder=res.data.data
													console.log('已更新')
												}
												
											})
										},
										fail() {
											uni.showToast({
												title:'删除失败',
												icon:'error'
											})
											
										}
									})
								}
	
							}
						})
					}
			
					
				}
				
				
			}
		}
	</script>

	<style>
	.button{
		display: flex;
		justify-content: center;
		align-items: center;
		margin: 50rpx;
		background-color: #55aaff;
		font:'Gill Sans', 
	}
	.stautes{
		margin-left: 200rpx;
		background-color:#ff0000;
		color: #ffffff;
		font:'Gill Sans', 
	}
	</style>

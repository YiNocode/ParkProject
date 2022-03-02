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
			
			<uni-list>
				<uni-list-item :title="username"  thumb="/static/R-C.jpeg"></uni-list-item>
				<button style="color: #2979FF;" @click="orderView">我的订单</button>
				<div  v-for="item in currentOrder" :key="item.id">
				<uni-list-item>
					<text slot="body">
						预计开始时间：{{ item.beginTime }}
						预计结束时间：{{item.endTime}} 
					    地点：{{parkingLot[item.parkingLotId-1].description}}
						当前状态：<text v-if="Date.parse(item.endTime) > new Date() " class="stautes"> {{status[item.cancelFlag]}}</text>
								<text v-else> 已过期</text>
					
					<button v-if="Date.parse(item.endTime) > new Date() " class="button" @click="orderOperate" :data-info='item'>{{operate[item.cancelFlag]}}</button></text>
				</uni-list-item>	
				</div>
			</uni-list>
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
					operate:['取消预约','删除订单'],
					username:'',
					
					
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
				uni.getStorage({
					key:'username',
					success(r)
					{
						that.username=r.data
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
		margin-left: 10rpx;
		background-color:#ff0000;
		color: #ffffff;
		font:'Gill Sans', 
	}
	</style>

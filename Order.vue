<template>
	<view>
		<text >{{description}}\n{{distance}}km</text>
		<text>\n计费规则：\n<text v-for="item in chargeRule" :key="item.id">
			停{{ item.val }}天: {{item.unit_price}} 元/天\n
		</text></text>
		<button @click="navigation">去这里</button>
		
			<uni-datetime-picker type="datetime" return-type="date" @change="setBegTime">选择停车开始时间\n{{begTime}}</uni-datetime-picker>
			<uni-datetime-picker type="datetime" @change="setEndTime" return-type="date">选择停车结束时间\n{{endTime}}</uni-datetime-picker>			
		<button @click="makeOrder">下单</button>
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
			navigation:function()
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
			}
			
			
		}
	}
</script>

<style>

</style>

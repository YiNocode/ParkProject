<template>
	<view>
		<map id="Mymap" :latitude="latitude" :longitude="longitude"  style="width:100% ; height: 1500rpx;" show-location=true :markers="parklot" ></map>
		
	</view>
</template>

<script>
	export default {
		onLoad(e) {
			console.log(e);
			let k=JSON.parse(e.data);
			this.latitude=k.latitude;
			this.longitude=k.longitude;
			this.parklot.latitude=k.latitude;
			this.parklot.longitude=k.longitude;
			console.log(this.parklot.latitude);			
			
		},
		data() {
			return {
				id:0,
				latitude:28,
				longitude:104,
				parklot:{
					latitude:28,
					longitude:104,
					iconPath:'../../static/point.png'
				},
			
				
				
				
				
			}
		},
		
		methods: {
			
		},
		}
		
	
</script>

<style>

</style>

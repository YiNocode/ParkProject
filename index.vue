<template>
	<view>
		<text>停车app</text>
		<text>用户名{{username}}</text>
		<button @click="LogOut">退出登录</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				username:'',
				token:'',
				
				
				
			}
		},
		onLoad() {
			var that=this
			uni.getStorage({
				key:'userInfo',
				fail() {
					uni.redirectTo({
						url:'../Login/Login'
					})
				},
				success:function(res){
					console.log('已登录')
					that.username=JSON.parse(res.data)
					console.log(that.username)
					
				}
			})
			uni.getStorage({
				key:'token',
				success:function(res){
					console.log(res)
					that.token=res.data
					
				}
			})
		},
		methods: {
			LogOut(){
				var that=this
				console.log(this.token)
				uni.request({
					url:'http://47.97.90.35:8080/logOut',
					method:'GET',
					header:{token:JSON.parse(that.token)},
					success(res) {
						console.log(res)
					}

				})
				uni.clearStorage();
				uni.redirectTo({
					url:'../Login/Login',
				})
			}
			
			
		}
	}
</script>

<style>

</style>

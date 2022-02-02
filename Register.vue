<template>
	<view class="screen">
		
		<form @submit="formSubmit">
			<input class="input" name="username" placeholder="请输入用户名" />
			<input class="input" name="password" placeholder="请输入密码" password=true/>
			<button class="button" form-type="submit" >登录</button>
		</form>
		<navigator class="button"  url="../Register/Register"><button>注册</button></navigator>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				token:'',
				isSuccess:false,
				latitude:0,
				longitude:0,
				
			}
		},
		onReady() {
			uni.getStorage({
				key:'userInfo',
				success:function(res)
				{
					uni.showToast({
						title:'您已登录',
						
					})
				}
			})
		},
		methods: {
			formSubmit:function(e){
				console.log("ok");
				uni.request({
					url:'http://47.97.90.35:8080/login',
					method:'POST',
					data:JSON.stringify(e.detail.value),
					success: res => {
						if(res.data.code=="200"){
							uni.showToast({
								title: '登录成功'
							});
							this.token=res.data.token;
							uni.setStorage({
								key:'userInfo',
								data:JSON.stringify(e.detail.value.username),
								complete(data) {
									console.log('complete')
									console.log(data)
								}
								
							})
							console.log(res)
							uni.setStorage({
								key:'token',
								data:JSON.stringify(res.data.data.token)
								
							})
							this.isSuccess=true;
							uni.getLocation({
								success(res) {
									uni.redirectTo({
										url:'../Map/Map?data='+JSON.stringify({latitude:res.latitude,longitude:res.longitude}) ,
										complete() {
											console.log(res.latitude)
											console.log(res.longitude)
										}
									})
								
								}
							})	
						}
						else{
							uni.showToast({
								title: '请检查用户名或密码',
								icon:"none"
							});
						}
					},
					fail() {
						uni.showToast({
							title:"网络错误，请检查网络"
						})
					}
				})
			},
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
	.input{
		margin: 70rpx;
	
	}
	
	.screen{
			
	}
	.button{
		margin: 50rpx;
		font:'Gill Sans', 
		
	}
	
</style>

<template>
	<view  >
	
		<image src="../../static/parking_PNG7.png" class="Logo"></image>
		
		<uni-forms ref="form" :modelValue="formData" class="input">
			<uni-forms-item name="username" >
				<uni-easyinput v-model="formData.username" placeholder="请输入用户名"  />
			</uni-forms-item>
			<uni-forms-item name="password">
				<uni-easyinput  v-model="formData.password" type="password" placeholder="请输入密码"/>
			</uni-forms-item>
		</uni-forms>
		<button class="button" @click="submit">登录</button>
		<navigator  url="../Register/Register"><button class="button" >注册</button></navigator>
		<image src="../../static/background.jpeg"style="margin: 40rpx;"></image>
		
		
	</view>
</template>

<script>
	export default {
		data() {
			return {
				formData:{
					username:'',
					password:'',
				},
			}
		},
		methods: {
			submit:function(){
				var that = this
				this.$refs.form.validate().then(res=>{
					// console.log(res);
					console.log(JSON.stringify(res));
					uni.request({
						url:'http://47.97.90.35:8080/login',
						method:'POST',
						data:JSON.stringify(res),
						success: r => {
							console.log(r);
							if(r.data.code=="200"){
								uni.showToast({
									title: '登录成功'
								});
								uni.setStorage({
									key:'username',
									data:res.username,
									success(data) {
										console.log('设置username完成')
										//console.log(data)
									}
								})
								uni.setStorage({
									key:'password',
									data:JSON.stringify(res.password),
									success(data) {
										console.log('设置password完成')
										//console.log(data)
									}
								})
								uni.setStorage({
									key:'token',
									data:JSON.stringify(r.data.data.token),
									complete(data) {
										console.log('设置token完成')
										console.log(this.username)
										uni.request({//获取用户信息
											url:'http://47.97.90.35:8080/user/findUserByUserName/'+that.formData.username,
											method:'GET',
											header:{token:r.data.data.token},//无语
											success: res => {
												//console.log(res);
												if(res.data.code=='200'){
													console.log("获取用户信息成功");
													console.log(res)
													uni.setStorage({
														key:'userId',
														data:res.data.data.userId,
														success(data) {
															console.log('设置userId成功');
															console.log('即将跳转');
															uni.navigateTo({
																url:'../Map/Map'
															})
														},
														fail() {
															console.log('登录失败');
														}
													})
												}else{
													console.log('登录失败');
												}
											},
											fail() {
												console.log('登录失败');
											},
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
				}).catch(err=>{
					console.log('no');
					console.log(err);
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
	.login{
		width: 100%;
		height: 100%;
		
	}
	
	
	.button{
		display: flex;
		justify-content: center;
		align-items: center;
		margin: 50rpx;
		background-color: #55aaff;
		font:'Gill Sans', 
		
		
	}
	.Logo{
		 margin-top: 50rpx;
		 margin-left: 300rpx;
		
		 width: 125rpx;
		 height: 125rpx;
	}
	
	
	
	.nvue-page-root {
	    background-color: #F8F8F8;
	    padding-bottom: 20px;
	}
	
	.page-title {
	    /* #ifndef APP-NVUE */
	    display: flex;
	    /* #endif */
	    flex-direction: row;
	    justify-content: center;
	    align-items: center;
	    padding: 35rpx;
	}
	
	.page-title__wrapper {
	    padding: 0px 20px;
	    border-bottom-color: #D8D8D8;
	    border-bottom-width: 1px;
	}
	
	.page-title__text {
	    font-size: 16px;
	    height: 48px;
	    line-height: 48px;
	    color: #BEBEBE;
	}
	
	.title {
	    padding: 5px 13px;
	}
	
	.uni-form-item__title {
	    font-size: 16px;
	    line-height: 24px;
	}
	
	.uni-input-wrapper {
	    /* #ifndef APP-NVUE */
	    display: flex;
	    /* #endif */
	    padding: 8px 13px;
	    flex-direction: row;
	    flex-wrap: nowrap;
	    background-color: #FFFFFF;
	}
	
	.uni-input {
	    height: 28px;
	    line-height: 28px;
	    font-size: 15px;
	    padding: 0px;
	    flex: 1;
	    background-color: #FFFFFF;
	}
	
	.uni-icon {
	    font-family: uniicons;
	    font-size: 24px;
	    font-weight: normal;
	    font-style: normal;
	    width: 24px;
	    height: 24px;
	    line-height: 24px;
	    color: #999999;
	}
	
	.uni-eye-active {
	    color: #007AFF;
	}
	
</style>

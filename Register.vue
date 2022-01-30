<template>
	<view>
		<uni-forms ref="form" :modelValue="formData" :rules="rules">
			<uni-forms-item name="username">
				<input  v-model="formData.username" placeholder="请输入用户名"  />
			</uni-forms-item>
			<uni-forms-item name="password">
				<input  v-model="formData.password" placeholder="请输入密码" password=true/>
			</uni-forms-item>
			<uni-forms-item name="passwordConfirm">
				<input  v-model="formData.passwordConfirm" placeholder="确认密码" password=true />
			</uni-forms-item>
		</uni-forms>
		<button @click="submit">注册</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				token:'',
				isSuccess:false,
				formData:{
					username:'',
					password:'',
					passwordConfirm:'',
				},
				rules:{
					username:{
						rules:[
							{
								required:true,
								errorMessage:"请输入用户名",
								
							},
							{
								maxLength:8,
								errorMessage:"用户名不能超过8个字符",
								
							},
							{
								minLength:3,
								errorMessage:"用户名不应少于3个字符"
							}
						]
					},
					password:{
						rules:[
							{
								required:true,
								errorMessage:"请输入密码",
							},
							{
								minLength:6,
								errorMessage:"密码应长于6个字符",
							},
							{
								maxLength:15,
								errorMessage:"密码不应长于15个字符"
							},
						]
					},
					passwordConfirm:{
						rules:[
							{
								required:true,
								errorMessage:"请确认密码",
							},
							{
								validateFunction:function(rule,value,data,callback){
									if(value!=data.password)
									{
										callback('两次密码输入不一致')
									}
									return true
								}
							}
						]
						
					}
				}
				
			}
		},
		onReady() {
			this.$refs.form.setRules(this.rules)
		},
		methods: {
			submit:function(){
				this.$refs.form.validate().then(res=>{
					uni.request({
						url:'http://47.97.90.35:8080/register',
						data:JSON.stringify(res),
						method:'POST',
						success(res) {
							if(res.data.code=="200")
							{
								uni.showToast({
									title:'注册成功',
								});
								this.token=res.data.data.token;
								this.isSuccess=true;
							}
							else{
								uni.showToast({
									title:'用户已存在',
									icon:'error'
								});
							}
							
						},
						fail(){
							uni.showToast({
								title:'网络错误，请重试',
								icon:'none'
							});
						}
					})
				}).catch(err=>{
					console.log(err);
				})
				
				
			}
			
		}
	}
</script>

<style>

</style>

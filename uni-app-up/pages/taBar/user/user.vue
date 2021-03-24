<template>
	<view class="user">
	  <!-- 头像 -->
	  <view class="left">
	  	 <image :src="user.face" mode=""></image>
	  </view>
	  <!-- 昵称 个性签名 -->
	  <view class="right">
	  	 <view class="username">{{user.username}}</view>
	  </view>
	  <view class="erweima" @tap="handleWRCode">
	  	<view class="icon iconfont">&#xe67c;</view>
	  </view>
	</view>
</template>
<script>
	export default{
		data(){
			return{
			   // 个人信息
			   user:{
				   username:"米斯特吴",
				   face:"/static/img/face.jpg"
			   }
			}
		},
		methods:{
			handleWRCode(){
				uni.navigateTo({
					url:"./prcode"
				})
			}
		}
		
	}
</script>

<style lang="scss">
	.user {
		width: 92%;
		padding: 3px 4%;
		display: flex;
		align-items: center;
		position: relative;
		top: -0.5px;
		background-color: #f06c7a;
		padding-bottom: 80upx;

		.left {
			width: 20vw;
			height: 20vw;
			flex-shrink: 0;
			margin-right: 20upx;
			border: solid 1upx #fff;
			border-radius: 100%;

			image {
				width: 20vw;
				height: 20vw;
				border-radius: 100%;
			}

		}

		.right {
			width: 100%;

			.username {
				font-size: 36upx;
				color: #fff;
			}
		}

		.erweima {
			flex-shrink: 0;
			width: 10vw;
			height: 10vw;
			margin-left: 5vw;
			border-radius: 100%;

			display: flex;
			justify-content: center;
			align-items: center;
			background: linear-gradient(to left, #fbbb37 0%, #fcf0d0 105%);

			.icon {
				color: #7b6335;
				font-size: 42upx;
			}
		}
	}
</style>

<template>
	<view> 
		<!-- 状态栏 -->
		<page-status></page-status>
		<!-- header -->
		<goodsHeader></goodsHeader>
		
		<!-- 轮播图 -->
		<view class="swiper-box">
			<swiper @change="swiperChange" circular="true" :autoplay="true">
				<swiper-item v-for="swiper in goodsData.swiperList" :key="swiper.id">
					<image :src="swiper.img" ></image>
				</swiper-item>
			</swiper>
			<view class="indicator">
				{{currentSwiper+1}}/{{goodsData.swiperList.length}}
			</view>
		</view>
		<!-- 标题和价格 -->
		<view class="info-box goods-info">
			<view class="price">¥{{goodsInfo.price}}</view>
			<view class="title">{{goodsInfo.name}}</view>
		</view>
		
		<!-- 选择规格 左边-->
		<view class="info-box">
			 <view class="row" @tap="spaceInfo.showSpace=true">
			 	 <view class="text">选择</view>
				 <view class="content">
					 <view class="">选择规格:</view>
					 <view class="sp">
					 	<view :class="{'on':item==goodsInfo.spec}" v-for="(item,index) in goodsData.spec" :key="index">
					 		{{item}}
					 	</view>
					 </view>
				 </view>
				 <!-- 箭头 右边-->
				 <view class="arrow">
				 	<view class="icon iconfont">&#xe6a3;</view>
				 </view>
			 </view>
		</view>
		
		<!-- 模态框 -->
		<popupSpec @add="add" @sub="sub" @hideSpec="hideSpec" @setSelectSpec="setSelectSpec" :goodsData="goodsData" :goodsInfo="goodsInfo" :spaceInfo="spaceInfo"></popupSpec>
		
		<!-- 评论 -->
		<view class="info-box comments">
			<view class="row">
				<view class="text">商品评价{{goodsData.comment.length}}</view>
				<view class="arrow" @tap="handLeRatings">
					<view class="show" >
						查看全部
						<view class="icon iconfont">&#xe6a3;</view>
					</view>
				</view>
			</view>
			<view class="comment" @tap="handLeRatings">
				<view class="user-info">
					<view class="face">
						<image :src="goodsData.comment[0].face"></image>
					</view>
					<view class="username">{{goodsData.comment[0].username}}</view>
				</view>
				<view class="content">
					{{goodsData.comment[0].content}}
				</view>
			</view>
		</view>
		
		<!-- 商品详情 -->
	    <view class="description">
	    	<view class="title">———— 商品详情 ————</view>
			<view class="content">
				<rich-text :nodes="goodsData.descriptionStr"></rich-text>
			</view>
	    </view>
		
		<!-- 底部菜单 -->
		<view class="footer">
			<view class="icons">
				<view class="box">
					<view class="icon iconfont">&#xe7e0;</view>
					<view class="text">分享</view>
				</view>
				<view class="box" @tap="keep">
					<view class="icon iconfont" v-if="isKeep">&#xe64b;</view>
					<view class="icon iconfont" v-else="isKeep">&#xe64c;</view>
					<view class="text" >{{isKeep?'已':''}}收藏</view>
				</view>
			</view>
			<view class="btn">
				<view class="joinCart" @tap="joinCart">加入购物车</view>
				<view class="buy" @tap="handleBuy">立即购买</view>
			</view>
		</view>
	</view>
</template>

<script>
	import goodsHeader from './goodsHeader.vue'
	import interfaces from '../../utils/interfaces.js'
	import popupSpec from '../../component/popupSpec.vue'
	export default{
	components:{
		goodsHeader,
		popupSpec
	},
		data(){
			return{
				  isKeep:false ,//收藏
				// 请求获取的数据
				goodsData:{
					comment:[{content:"",face:"",username:""}],
					swiperList:[],
					spec:[],
					descriptionStr:""
				},
				// 点击跳转过来的数据
				goodsInfo:{
					name:"",
					price:"",
					number:1,
					spec:""
				},
				// 轮播图导航标识下标
				currentSwiper:0,
				spaceInfo:{showSpace:false}
			}
		},
		onLoad(option) {
				console.log(option)
			this.goodsInfo=JSON.parse(option.goodsList)
		   this.initData();
		   
		},
		methods:{
			hideSpec(){
				 this.spaceInfo.showSpace=false	
			},
			setSelectSpec(item){
				// 选择规格
				this.goodsInfo.spec=item;
			},
			
			initData(){
				this.request({
					url:interfaces.getGoods,
					success:((res)=>{
					 this.goodsData=res.data
					})
				})
			},
			// 获取指示器的下标位置
			swiperChange(event){
				console.log(event)
				// 当前data中的数据等于指示器的下标位置
				this.currentSwiper=event.detail.current
			},
			handLeRatings(){
				// 本地存储
				uni.setStorageSync('comments',this.goodsData.comment);
				// 页面跳转
				uni.navigateTo({
					url:"./ratings"
				})
			},
			
			// 收藏按钮的事件
			keep(){
				this.isKeep=!this.isKeep;
				if(this.isKeep){
					uni.showToast({
					icon:"success",
					title:"已收藏"
					})
				}
			},
			joinCart(){
				// 存储到本地存储里
				uni.getStorage({
					key:"goodsList",
					success:(res=>{
						// 拿数据
						let goodsList=res.data;
						
						// 查找商品是否存在
						let isExist=false;
						// 循环 当前缓存中的每一个数据
						goodsList.forEach(goods=>{
							//如果 当前循环的id等于 当前点击添加的的id 并且型号等于 型号
							if(goods.goods_id==this.goodsInfo.goods_id && goods.spec==this.goodsInfo.spec){
								
								//如果存在 修改商品数量
								goods.number+=this.goodsInfo.number;
								isExist=true;
							}  
						})
						if(!isExist){
							goodsList.push(this.goodsInfo);
							// 更新本地存储
							this.setGoodsList(goodsList);
						}
					}),
					fail:(err=>{ //没有得到数据,那么就存
					let goodsList=[];
					 goodsList.push(this.goodsInfo);
					 
					 // 往本地存储中存储数据
						this.setGoodsList(goodsList);
					})
				})
			},
			setGoodsList(goodsList){
				// 存储到本地存储中
				uni.setStorage({
					key:"goodsList",
					data:goodsList,
					success:function(){
						uni.showToast({
							icon:"success",
							title:"添加购物车成功"
						})
					}
				})
			},
			
		handleBuy(){
			let tempList=[];
			tempList.push(this.goodsInfo);
			uni.setStorage({
				key:"confirmList",
				data:tempList,
				success:()=>{
					uni.navigateTo({
						url:"../order/confirm"
					})
				}
			})
		},
		
		add(){
			this.goodsInfo.number++;
		},
		sub(){
			if(this.goodsInfo.number<=1){
				return ;
			}
			this.goodsInfo.number--;
		}
	},
		
	}
</script>
<style lang="scss">
	/*  修改状态栏样式 */
	.status {
		/* #ifdef APP-PLUS */
		opacity: 0;
		/* #endif */
	}

	.swiper-box {
		position: relative;
		width: 100%;
		height: 100vw;

		swiper {
			width: 100%;
			height: 100vw;

			swiper-item {
				image {
					width: 100%;
					height: 100vw;
				}
			}
		}

		.indicator {
			display: flex;
			justify-content: center;
			align-items: center;
			padding: 0 25upx;
			height: 40upx;
			border-radius: 40upx;
			font-size: 22upx;
			position: absolute;
			bottom: 20upx;
			right: 20upx;
			color: #fff;
			background-color: rgba(0, 0, 0, 0.2);
		}
	}

	.info-box {
		width: 92%;
		padding: 20upx 4%;
		background-color: #fff;
		margin-bottom: 20upx;
	}

	.goods-info {
		.price {
			font-size: 46upx;
			font-weight: 600;
			color: #f47925;
		}

		.title {
			font-size: 30upx;
		}
	}

	.row {
		width: 100%;
		display: flex;
		align-items: center;
		margin: 0 0 30upx 0;

		.text {
			font-size: 28upx;
			color: #a2a2a2;
			margin-right: 20upx;
		}

		.content {
			font-size: 28upx;
			display: flex;
			flex-wrap: wrap;

			.serviceitem {
				margin-right: 10upx;
			}

			.sp {
				width: 100%;
				display: flex;

				view {
					background-color: #f6f6f6;
					padding: 10upx;
					color: #999;
					margin-right: 10upx;
					font-size: 20upx;
					border-radius: 5upx;

					&.on {
						border: solid 1upx #f47952;
						padding: 4upx 8upx;
					}
				}
			}
		}

		.arrow {
			position: absolute;
			right: 4%;

			.icon {
				color: #ccc;
			}
		}
	}

	.comments {
		.row {
			width: 100%;
			height: 40upx;
			display: flex;
			align-items: center;
			margin: 0 0 30upx 0;

			.text {
				font-size: 30upx;
			}

			.arrow {
				font-size: 28upx;
				position: absolute;
				right: 4%;
				color: #17e6a1;

				.show {
					display: flex;
					align-items: center;

					.icon {
						color: #17e6a1;
					}
				}
			}
		}

		.comment {
			width: 100%;

			.user-info {
				width: 100%;
				height: 40upx;
				display: flex;
				align-items: center;

				.face {
					width: 40upx;
					height: 40upx;
					margin-right: 8upx;

					image {
						width: 40upx;
						height: 40upx;
						border-radius: 100%;
					}
				}

				.username {
					font-size: 24upx;
					color: #999;
				}
			}

			.content {
				margin-top: 10upx;
				font-size: 26upx;
			}
		}
	}

	.description {
		.title {
			width: 100%;
			height: 80upx;
			display: flex;
			justify-content: center;
			align-items: center;
			font-size: 26upx;
			color: #999;
		}
	}

	.footer {
		position: fixed;
		bottom: 0upx;
		width: 92%;
		padding: 0 4%;
		height: 99upx;
		border-top: solid 1upx #eee;
		background-color: #fff;
		z-index: 2;
		display: flex;
		justify-content: space-between;
		align-items: center;

		.icons {
			display: flex;
			height: 80upx;
			margin-left: -4%;

			.box {
				width: 80upx;
				height: 80upx;
				display: flex;
				justify-content: center;
				flex-wrap: wrap;

				.icon {
					font-size: 40upx;
					color: #828282;
				}

				.text {
					display: flex;
					justify-content: center;
					width: 100%;
					font-size: 22upx;
					color: #666;
				}
			}
		}

		.btn {
			height: 80upx;
			border-radius: 40upx;
			overflow: hidden;
			display: flex;
			margin-right: -2%;

			.joinCart,
			.buy {
				height: 80upx;
				padding: 0 40upx;
				color: #fff;
				display: flex;
				align-items: center;
				font-size: 28upx;
			}

			.joinCart {
				background-color: #f0b46c;
			}

			.buy {
				background-color: #f06c7a;
			}
		}
	}
</style>
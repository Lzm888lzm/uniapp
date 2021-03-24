<template>
	<view>
		<!-- 购物列表 -->
		<view class="goods-list">
			<view class="empty" v-if="goodsList.length==0">购物车空空如也~</view>
			<view class="row" v-for="(item,index) in goodsList" :key="index">
				<!-- 删除按钮 -->
				<view class="menu" @tap="handleSingleDelete(item)">
					<view class="icon iconfont">&#xe6a6;</view>
				</view>
				<!-- 商品 -->
				<view class="production" 
				@touchstart="hendleTouchStart(index,$event)" 
				@touchmove="hendleTouchMove(index,$event)"
				@touchend="hendleTouchEnd(index,$event)"
				:class="[theIndex==index?'open':oldIndex==index?'close':'']">
					<!-- checkbox 单选按钮-->
					<view class="container" @tap="handleCheckbox(item)">
						<view class="checkbox">
							<view :class="{'on':item.selected}"></view>
						</view>
					</view>
					
					<!-- 商品详情 -->
					<view class="goods-info" @tap="handleGoodsInfo(item)">
						<view class="img">
							<image :src="item.img"></image>
						</view>
						<view class="info">
							<view class="title">{{item.name}}</view>
							<view class="spec">{{item.spec}}</view>
							<view class="price-number">
								<view class="price">¥{{item.price}}</view>
								<counter :goodsInfo="item" @change="sum" @add="add(item)" @sub="sub(item)"></counter>
							</view>
						</view>
						
					</view>
				</view>
			</view>
		</view>
		
		<!-- 底部菜单 -->
		<view class="footer" :style="{bottom:footerbottom}">
			<!-- checkbox 全选按钮-->
			<view class="container" @tap="handleSelectedAll">
				<view class="checkbox">
					<view :class="{'on':isAllSelected}"></view>
				</view>
				<view class="text">全选</view>
			 </view>
				<view class="delBtn" @tap="handleMudelete" v-if="selectedList.length>0">删除</view>
				<view class="settlement">
					<view class="sum">合计:<view class="money">¥{{sumprice}}</view></view>
					<view class="btn" @tap="handleConfirm">结算{{selectedList.length}}</view>
				</view>
				
			</view>
	</view>
</template>

<script>
	import counter from '../../../component/counter.vue'
	export default{
		components:{
			counter
		},
		data(){
			return{
				footerbottom: 0,
				goodsList:[],
				theIndex:null, //控制滑动效果当前滑动下标
				oldIndex:null,  //上一个左滑下标
				selectedList:[],
				isAllSelected:false,
				sumprice:'0.00'
			}
		},
		methods:{
			add(item){
				item.number++;
				this.sum;
			},
			sub(item){
				if(item.number<=1){
					return;
				}
				item.number--;
				this.sum();
			},
			handleCheckbox(item){ //单选按钮
				//取反商品添加的属性
				item.selected=!item.selected;
				
				// 条件：数组是否包含该元素
				let isExist=this.selectedList.indexOf(item);
				if(isExist>-1){
					this.selectedList.splice(isExist,1)
				}else{
					// push数组里
					this.selectedList.push(item);
				}
				
			  // 全选状态
			  if(this.selectedList.length==this.goodsList.length){
				  this.isAllSelected=true
			  }else{
				  this.isAllSelected=false
			  } 
			  
			  // 调用 选中的价格
			  this.sum();
			},
			// 全选按钮判断
			 handleSelectedAll(){ //全选按钮
			  // 当前事件不等于 当前事件  取反
				 this.isAllSelected=!this.isAllSelected;
	
				 // 数据处理
				 let arr=[]; //空数组
				 // 遍历当前goodsList里面的数据 
				 this.goodsList.forEach((item,i)=>{
					// 数据里的selected =当前事件
					 item.selected=this.isAllSelected;
					 // 把遍历个其余数据 放到arr空数组中
					 arr.push(item);
				 })
				 // 当前数据 等于单前isAllSelected 如果是 否则空
			  this.selectedList=this.isAllSelected ? arr :[]; 
                  // 调用 选中的价格
                  this.sum();
			 },
			 handleSingleDelete(item){  //滑块删除事件
				  // 更新storage 
				  uni.getStorage({
				  	key:"goodsList",
					success:(res=>{
						res.data.splice(res.data.indexOf(item),1);
					uni.setStorageSync("goodsList",res.data)
					})
				  })
				  
				  //更新数组
				  this.goodsList.splice(this.goodsList.indexOf(item),1);
				  this.selectedList.splice(this.selectedList.indexOf(item),1);
				  
				  // 更新 滑块
				  this.oldIndex=null;
				  this.theIndex=null;
				  
				  // 调佣总价
				  this.sum();
			  },
			 
			 handleMudelete(){ //全选删除事件
		      	 // 循环删除所有选中的商品
				 while(this.selectedList.length>0){
					 this.handleSingleDelete(this.selectedList[0]);
				 }
				 
				 // 初始化数据
				 this.selectedList=[];
				 this.isAllSelected=false;
				 this.sum();
			 },
			
			  handleConfirm(){  //结算
				  if(this.selectedList.length<1){
					  uni.showToast({
					  	title:"请选择结算的商品",
						icon:"none"
					  })
					  return ;
				  }
				  // 本地存储 
				  uni.setStorage({
				  	key:"confirmList",
					data:this.selectedList,
					success:()=>{
						uni.navigateTo({
							url:"../../order/confirm"
						})
					}
				  })
			  },
			 
			sum(){// 总价合计
			  this.sumprice=0;  //当前价格  
			  // 遍历当前数组里的数据  
			  this.goodsList.forEach((item,i)=>{
				  // 如果当前数组的数据selected   正确
				  if(item.selected){
					 // 当前数组的价格 等于当前数组价格 加数组数据里的 数量* 价格
					  this.sumprice=this.sumprice + (item.number * item.price) 
				  }
			  })
			    // 当前价格 保留两位小数
			  this.sumprice=this.sumprice.toFixed(2)
			},
			
			// 点击跳转
			handleGoodsInfo(item){
				uni.navigateTo({
					url:"../../goods/goods?goodsList="+JSON.stringify(item)
				})
			},
			
			// 滑块事件
			hendleTouchStart(index,$event){
				console.log(event)
				// 多点触控 不触发
				if(event.touches.length>1){
					return;
				}
				// 初始化坐标
				this.initXY=[event.touches[0].pageX,event.touches[0].pageY];
			},
			
			hendleTouchMove(index,$event){	
				if(event.touches.length>1){
					return;
				}
				this.oldIndex=this.theIndex;
				this.theIndex=null
				// 移动位置
				let moveX=event.touches[0].pageX-this.initXY[0];
				let moveY=event.touches[0].pageY-this.initXY[1];
				
				// 滑动位置小 不触发
				  if(Math.abs(moveX>5)){
					  return;
				  }
				  //竖向滑动  不触发
				  if(Math.abs(moveY)>Math.abs(moveX)){
					  return;
				  }
				 // 左滑
				 if(moveX<0){
					 this.theIndex=index
				 }
			},
			hendleTouchEnd(index,$event){
			}
		  },
		  
		// 获取缓存中的数据
		onShow() {
			uni.getStorage({
				key:"goodsList",
                success:((res)=>{
					console.log(res.data)
					//为商品添加新属性 将所有商品选中状态都设置为false 
					for(let i=0; i<res.data.length; i++){
						res.data[i].selected=false;
					}
					this.goodsList=res.data;
					
					//属性数据的初始化
					this.selectedList=[];
					this.isAllSelected=false;
					this.sumprice='0.00'
				})
			})
		},
		
		// 底部菜单获取 
		onLoad() {
			// 兼容h5下的底部菜单
			// #ifdef H5
			this.footerbottom = document.getElementsByTagName("uni-tabbar")[0].offsetHeight + "px";
			// #endif
		}
	}
</script>

<style lang="scss">
	.container {
		display: flex;
		align-items: center;

		.checkbox {
			width: 35upx;
			height: 35upx;
			border-radius: 100%;
			border: solid 2upx #f06c7a;
			display: flex;
			justify-content: center;
			align-items: center;

			.on {
				width: 25upx;
				height: 25upx;
				border-radius: 100%;
				background-color: #f06c7a;
			}
		}

		.text {
			font-size: 28upx;
			margin-left: 10upx;
		}
	}


	.goods-list {
		width: 100%;
		padding: 20upx 0 120upx 0;

		.empty {
			width: 100%;
			height: 60upx;
			display: flex;
			justify-content: center;
			align-items: center;
			font-size: 32upx;
			color: #a7a7a7;
		}

		.row {
			width: calc(92%);
			height: calc(22vw + 40upx);
			margin: 20upx auto;

			border-radius: 15upx;
			box-shadow: 0upx 5upx 20upx rgba(0, 0, 0, 0.1);
			display: flex;
			align-items: center;
			position: relative;
			overflow: hidden;
			z-index: 4;
			border: 0;

			.menu {
				.icon {
					color: #fff;
					font-size: 60upx;
				}

				position: absolute;
				width: 30%;
				height: 97%;
				left: 480upx;
				display: flex;
				justify-content: center;
				align-items: center;
				background-color: red;
				color: #fff;
				z-index: 2;
			}

			.production {
				@keyframes showMenu {
					0% {
						transform: translateX(0);
					}

					100% {
						transform: translateX(-30%);
					}
				}

				@keyframes closeMenu {
					0% {
						transform: translateX(-30%);
					}

					100% {
						transform: translateX(0);
					}
				}

				&.open {
					animation: showMenu 0.25s linear both;
				}

				&.close {
					animation: closeMenu 0.15s linear both;
				}

				background-color: #fff;

				.container {
					padding-left: 20upx;
					flex-shrink: 0;
					height: 22vw;
					margin-right: 20upx;
				}

				position: absolute;
				width: 100%;
				padding: 0 0;
				height: 100%;
				z-index: 3;
				display: flex;
				align-items: center;

				.goods-info {
					width: 100%;
					display: flex;
					padding-right: 20upx;

					.img {
						width: 22vw;
						height: 22vw;
						border-radius: 10upx;
						overflow: hidden;
						flex-shrink: 0;
						margin-right: 10upx;

						image {
							width: 22vw;
							height: 22vw;
						}
					}

					.info {
						width: 100%;
						height: 22vw;
						overflow: hidden;
						display: flex;
						flex-wrap: wrap;
						position: relative;

						.title {
							width: 100%;
							font-size: 28upx;
							display: -webkit-box;
							-webkit-box-orient: vertical;
							-webkit-line-clamp: 2;
							// text-align: justify;
							overflow: hidden;
						}

						.spec {
							font-size: 20upx;
							background-color: #f3f3f3;
							color: #a7a7a7;
							height: 30upx;
							display: flex;
							align-items: center;
							padding: 0 10upx;
							border-radius: 15upx;
							margin-bottom: 20vw;
						}

						.price-number {
							position: absolute;
							width: 100%;
							bottom: 0upx;
							display: flex;
							justify-content: space-between;
							align-items: flex-end;
							font-size: 28upx;
							height: 60upx;

							.price {}

						}
					}
				}
			}
		}
	}

	.footer {
		width: 92%;
		padding: 0 4%;
		background-color: #fbfbfb;
		height: 100upx;
		display: flex;
		justify-content: space-between;
		align-items: center;
		font-size: 28upx;
		position: fixed;
		bottom: 0upx;
		z-index: 5;

		.delBtn {
			border: solid 1upx #f06c7a;
			color: #f06c7a;
			padding: 0 30upx;
			height: 50upx;
			border-radius: 30upx;
			display: flex;
			justify-content: center;
			align-items: center;
		}

		.settlement {
			width: 60%;
			display: flex;
			justify-content: flex-end;
			align-items: center;

			.sum {
				width: 50%;
				font-size: 28upx;
				margin-right: 10upx;
				display: flex;
				justify-content: flex-end;

				.money {
					font-weight: 600;
				}
			}

			.btn {
				padding: 0 30upx;
				height: 50upx;
				background-color: #f06c7a;
				color: #fff;
				display: flex;
				justify-content: center;
				align-items: center;

				border-radius: 30upx;
			}
		}
	}
</style>
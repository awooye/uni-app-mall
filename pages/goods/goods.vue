<template>
	<view>
		<!-- 状态栏 -->
		<page-status></page-status>
		<!-- header -->
		<goodsHeader></goodsHeader>

		<!-- 轮播图 -->
		<view class="swiper-box">
			<swiper @change="swiperChange" circular="true" autoplay="true">
				<swiper-item v-for="swiper in goodsData.swiperList" :key="swiper.id">
					<image :src="swiper.img"></image>
				</swiper-item>
			</swiper>
			<view class="indicator">
				{{currentSwiper+1}}/{{goodsData.swiperList.length}}
			</view>
		</view>
		<!-- 标题和价格 -->
		<view class="info-box goods-info">
			<view class="price">
				￥{{goodsInfo.price}}
			</view>
			<view class="title">
				{{goodsInfo.name}}
			</view>
		</view>

		<!-- 选择规格 -->
		<view class="info-box">
			<view class="row" @tap="spaceInfo.showSpace=true">
				<view class="text">
					选择
				</view>
				<view class="content">
					<view>选择规格：</view>
					<view class="sp">
						<view :class="{'on':item==goodsInfo.spec}" v-for="(item,index) in goodsData.spec" :key="index">
							{{item}}
						</view>
					</view>
				</view>

				<!-- 箭头 -->
				<view class="arrow">
					<view class="icon iconfont">&#xe6a3;</view>
				</view>

			</view>
		</view>


		<!-- 模态框 -->
		<popupSpec @add="add" @sub="sub" @setSelectSpec="setSelectSpec" @hideSpec="hideSpec" :goodsInfo="goodsInfo" :goodsData="goodsData" :spaceInfo="spaceInfo"></popupSpec>

		<!-- 评价 -->
		<view class="info-box comments">
			<view class="row">
				<view class="text">商品评价{{goodsData.comment.length}}</view>
				<view class="arrow" @tap="handleRatings">
					<view class="show">
						查看全部
						<view class="icon iconfont">&#xe6a3;</view>
					</view>
				</view>
			</view>
			<view class="comment" @tap="handleRatings">
				<view class="user-info">
					<view class="face">
						<image :src="goodsData.comment[0].face"></image>
					</view>
					<view class="username">
						{{goodsData.comment[0].username}}
					</view>
				</view>
				<view class="content">
					{{goodsData.comment[0].content}}
				</view>
			</view>
		</view>

		<!-- 商品详情 -->
		<view class="description">
			<view class="title">
				———— 商品详情 ————
			</view>
			<view class="content">
				<rich-text :nodes="goodsData.descriptionStr"></rich-text>
			</view>
		</view>
		<!-- 底部菜单 -->
		<view class="footer">
			<view class="icons">
				<view class="box">
					<view class="icon iconfont">
						&#xe7e0;
					</view>
					<view class="text">
						分享
					</view>
				</view>
				<view class="box" @tap="keep">
					<view class="icon iconfont" v-if="isKeep">&#xe64b;</view>
					<view class="icon iconfont" v-else>&#xe64c;</view>
					<view class="text">
						{{isKeep?'已':''}}收藏
					</view>
				</view>
			</view>
			<view class="btn">
				<view class="joinCart" @tap="joinCart">
					加入购物车
				</view>
				<view class="buy" @tap="handleBuy">
					立即购买
				</view>
			</view>
		</view>

	</view>
</template>

<script>
	import goodsHeader from './goodsHeader.vue';
	import interfaces from '../../utils/interfaces.js';
	import popupSpec from '../../components/popupSpec.vue';
	export default {
		data() {
			return {
				isKeep: false, // 默认不收藏
				goodsData: {
					swiperList: [],
					spec: [],
					comment: [{
						face: "",
						content: "",
						username: ""
					}],
					descriptionStr: "",
				},
				goodsInfo: {
					nama: "",
					price: "",
					number: 1,
					spec: ""
				},
				currentSwiper: 0, //轮播图下标
				spaceInfo: {
					showSpace: false
				}
			}
		},
		components: {
			goodsHeader,
			popupSpec
		},
		onLoad(option) {
			// 参数赋值
			this.goodsInfo = JSON.parse(option.goodsInfo)
			//初始化数据
			this.goodsInfo.number = 1;
			this.goodsInfo.spec = 'XS'

			this.initData()
		},
		methods: {
			setSelectSpec(item){
				// console.log(item)
				// 选择规格
				this.goodsInfo.spec=item;
			},
			hideSpec(){
				this.spaceInfo.showSpace=false;
			},
			add(){
				this.goodsInfo.number++;
			},
			sub(){
				if(this.goodsInfo.number<=1){
					return;
				}
				this.goodsInfo.number--;
			},
			initData() {
				this.request({
					url: interfaces.getGoods,
					success: ((res) => {
						this.goodsData = res.data;
						// console.log(res.data);
					})
				})
			},
			swiperChange(event) {
				// console.log(event.detail.current);
				this.currentSwiper = event.detail.current;
			},
			handleRatings() {
				// console.log(this.goodsData.comment[0].face)
				// 本地存储
				uni.setStorageSync('comments', this.goodsData.comment)
				console.log(this.goodsData.comment)
				// 页面跳转
				uni.navigateTo({
					url: './ratings'
				})
			},
			keep() {
				this.isKeep = !this.isKeep;
				if (this.isKeep) {
					uni.showToast({
						icon: "success",
						title: "已收藏"
					})
				}
			},
			joinCart() {
				console.log(this.goodsInfo);

				// 存储到本地存储


				// 1.先去本地存储中取
				uni.getStorage({
					key: 'goodsList',
					success: (res => {
						let goodsList = res.data;
						let isExist=false;
						goodsList.forEach(goods=>{
							if(goods.goods_id==this.goodsInfo.goods_id){
								goods.number+=this.goodsInfo.number;
								isExist=true;
							}
						})
						if(!isExist)
						goodsList.push(this.goodsInfo)
						
						this.setGoodsList(goodsList)

						//拿数据
						console.log("加入成功");

						// this.setGoodsList(goodsList)
					}),
					fail: (err => { //拿不到数据再存数据
						console.log("加入失败")
						let goodsList = [];
						goodsList.push(this.goodsInfo);

						// 往本地存储中存储数据
						this.setGoodsList(goodsList);
					})
				})
			},
			setGoodsList(goodsList) {
				console.log("存储到本地存储")
				//存储到本地存储
				uni.setStorage({ 
					key: 'goodsList',
					data: goodsList,
					success: function () {
						uni.showToast({
							icon:'success',
							title:'添加购物车成功'
						})
					}
				});
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
			}
		}
	}
</script>

<style lang="scss">
	.status {
		// 修改状态栏样式
		/*  #ifdef  APP-PLUS  */
		opacity: 0;
		/*  #endif  */
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

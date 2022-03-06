<template>
	<view class="content">
		<scroll-view scroll-y class="left-aside">
			<view v-for="item in flist" :key="item.id" class="f-item b-b" :class="{active: item.id === currentId}" @click="tabtap(item)">
				{{item.name}}
			</view>
		</scroll-view>
		<scroll-view scroll-with-animation scroll-y class="right-aside" @scroll="asideScroll" :scroll-top="tabScrollTop">
			<template v-if="goods_list.length > 0">
				<view v-for="item in goods_list" :key="item.id" class="s-list" :id="'main-'+item.id">
					<view class="t-list">
						<view  class="t-item text-cut" >
							<view class="">
								<image :src="item.resources.img  | smallImage(80)" lazy-load></image>
							</view>
							<view class="text-cut text-center">
								<text >{{item.name}}</text>
								
							</view>
							<view  class="all_qian">
								<text class="qian">￥{{item.order_price}}</text>
								<view class="jia" @click="toggleSpec(false,item)" :key="item.id" ><text class="cuIcon-add text-red" ></text></view>
								
							</view>
							
							
						</view>
					</view>
				</view>
			</template>
		</scroll-view>
		
		<view class="popup spec" :class="specClass" @touchmove.stop.prevent="stopPrevent" @click="toggleSpec">
			<view class="mask"></view>
			<view class="layer attr-content" @click.stop="stopPrevent">
				<sku ref="my_sku" :getList="getList" :buy="buy" @toggleSpec="toggleSpec" :update="update" :loaded="loaded"  @purchasePattern="purchasePattern"></sku>
			</view>
		</view>
	</view>
</template>

<script>
	import Good from '../../api/good'
	import sku from '@/components/sku';
	import uParse from '@/components/gaoyia-parse/parse.vue'
	import share from '@/components/share';
	import { param2Data } from '@/components/sku/sku2param';
	import Browse from '../../api/browse';
	import Collect from '../../api/collect';
	import {
			mapState,
			mapMutations
		} from 'vuex';
	export default {
		components: {
			share,
			sku,
			uParse
		},
		data() {
			return {
				sizeCalcState: false,
				tabScrollTop: 0,
				specificationDefaultDisplay:'',
				currentId: 49,	//一级类目默认选中的ID
				flist: [],
				slist: [],
				specClass: 'none',
				tlist: [],
				getList:{
					is_delete:0,
					is_show:1
				},
				goods_list:[],
				tap: 0,
				buy: false,
				update: false,
				loaded: false,
				resources_many:[]
			}
		},
		async onLoad(options) {
			this.loadData();
		},
		// onLoad(){
		// 	this.loadData();
		// },
		methods: {
			async loadData(){
				const that = this
				// 分类
				await Good.goodCategory({},function(res){
					res.forEach(item=>{
						if(!item.pid){
							that.flist.push(item);  //pid为父级id, 没有pid或者pid=0是一级分类
						}
					})
					if(that.flist[0]){
						that.loadGoods(that.flist[0].id);
					}
				})
			},
			async loadGoodsData(id) {
				// 商品详情
				const that = this;
				await Good.detail(id, {}, function(res) {
					if (res.resources_many.length > 0) {
						res.resources_many.forEach((item,index)=>{
							if(item.depict.indexOf('_video') !== -1){
								item.type = 'video'
								that.resources_many.unshift(item)
							} else if(item.depict.indexOf('_poster') !== -1){
								that.poster = item.img
							} else {
								item.type = 'img'
								that.resources_many.push(item)
							}
						})
					}
					console.log(res)
					that.getList = res
					if(that.getList.good_sku.length<=0){
						that.inventoryFlag = false
					}
					if (that.hasLogin){
						that.browse()
					}
				})
				if (that.hasLogin){
					await Collect.detail(id, function(res) {
						if(res === 1){
							that.favorite = true
						} else {
							that.favorite = false
						}
					})
				}
			},
			toggleSpec(state,item) {
				if (!this.hasLogin && state === true){
					this.$api.msg('请先登录')
					return false
				}
				if (typeof state === 'boolean'){
					this.buy=state
				}
				if (this.specClass === 'show') {
					this.specClass = 'hide';
					setTimeout(() => {
						this.specClass = 'none';
					}, 250);
				} else if (this.specClass === 'none') {
					this.specClass = 'show';
				}
				if(item){
					this.getList = item
					//this.loadGoodsData(item.id);					
				}
			},
			async loadGoods(id){
				const that = this
				// 分类
				await Good.getList({'first_category_id':id},function(res){
					console.log(res);
					that.goods_list = res.data;					
				})
			},
			//一级分类点击
			tabtap(item){
				this.tap = 1
				this.currentId = item.id;
				this.loadGoods(item.id);
			},
			asideScroll(){
				
			},
			stopPrevent() {},
			purchasePattern(data) {
				this.specificationDefaultDisplay = data;
			},

			
		}
	}
</script>

<style lang='scss'>
	page {
		background: $page-color-base;
		padding-bottom: 160upx;
	}
	.popup {
		position: fixed;
		left: 0;
		top: 0;
		right: 0;
		bottom: 0;
		z-index: 99;
	
		&.show {
			display: block;
			.mask {
				animation: showPopup 0.2s linear both;
			}
			.layer {
				animation: showLayer 0.2s linear both;
			}
		}
		&.hide {
			.mask {
				animation: hidePopup 0.2s linear both;
			}
			.layer {
				animation: hideLayer 0.2s linear both;
			}
		}
		&.none {
			display: none;
		}
		.mask {
			position: fixed;
			top: 0;
			width: 100%;
			height: 100%;
			z-index: 1;
			background-color: rgba(0, 0, 0, 0.4);
		}
		.layer {
			position: fixed;
			z-index: 99;
			bottom: 0;
			width: 100%;
			min-height: 40vh;
			border-radius: 10upx 10upx 0 0;
			background-color: #fff;
			.btn {
				height: 66upx;
				line-height: 66upx;
				border-radius: 100upx;
				background: $uni-color-primary;
				font-size: $font-base + 2upx;
				color: #fff;
				margin: 30upx auto 20upx;
			}
		}
		@keyframes showPopup {
			0% {
				opacity: 0;
			}
			100% {
				opacity: 1;
			}
		}
		@keyframes hidePopup {
			0% {
				opacity: 1;
			}
			100% {
				opacity: 0;
			}
		}
		@keyframes showLayer {
			0% {
				transform: translateY(120%);
			}
			100% {
				transform: translateY(0%);
			}
		}
		@keyframes hideLayer {
			0% {
				transform: translateY(0);
			}
			100% {
				transform: translateY(120%);
			}
		}
	}
	/* 规格选择弹窗 */
	.attr-content {
		padding: 10upx 30upx;
		.a-t {
			display: flex;
			image {
				width: 170upx;
				height: 170upx;
				flex-shrink: 0;
				margin-top: -40upx;
				border-radius: 8upx;
			}
			.right {
				display: flex;
				flex-direction: column;
				padding-left: 24upx;
				font-size: $font-sm + 2upx;
				color: $font-color-base;
				line-height: 42upx;
				.price {
					font-size: $font-lg;
					color: $uni-color-primary;
					margin-bottom: 10upx;
				}
				.selected-text {
					margin-right: 10upx;
				}
			}
		}
		.specification {
			max-height: 700upx;
			overflow-y: auto;
		}
		.attr-list {
			position: relative;
			display: flex;
			flex-direction: column;
			font-size: $font-base + 2upx;
			color: $font-color-base;
			padding-top: 30upx;
			padding-left: 10upx;
		}
		.item-left {
			display: flex;
			width: 120upx;
			margin-bottom: 60upx;
		}
		.item-right .step {
			right: 0upx;
			left: auto;
			bottom: 50upx;
		}
		.item-list {
			padding: 20upx 0 0;
			display: flex;
			flex-wrap: wrap;
			text {
				display: flex;
				align-items: center;
				justify-content: center;
				background: #eee;
				margin-right: 20upx;
				margin-bottom: 20upx;
				border-radius: 100upx;
				min-width: 60upx;
				height: 60upx;
				padding: 0 20upx;
				font-size: $font-base;
				color: $font-color-dark;
			}
			.selected {
				background: #fbebee;
				color: $uni-color-primary;
			}
			.disabled {
				color: $font-color-disabled;
			}
		}
	}
	.content {
		height: 100%;
		background-color: #f8f8f8;
	}

	.content {
		display: flex;
	}
	.left-aside {
		flex-shrink: 0;
		width: 200upx;
		height: 100%;
		background-color: #fff;
	}
	.f-item {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 100%;
		height: 100upx;
		font-size: 28upx;
		color: $font-color-base;
		position: relative;
		&.active{
			color: $base-color;
			background: #f8f8f8;
			&:before{
				content: '';
				position: absolute;
				left: 0;
				top: 50%;
				transform: translateY(-50%);
				height: 36upx;
				width: 8upx;
				background-color: $base-color;
				border-radius: 0 4px 4px 0;
				opacity: .8;
			}
		}
	}

	.right-aside{
		flex: 1;
		overflow: hidden;
		padding-left: 20upx;
	}
	.s-item{
		display: flex;
		align-items: center;
		height: 70upx;
		padding-top: 8upx;
		font-size: 28upx;
		color: $font-color-dark;
	}
	.t-list{
		display: flex;
		flex-wrap: wrap;
		width: 100%;
		background: #f8f8f8;
		padding-top: 12upx;
		&:after{
			content: '';
			flex: 99;
			height: 0;
		}
	}
	.t-item{
		flex-shrink: 0;
		justify-content: center;
		align-items: center;
		flex-direction: column;
		width: 98%;
		font-size: 26upx;
		color: #666;
		padding-bottom: 20upx;
		background: #fff;
		margin: 19rpx 12rpx 19rpx 0rpx;
		border-radius: 19rpx;
		image{
			width: 140upx;
			height: 140upx;
			float: left;
			margin-top: 33rpx;
		}
		.text-cut{
			float: left;
			width: 360rpx;
			overflow: hidden;
			text-overflow: ellipsis;
			display: -webkit-box;
			-webkit-line-clamp: 2;
			-webkit-box-orient: vertical;
			display: -moz-box;
			-moz-line-clamp: 2;
			-moz-box-orient: vertical;
			word-wrap: break-word;
			word-break: break-all;
			white-space: normal; 
			margin-top: 33rpx;
		}
		.all_qian{
			width: 65%;
			margin-top: 39rpx;
			float: left;
			.qian{	
				float: left;
				font-size: 36rpx;
				color: red;
			}
			.jia{
				float: right;
				.text-red{
					color: red;
					font-size: 44rpx;
				}
				margin-left: 15rpx;
				
			}
		}
		
	}
</style>

<template>
	<view>
		<custom-navbar title="报价" />

		<back-container>
			<template #headerSlot>
				<offer-head :title="$tool.orderType(order.orderType)" :text="`订单编号：${order.id}`" />
			</template>

			<view class="offer">
				<quoted-iten :order="order" :isPlaceOrder="isPlaceOrder" />
			</view>
		</back-container>

		<view class="offer-9" v-if="isPlaceOrder">
			<view class="offer-91">
				<view class="offer-91-text">承接方</view>
				<view class="offer-91-icon" :class="{
					'offer-91-icon-yellow': order.receiverType == 1
				}" @click="toTeamDetail">{{ order.receiverType == 1 ? '个人' : '查看团队信息' }}</view>
			</view>
			<view class="offer-92">
				<view class="offer-92-item" v-if="order.receiverType == 1" @click="toPersonDetail(order.receiverId)">
					<image v-if="order.receiverHeadImgUrl" class="offer-92-img" :src="order.receiverHeadImgUrl" />
					<image v-else class="offer-92-img"
						src='https://ttd-public.obs.cn-east-3.myhuaweicloud.com/app-img/mine/MDicon.png' />
					<view class="offer-92-name">{{ order.receiverUserName }}</view>
				</view>
				<view class="offer-92-item" v-else v-for="(i, index) in memberList" :key="i.userId"
					@click="toPersonDetail(i.userId)">
					<image v-if="i.headImgUrl" class="offer-92-img" :src="i.headImgUrl" />
					<image v-else class="offer-92-img"
						src='https://ttd-public.obs.cn-east-3.myhuaweicloud.com/app-img/mine/MDicon.png' />
					<view class="offer-92-name">{{ i.userName }}</view>
				</view>
			</view>
		</view>

		<view class="offer-8900" v-if="!isPlaceOrder && order.receiverType == 2">
			<member-title title="参与人员" :show-right="!isPlaceOrder" right-text="选择人员" @add="toSelectPerson" />
			<team-list-item v-for="(i, index) in memberList" :key="i.userId" :member="i" :showDelete="!isPlaceOrder"
				@onDelete="removePerson(i)" @onClick="toPersonDetail(i.userId)" />
		</view>

		<view class="offer-5">
			<view class="offer-51">工作内容</view>
			<view class="offer-52">
				<offer-content-card v-for="(item, index) in showWorkList" :key="index"
					:right-type="isPlaceOrder ? 5 : item.quoteAmount ? 1 : 2" :title="getItemTitle(item)"
					:specItem="getSpecList(item)" :image="itemImage" :price="item.quoteAmount ? item.quoteAmount / 100 : null"
					:show-last-border-bottom="index < (showWorkList.length - 1)" @onClick="checkItem(item)"
					@onChange="changeQuote(item)" />
			</view>
			<view v-if="workList.length > 5" class="offer-53" @click="showWork">
				<text class="offer-531">{{ showWorkMore ? '收起' : '展开更多' }}</text>
				<uni-icons class="oc-arrow" type="arrowright" size="20" color="#BDBDBD" />
			</view>
		</view>

		<view class="offer-7">
			<add-remark @input="inputChange" :value="remark" />
		</view>

		<view style="height: 300rpx" />

		<iphonex-bottom v-if="!isPlaceOrder">
			<bottom-price-and-btn :price="getQuoteCount(3)" @onCancel="cancelQuote" @onConfirm="submitQuote">
				<view class="offer-8">
					<text class="offer-81">已报价</text>
					<corner-mark :num="getQuoteCount(1)" />
					<text class="offer-81">未报价</text>
					<corner-mark :num="getQuoteCount(2)" color="#828282" />
				</view>
			</bottom-price-and-btn>
		</iphonex-bottom>

	</view>
</template>
<script>
	import config from '../../../utils/config.js';
	import BackContainer from "../../mine/addressManage/component/backContainer";
	import OfferHead from "../component/offerHead";
	import UniIcons from "../../../uni_modules/uni-icons/components/uni-icons/uni-icons";
	import OfferContentCard from "../component/offerContentCard";
	import AddRemark from "../component/addRemark";
	import IphonexBottom from "../../mine/addressManage/component/iphonexBottom";
	import CornerMark from "../component/cornerMark";
	import MyPrice from "../component/myPrice";
	import QuotedIten from "../component/quotedIten";
	import BottomPriceAndBtn from "../component/bottomPriceAndBtn";
	import MemberTitle from "../myTeam/memberTitle";
	import TeamListItem from "../myTeam/teamListItem";

	export default {
		name: "offer",
		components: {
			TeamListItem,
			MemberTitle,
			BottomPriceAndBtn,
			QuotedIten,
			MyPrice,
			CornerMark,
			IphonexBottom,
			AddRemark,
			OfferContentCard,
			UniIcons,
			OfferHead,
			BackContainer
		},
		data() {
			return {
				id: '',
				isPlaceOrder: false,
				order: {},
				workList: [],
				itemImage: '',
				memberList: [],
				showWorkList: [],
				showWorkMore: false,
				remark: ''
			};
		},
		onLoad(option) {
			if (option.isPlaceOrder) {
				this.isPlaceOrder = option.isPlaceOrder == '1';
			}
			if (option.id) {
				this.id = option.id;
				this.queryOrderInfo();
				this.queryWorkList();
			}
			if (this.isPlaceOrder) {
				this.queryMemberList();
			}
		},
		methods: {
			queryOrderInfo() {
				this.$http.post('/b/orderreceive/query', {
						id: this.id
					}, true)
					.then(res => {
						this.order = res;
						this.remark = res.receiveRemark || '';
						this.queryItemImg();
					})
			},
			queryItemImg() {
				this.$http.post('/b/ordermaster/orderTypeImg', {})
					.then(res => {
						const item = res.filter((i) => i.type == this.order.orderType)[0];
						this.itemImage = item ? item.icon : '';
						console.log('itemImage ', this.itemImage);
					})
			},
			queryWorkList() {
				this.$http.post('/b/orderquote/receiveQuoteDetail', {
						id: this.id
					}, true)
					.then(res => {
						this.workList = res.orderItemList;
						this.showWorkList = this.workList.slice(0, 5);
					})
			},
			queryMemberList() {
				this.$http.post('/b/ordermember/queryMemberListAndApplyInfo', {
						id: this.id
					}, true)
					.then(res => {
						this.memberList = (res.applyMemberList || []).map((m) => {
							return {
								headImgUrl: m.headImgUrl,
								userName: m.name,
								phone: m.phone,
								skills: m.skills,
								userId: m.id,
							}
						});
					})
			},
			inputChange(value) {
				this.remark = value;
			},
			showWork() {
				this.showWorkMore = !this.showWorkMore;
				if (this.showWorkMore) {
					this.showWorkList = this.workList;
				} else {
					this.showWorkList = this.workList.slice(0, 5);
				}
			},
			checkItem(work) {
				const typeArray = [
					'/pages/place-order/addImplementation/addImplementation?isEdit=0',
					'/pages/place-order/addCanBeSetWork/addCanBeSetWork?isEdit=0',
					'/pages/place-order/addPersonWork/addPersonWork?isEdit=0',
					'/pages/place-order/addLeaseWork/addLeaseWork?isEdit=0',
					'/pages/place-order/addSoftwareDevelop/addSoftwareDevelop?isEdit=0',
				];
				uni.navigateTo({
					url: typeArray[this.order.orderType - 1],
					success: (res) => {
						// 通过eventChannel向被打开页面传送数据
						res.eventChannel.emit('editWork', work);
					}
				})
			},
			changeQuote(work) {
				uni.showModal({
					title: '设置报价',
					editable: true,
					placeholderText: '请输入报价',
					success: (res) => {
						if (res.confirm) {
							work.quoteAmount = Number(res.content) * 100;
							console.log('res ', res.content, work);
						}
					}
				})
			},
			getItemTitle(work) {
				if (work.itemType == 1) {
					return work.type == 1 ? '实施' : '维修';
				} else if (work.itemType == 2) {
					return '勘测';
				} else if (work.itemType == 3) {
					return work.cateName;
				} else if (work.itemType == 4) {
					return work.cateName;
				} else if (work.itemType == 5) {
					return work.cateName;
				}
				return '';
			},
			getSpecList(work) {
				if (work.itemType == 1) {
					// 实施
					return [{
							label: '技能：',
							value: work.cateName
						},
						{
							label: '品牌/型号：',
							value: `${work.brand}/${work.model ? work.model : '-'}`
						},
						{
							label: '数量：',
							value: work.number
						},
					];
				} else if (work.itemType == 2) {
					// 勘测
					return [{
						label: '面积：',
						value: work.number
					}, ];
				} else if (work.itemType == 3) {
					// 人员岗位
					return [{
						label: '数量：',
						value: work.number
					}, ];
				} else if (work.itemType == 4) {
					// 设备
					return [{
							label: '数量：',
							value: work.number
						},
						{
							label: '使用路程：',
							value: work.distance
						},
					];
				}
				return [];
			},
			getQuoteCount(type) {
				const quoteList = this.workList.filter((w) => w.quoteAmount);
				if (type == 1) {
					// 已报价
					return quoteList.length;
				} else if (type == 2) {
					// 未报价
					return this.workList.length - quoteList.length;
				} else {
					// 已报价金额
					let amount = 0;
					quoteList.forEach((q) => {
						amount = amount + Number(q.quoteAmount);
					})
					return amount / 100;
				}
			},
			toSelectPerson() {
				uni.navigateTo({
					url: `/pages/receive-order/selectPerson/selectPerson?id=${this.order.receiverId || ''}`,
					events: {
						onSelect: (members) => {
							this.memberList = members;
						}
					}
				})
			},
			removePerson(person) {
				if (this.isPlaceOrder) return;
				const index = this.memberList.findIndex((p) => p.userId == person.userId);
				this.memberList.splice(index, 1);
			},
			// 查看承接方个人详情
			toPersonDetail(userId) {
				uni.navigateTo({
					url: `/pages/receive-order/peopleDetail/peopleDetail?id=${userId}&hideInfo=${
					this.order.state == 10 || this.order.state == 20 || this.order.state == 90 ? 1 : 0}`
				})
			},
			// 查看团队详情
			toTeamDetail() {
				if (this.order.receiverType == 1) return;
				// uni.navigateTo({
				// 	url: `/pages/receive-order/myTeam/myTeam?id=${this.order.receiverId}&sensitive=1`
				// })

				uni.navigateTo({
					url: `/pages/receive-order/teamDetail/teamDetail?id=${this.order.receiverId}`,
				})
			},
			cancelQuote() {
				uni.navigateBack({});
			},
			submitQuote() {
				uni.showModal({
					title: '提示',
					content: '参与报价需要缴纳报价保证金，正常服务完成或未被选中，保证金原路退回',
					success: (res) => {
						if (res.confirm) {
							this.createQuote();
						}
					}
				})
			},
			createQuote() {
				const orderQuoteList = this.workList.map((w) => {
					return {
						price: w.quoteAmount,
						workId: w.id
					}
				}).filter(v => v.price !== null);
				const params = {
					receiveOrderId: this.id,
					orderQuoteList,
					memberIdList: this.memberList.map(p => p.userId),
					remark: this.remark,
				};
				this.$http.post('/b/orderquote/createQuote', params, true)
					.then(res => {
						// 如果支付金额是0,则不需要缴纳, 大于0,需要缴纳保证金
						if (res.payCash > 0) {
							this.payOrder(res.id);
						} else {
							uni.showToast({
								title: '报价完成',
								success: () => {
									uni.navigateBack({});
								}
							})
						}
					})
			},
			// 支付保证金
			payOrder(orderId) {
				const user = this.$store.state.user;
				const params = {
					appId: config.appId,
					openId: user.openId,
					orderId: orderId,
					wayId: 6, // 6、微信小程序支付
				}
				console.log('user ', user);
				this.$http.post('/core/pay/build4ReceiveOrder', params, true)
					.then(res => {
						// 调起微信支付
						this.wxPay(res.payResult);

						// uni.showToast({
						// 	title: '订单支付完成',
						// 	success: () => {
						// 		uni.navigateBack({});
						// 	}
						// })
					})
			},
			wxPay(res) {
				uni.requestPayment({
					provider: 'wxpay',
					timeStamp: res.timestamp,
					nonceStr: res.noncestr,
					package: res.packageName,
					signType: 'MD5',
					paySign: res.sign,
					success: (res) => {
						console.log('success:' + JSON.stringify(res));
						uni.showToast({
							title: '保证金支付完成',
							success: () => {
								uni.navigateBack({});
							}
						})
					},
					fail: function(err) {
						console.log('fail:' + JSON.stringify(err));
						uni.showToast({
							title: '保证金支付失败',
							icon: 'none'
						});
					},
				});
			}
		}
	}
</script>
<style lang="scss">
	.offer {
		padding-bottom: 34rpx;

		.offer-1 {
			width: 750rpx;
			height: 96rpx;
			background-color: white;
			padding: 24rpx 32rpx;
			box-sizing: border-box;

			.offer-11 {
				margin-left: auto;
				width: 116rpx;
				height: 48rpx;
				background: #5AC8FA;
				border-radius: 8rpx;
				font-size: 28rpx;
				font-family: PingFang SC-Regular, PingFang SC;
				font-weight: 400;
				color: #FFFFFF;
				@include flexCenter;
			}
		}

	}

	.offer-5 {
		background-color: white;
		margin: 30rpx 0 32rpx 0;

		.offer-51 {
			width: 750rpx;
			height: 88rpx;
			padding: 24rpx 32rpx;
			box-sizing: border-box;
		}

		.offer-52 {
			margin-left: 32rpx;
			border-top: 1rpx solid #EAECEF;
			border-bottom: 1rpx solid #EAECEF;
		}

		.offer-53 {
			height: 80rpx;
			padding: 0 32rpx;
			@include flexBetween;
			background: #FFFFFF;

			.offer-531 {
				font-size: 28rpx;
				font-family: PingFang SC-Regular, PingFang SC;
				font-weight: 400;
				color: #1889F9;
			}
		}
	}

	.offer-7 {
		background-color: white;
		padding-bottom: 32rpx;
	}

	.offer-8 {
		display: flex;
		align-items: center;
		height: 80rpx;
		background: #FFFBE8;

		.offer-81 {
			margin: 0 12rpx 0 32rpx;
		}
	}

	.offer-8900 {
		background-color: white;
		margin: 32rpx 0;
	}

	.offer-9 {
		background-color: white;
		margin: 32rpx 0;

		.offer-91 {
			height: 80rpx;
			padding: 0 32rpx;
			@include flexBetween;

			.offer-91-text {
				font-size: 28rpx;
				font-family: PingFang SC-Regular, PingFang SC;
				font-weight: 400;
			}

			.offer-91-icon {
				padding: 0 12rpx;
				height: 40rpx;
				background: #2C3580;
				border-radius: 8rpx;
				font-size: 24rpx;
				font-family: PingFang SC-Regular, PingFang SC;
				font-weight: 400;
				color: #FFFFFF;
				@include flexCenter;
			}

			.offer-91-icon-yellow {
				background-color: #FF9500;
			}
		}

		.offer-92 {
			padding: 32rpx;
			display: flex;
			flex-flow: wrap row;

			.offer-92-item {
				margin-right: 58rpx;
				width: 127rpx;

				&:nth-of-type(4n + 4) {
					margin: 0;
				}

				.offer-92-img {
					width: 127rpx;
					height: 127rpx;
					border-radius: 8rpx;
				}

				.offer-92-name {
					font-size: 28rpx;
					font-family: PingFang SC-Regular, PingFang SC;
					font-weight: 400;
					text-align: center;
					margin-top: 16rpx;
					color: #323232;
					line-height: 36rpx;
				}
			}
		}
	}
</style>
<style lang="scss">
	@import "../../mine/addressManage/_pageStyle.scss";

	.placeholder-class {
		font-size: 28rpx;
		font-family: PingFang SC-Regular, PingFang SC;
		font-weight: 400;
		color: #828282;
		line-height: 36rpx;
	}
</style>

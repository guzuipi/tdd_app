<template>
	<view class="my-wallet">
		<custom-navbar title="我的钱包" />
		<back-container>
			<template v-slot:headerSlot>
				<view class="wallet-top">
					<view class="wallet-rule" @click="showRule">查看提现规则</view>
					<view class="wallet-top-num">
						我的余额
						<uni-icons :type="showBalance ? 'eye-slash' : 'eye-slash-filled'" class="eye-slash-filled" size="16" color="#ffffff"
							@click="()=>showBalance = !showBalance" />
					</view>
					<view class="wallet-top-ac">
						<view class="wallet-top-number">{{ showBalance ? balance : '***'}}</view>
						<view class="wallet-top-btn" @click="withdrawCheck">提现</view>
					</view>
				</view>
			</template>

			<scroll-view class="wallet-center"
                   scroll-y
                   refresher-enabled
                   :refresher-triggered="refresherTriggered"
                   @refresherpulling="refresherpulling"
                   :style="{ height: `calc(100vh - ${(barHeight + 454)}rpx)` }">
				<view class="wallet-item">
					<view class="edit-lable">收支明细</view>
				</view>

				<view class="wallet-detail-item" v-for="(item, index) in balanceList" :key="item.id">
					<view class="wallet-detail-item-left">
						<view class="wallet-item-left1">{{ item.title }}</view>
						<view class="wallet-item-left2">
							{{ `${item.inOutType == 1 ? '+': '-'} ${item.inOutType == 1 ? (item.inMoney / 100) : (item.payMoney / 100) }`}}
							元
						</view>
					</view>
					<view class="wallet-detail-item-right">
						<view class="wallet-item-right1">{{ item.addTime }}</view>
						<view class="wallet-item-right2">交易成功</view>
					</view>
				</view>

				<list-empty v-if="!balanceList.length" />
			</scroll-view>
		</back-container>

		<!-- 提现弹窗 -->
		<model-slot v-if="visible" title="提现" ref="modelSlot" @sure="withdrawMoney" @hide="hide">
			<template #slot1>
				<view class="wallet-model">
					<view class="wallet-model1">
						提现金额：
						<text class="wallet-model1-1">{{ balance }}</text>
						<text class="wallet-model1-2">元</text>
					</view>
					<view class="wallet-model1">提现到：</view>
					<view class="wallet-model3">
						<bank-card-item v-if="Object.keys(bankCard).length" background-color="#F3F4F5" :i="2" :item="bankCard" @click="selectBankCard" />
            <empty-bank-card-item v-else @add="toAddCard"/>
					</view>
				</view>
			</template>
		</model-slot>

		<iphonex-bottom z-index="99" :show-common-area="false" />
	</view>
</template>
<script>
	import BackContainer from "../addressManage/component/backContainer";
	import IphonexBottom from "../addressManage/component/iphonexBottom";
	import BottomOperate from "../addressManage/component/bottomOperate";
	import UniIcons from "../../../uni_modules/uni-icons/components/uni-icons/uni-icons";
	import ModelSlot from "../aboutUs/modelSlot";
	import BankCardItem from "../myBankCard/bankCardItem";
	import ListEmpty from "../../place-order/orderList/listEmpty";
  import EmptyBankCardItem from "../myBankCard/emptyBankCardItem";

  let systemInfo = uni.getSystemInfoSync();

	export default {
		name: 'myWallet',
		components: {
      EmptyBankCardItem,
			ListEmpty,
			BankCardItem,
			ModelSlot,
			UniIcons,
			BottomOperate,
			IphonexBottom,
			BackContainer
		},
    computed: {
      barHeight() {
        return (systemInfo.platform == 'ios' ? 44 : 48) + systemInfo.statusBarHeight - 32
      }
    },
		data() {
			return {
				balance: 0,
				balanceList: [],
				showBalance: true,
				visible: false,
				bankCard: {},
        refresherTriggered: false,
			};
		},
		onShow() {
			this.refresh();
		},
		// onPullDownRefresh() {
		// 	this.refresh();
		// },
		methods: {
      refresherpulling(e) {
        this.refresherTriggered = true
        setTimeout(() => {
          this.refresherTriggered = false
        }, 3000)
        console.log(e, 'refresherpulling-------------------');
      },
			refresh() {
				this.queryBalanceInfo();
				this.queryBalanceList();
			},
			queryBalanceInfo() {
				this.$http.post('/b/account/queryBalance', {}, true)
					.then(res => {
						uni.stopPullDownRefresh();
						this.balance = (res.balance || 0) / 100;
					}).catch((e) => {
						uni.stopPullDownRefresh();
					})
			},
			queryBalanceList() {
				this.$http.post('/b/account/queryBookKeepPageList', {
						pageSize: 100,
						sortInfos: [{
							field: 'addTime',
							sort: 'desc',
						}]
					}, true)
					.then(res => {
						this.balanceList = res.dataList;
					})
			},
			showRule() {
				uni.showModal({
					title: '提现规则',
					content: '（1）每次提现需大于20元\n（2）提现申请审核通过后，36小时内打到您提供的本人有效银行卡上',
					showCancel: false,
				})
			},
			hide() {
				this.visible = false;
			},
			withdrawCheck() {
				if (this.balance < 20) {
					uni.showToast({ title: '最低提现金额20元', icon: 'none' })
					return;
				}

				this.$tool.actionForAuth(() => {
					this.checkBankCard();
				})
			},
			checkBankCard() {
				this.$http.post('/b/customerbank/queryPageList', {
						pageSize: 1
					}, true)
					.then(res => {
						this.bankCard = res.dataList[0];
						this.visible = true;
					})
			},
			toAddCard() {
				uni.navigateTo({
					url: `/pages/mine/bankAccount/bankAccount`,
					events: {
						onAdd: () => {
							this.showWithdraw();
						}
					}
				})
			},
			selectBankCard() {
				uni.navigateTo({
					url: `/pages/mine/myBankCard/myBankCard?isSelect=1&selectCard=${this.bankCard.bankCardNo || ''}`,
					events: {
						onSelect: (card) => {
							this.bankCard = card;
						}
					}
				})
			},
			// 提现申请
			withdrawMoney() {
				const params = {
					// cash: this.balance * 100,
					customerBankId: this.bankCard.id,
				}
				this.$http.post('/b/withdraworder/withdrawApply', params, true)
					.then(res => {
						uni.showToast({
							title: '申请提现已提交，等待管理员审核'
						})
						this.visible = false;
					})
			}
		},
	}
</script>
<style lang="scss" scoped>
	.my-wallet {
		.wallet-top {
			height: 276rpx;
			box-sizing: border-box;
			padding: 30rpx 32rpx 48rpx 64rpx;

			.wallet-rule {
				font-size: 24rpx;
				text-align: right;
				font-family: PingFang SC-Regular, PingFang SC;
				font-weight: 400;
				color: #FFFFFF;
			}

			.wallet-top-num {
				font-size: 28rpx;
				font-family: PingFang SC-Regular, PingFang SC;
				font-weight: 400;
				color: #FFFFFF;
				line-height: 36rpx;

				.eye-slash-filled {
					margin-left: 24rpx;
				}
			}

			.wallet-top-ac {
				display: flex;
				justify-content: space-between;
				margin-top: 10rpx;

				.wallet-top-number {
					font-size: 96rpx;
					font-family: PingFang SC-Semibold, PingFang SC;
					font-weight: 600;
					color: #FFFFFF;
					line-height: 113rpx;
				}

				.wallet-top-btn {
					margin-top: 20rpx;
					width: 116rpx;
					height: 64rpx;
					border-radius: 32rpx;
					border: 2rpx solid #EDEDED;
					font-size: 24rpx;
					font-family: PingFang SC-Regular, PingFang SC;
					font-weight: 400;
					color: #FFFFFF;
					@include flexCenter;
				}
			}
		}

		.wallet-center {
			box-sizing: border-box;
			padding: 15rpx 32rpx 0 32rpx;
      overflow-y: scroll;

			.wallet-item {
				display: flex;
				align-items: center;
				height: 88rpx;

				.edit-lable {
					flex-shrink: 0;
					font-size: 28rpx;
					font-family: PingFang SC-Regular, PingFang SC;
					font-weight: 400;
					color: #323335;
				}

				.edit-midle {
					margin-left: auto;
					font-size: 28rpx;
					font-family: PingFang SC-Regular, PingFang SC;
					font-weight: 400;
					color: #BDBDBD;
					text-align: right;
				}

				.edit-right {
					padding-left: 18rpx;
				}
			}

			.wallet-detail-item {
				padding: 28rpx 0;
				min-height: 140rpx;
				box-sizing: border-box;
				border-top: 1rpx solid #EBEDF0;

				.wallet-detail-item-left {
					@include flexBetween;

					.wallet-item-left1 {
						font-size: 28rpx;
						font-family: PingFang SC-Regular, PingFang SC;
						font-weight: 400;
						color: #323335;
						line-height: 36rpx;
					}

					.wallet-item-left2 {
						font-size: 32rpx;
						font-family: PingFang SC-Regular, PingFang SC;
						font-weight: 400;
						color: #323335;
						line-height: 40rpx;
					}
				}

				.wallet-detail-item-right {
					margin-top: 18rpx;
					@include flexBetween;

					.wallet-item-right1 {
						font-size: 24rpx;
						font-family: PingFang SC-Regular, PingFang SC;
						font-weight: 400;
						color: #828282;
						line-height: 32rpx;
					}

					.wallet-item-right2 {
						font-size: 24rpx;
						font-family: PingFang SC-Regular, PingFang SC;
						font-weight: 400;
						color: #34C759;
						line-height: 32rpx;
					}
				}
			}
		}
	}

	.wallet-model {
		.wallet-model1 {
			padding: 36rpx 32rpx 0 32rpx;
			font-size: 28rpx;
			font-family: PingFang SC-Regular, PingFang SC;
			font-weight: 400;
			color: #323335;
			line-height: 36rpx;

			.wallet-model1-1 {
				font-size: 36rpx;
				font-family: PingFang SC-Medium, PingFang SC;
				font-weight: 500;
				color: #FF3B30;
				line-height: 44rpx;
			}

			.wallet-model1-2 {
				font-size: 24rpx;
				font-family: PingFang SC-Regular, PingFang SC;
				font-weight: 400;
				color: #828282;
				display: inline-block;
				padding-left: 10rpx;
				line-height: 32rpx;
			}
		}

		.wallet-model3 {
			margin-top: 12rpx;
		}
	}
</style>
<style lang="scss">
	page {
		background-color: #ECEDF9;
	}
</style>

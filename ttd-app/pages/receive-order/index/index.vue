<!-- 接单 -->
<template>
	<view>
		<custom-navbar title="接单" :show-left-icon="false" />

		<back-container>

			<template v-slot:headerSlot>
				<view class="receive-order">
					<view class="ro-1" v-for="i in list" :key="i.title" @click="toOrderList(i.state)">
						<view class="ro-11">{{ i.num }}</view>
						<view class="ro-12">{{ i.title }}</view>
					</view>

					<view class="ro-2" />

					<view class="ro-1 ro-16" @click="toOrderList(10)">
						<view class="ro-11">
							<image class="email-filled"
								src="https://ttd-public.obs.cn-east-3.myhuaweicloud.com/app-img/mine/teamView.svg" />
						</view>
						<view class="ro-12 ro-13">所有订单</view>
					</view>
				</view>
			</template>

			<view class="ro-3">
				<view class="ro-31" v-for="(i, index) in earningList" :key="i.num">
					<view class="ro-32">
						<text class="ro-33" :class="index == 1 ? 'ro-33-active' : ''">{{ i.num }}</text>
						<text v-if="index != 0" class="ro-34">k</text>
					</view>
					<view class="ro-35">{{ i.title }}</view>
				</view>
			</view>

		</back-container>

		<view class="ro-4">
			<view class="ro-41">我的团队：</view>
			<image v-if="showCreateBtn"
				src="https://ttd-public.obs.cn-east-3.myhuaweicloud.com/app-img/mine/iconCircleAdd.svg"
				class="add-img" />
			<text v-if="showCreateBtn" class="ro-43" @click="showCreateTeam()">创建团队</text>
		</view>

		<view class="ro-5">
			<view class="ro-51" v-for="(item, i) in teamList" :key="i" @click="toTeamDetail(item)">
				<image :src="item.teamLogo" class="ro-52" />
				<view class="ro-53">{{ item.teamName }}</view>
			</view>
		</view>

		<view class="ro-6" @click="toSet">
			<image class="ros-img"
				src="https://ttd-public.obs.cn-east-3.myhuaweicloud.com/app-img/mine/receiving-order-set-icon.svg" />
			<view class="set-title">我的接单设置</view>
			<uni-icons type="right" color="#999999"></uni-icons>
		</view>

		<edit-team ref="editTeam" btnText="创建" @onSave="createTeam" />

		<!-- 协议弹层 -->
		<agreement-modal ref="agreementModal"></agreement-modal>
	</view>
</template>
<script>
	import BackContainer from "../../mine/addressManage/component/backContainer";
	import UniIcons from "../../../uni_modules/uni-icons/components/uni-icons/uni-icons";
	import EditTeam from "../myTeam/editTeam";

	export default {
		components: {
			UniIcons,
			BackContainer,
			EditTeam
		},
		data() {
			return {
				list: [{
						num: 0,
						title: '待报价',
						state: 10
					},
					{
						num: 0,
						title: '待确认',
						state: 20
					},
					{
						num: 0,
						title: '待开始',
						state: 30
					},
					{
						num: 0,
						title: '待完工',
						state: 40
					},
				],
				earningList: [{
						num: 0,
						title: '完成订单'
					},
					{
						num: 0,
						title: '总收益'
					},
					{
						num: 0,
						title: '月平均收益'
					},
				],
				teamList: [],
				showCreateBtn: false,
			};
		},
		onReady() {
			
		},
		onShow() {
			this.$tool.actionForLogin(() => {
				this.refresh();
			});
			this.$store.dispatch('queryAuthenticationInfo')
		},
		onPullDownRefresh() {
			this.$tool.actionForLogin(() => {
				this.refresh();
			});
		},
		methods: {
			refresh() {
				this.queryMyTeamList();
				this.queryOrderCount();
				this.queryProfitStatistics();
			},
			queryOrderCount() {
				this.$http.post('/b/orderreceive/orderNumStatistics', {}, true)
					.then(res => {
						uni.stopPullDownRefresh();
						this.list[0].num = res.unQuoteNum;
						this.list[1].num = res.unConfirmNum;
						this.list[2].num = res.unStartNum;
						this.list[3].num = res.unCompleteNum;
					}).catch((e) => {
						uni.stopPullDownRefresh();
					})
			},
			queryProfitStatistics() {
				this.$http.post('/b/orderreceive/profitStatistics', {}, true)
					.then(res => {
						this.earningList[0].num = res.completeNum;
						this.earningList[1].num = res.totalProfit / 1000;
						this.earningList[2].num = res.monthAvgProfit / 1000;
					})
			},
			queryMyTeamList() {
				this.$http.post('/b/teaminfo/teamList', {}, true)
					.then(res => {
						this.teamList = res || [];
						let hasTeam = false;
						(res || []).forEach((t) => {
							if (t.leaderFlag) {
								hasTeam = true
							}
						});
						this.showCreateBtn = !hasTeam;
					})
			},
			toOrderList(state) {
				this.$tool.actionForAuth(() => {
					this.$tool.masterWorker(() => {
						uni.navigateTo({
							url: `/pages/place-order/orderList/orderList?isPlaceOrder=0&state=${state}`,
						})
					})
				});
			},
			showCreateTeam() {
				this.$tool.actionForAuth(() => {
					this.$tool.masterWorker(() => {
						this.$refs.editTeam.show();
					})
				});
			},
			createTeam(teamLogo, teamName, teamIntroduce) {
				if (!teamLogo) {
					uni.showToast({ title: '请上传团队头像', icon: 'none' });
					return;
				}
				if (!teamName) {
					uni.showToast({ title: '请输入团队名称', icon: 'none' });
					return;
				}
				if (!teamIntroduce) {
					uni.showToast({ title: '请输入团队介绍', icon: 'none' });
					return;
				}
				const params = {
					teamLogo,
					teamName,
					teamIntroduce,
				}
				this.$http.post('/b/teaminfo/add', params, true)
					.then(res => {
						uni.showToast({
							title: '创建成功'
						});
						this.queryMyTeamList();
					})
			},
			toTeamDetail(team) {
				this.$tool.actionForAuth(() => {
					this.$tool.masterWorker(() => {
						uni.navigateTo({
							url: `/pages/receive-order/myTeam/myTeam?id=${team.id}`
						})
					})
				});
			},
			toSet() {
				this.$tool.actionForAuth(() => {
					this.$tool.masterWorker(() => {
						uni.navigateTo({
							url: `/pages/receive-order/orderToSet/orderToSet`
						})
					})
				});
			}
		}
	}
</script>
<style lang="scss" src="./style.scss" scoped>

</style>
<style lang="scss" scoped>
	.receive-order {
		display: flex;
		padding: 48rpx 36rpx 64rpx 46rpx;

		.ro-1 {
			min-width: 116rpx;
			height: 116rpx;
			margin-right: 12rpx;

			.ro-11 {
				font-size: 52rpx;
				font-family: PingFang SC-Medium, PingFang SC;
				font-weight: 500;
				color: #FFFFFF;
				line-height: 64rpx;
				text-align: center;
			}

			.email-filled {
				width: 50rpx;
				height: 50rpx;
			}

			.ro-12 {
				margin-top: 16rpx;
				text-align: center;
				font-size: 28rpx;
				font-family: PingFang SC-Regular, PingFang SC;
				font-weight: 400;
				color: rgba(255, 255, 255, 0.6);
				line-height: 36rpx;
			}

			.ro-13 {
				margin-top: 17rpx;
			}
		}

		.ro-16 {
			margin: 0 0 0 22rpx;
		}

		.ro-2 {
			width: 2rpx;
			height: 104rpx;
			background: #FFFFFF;
			opacity: 0.15;
		}
	}

	.ro-3 {
		height: 195rpx;
		background-color: white;
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 0 78rpx;

		.ro-31 {
			.ro-32 {
				text-align: center;

				.ro-33 {
					font-size: 36rpx;
					font-family: PingFang SC-Medium, PingFang SC;
					font-weight: 500;
					color: #5AC8FA;
					line-height: 44rpx;
				}

				.ro-34 {
					font-size: 20rpx;
					font-family: PingFang SC-Medium, PingFang SC;
					font-weight: 500;
					line-height: 44rpx;
					color: #2C3580;
					padding-left: 8rpx;
				}

				.ro-33-active {
					font-size: 52rpx;
					font-family: PingFang SC-Medium, PingFang SC;
					font-weight: 500;
					line-height: 64rpx;
				}
			}

			.ro-35 {
				text-align: center;
				font-size: 28rpx;
				font-family: PingFang SC-Regular, PingFang SC;
				font-weight: 400;
				line-height: 36rpx;
				color: #2C3580;
			}
		}
	}

	.ro-4 {
		display: flex;
		height: 104rpx;
		padding: 48rpx 32rpx 16rpx 32rpx;
		box-sizing: border-box;

		.ro-41 {
			flex: 1;
			font-size: 28rpx;
			font-family: PingFang SC-Regular, PingFang SC;
			font-weight: 400;
			color: #4F4F4F;
		}

		.add-img {
			width: 32rpx;
			margin-top: 5rpx;
			height: 32rpx;
		}

		.ro-43 {
			padding-left: 12rpx;
			font-size: 28rpx;
			font-family: PingFang SC-Regular, PingFang SC;
			font-weight: 400;
			color: #1059FA;
		}
	}

	.ro-5 {
		display: flex;
		flex-direction: row;
		overflow-x: scroll;
		width: 748rpx;
		height: 244rpx;
		background: #FFFFFF;
		box-sizing: border-box;
		padding: 32rpx 0 32rpx 32rpx;

		.ro-51 {
			margin-right: 32rpx;
			width: 128rpx;
			flex-shrink: 0;

			.ro-52 {
				width: 128rpx;
				height: 128rpx;
				border-radius: 8rpx;
			}

			.ro-53 {
				font-size: 28rpx;
				font-family: PingFang SC-Regular, PingFang SC;
				font-weight: 400;
				line-height: 36rpx;
				text-align: center;
				color: #323232;
			}
		}
	}

	.receive-order-set {
		height: 112rpx;
		background: #FFFFFF;
		display: flex;
		align-items: center;
		margin-top: 32rpx;
		padding: 0 32rpx;

		.ros-img {
			width: 48rpx;
			height: 48rpx;
			margin-right: 16rpx;
		}

		.ros-label {
			flex: 1;
			font-size: 32rpx;
			font-family: PingFang SC-Regular, PingFang SC;
			font-weight: 400;
			line-height: 40rpx;
			color: #3D49AB;
		}

		.ros-icon {
			padding-top: 3rpx;
		}
	}

	.ro-6 {
		display: flex;
		box-sizing: border-box;
		flex-direction: row;
		justify-content: flex-start;
		align-items: center;
		margin-top: 30rpx;
		height: 100rpx;
		background-color: #FFFFFF;
		padding-left: 30rpx;
		padding-right: 30rpx;

		.ros-img {
			width: 48rpx;
			height: 48rpx;
		}

		.set-title {
			color: #3D49AB;
			font-size: 32rpx;
			margin-left: 20rpx;
			flex: 1;
		}

	}
</style>
<style lang="scss">
	@import "../../mine/addressManage/_pageStyle.scss";
</style>

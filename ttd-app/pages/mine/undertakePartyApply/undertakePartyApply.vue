<template>
  <view>
    <custom-navbar title="承接方申请" />

    <view class="upa-list">
      <view class="upa-item" v-for="(apply, index) in applyList" :key="index">
        <view class="upa-item-2">{{ apply.name }}</view>

        <view class="upa-item-1">
          <view class="upa-item-31">
            <view class="upa-item-3">
              <view class="upa-item-4">介绍：</view>
              <view class="upa-item-5">{{ apply.remark }}</view>
            </view>
            <view class="upa-item-3">
              <view class="upa-item-4">技能：</view>
              <view class="upa-item-9">
                <view class="upa-item-10" v-for="s in apply.skillApplyList" :key="s.skillId">{{ s.nodeLink }}</view>
              </view>
            </view>
            <view class="upa-item-3">
              <view class="upa-item-4">岗位：</view>
              <view class="upa-item-5">{{ (apply.userRoleApplyList || []).map(u => u.nodeLink).join('、') }}</view>
            </view>
            <view class="upa-item-3">
              <view class="upa-item-4">项目：</view>
              <view class="upa-item-5">{{ (apply.projectApplyList || []).map(p => p.projectName).join('、') }}</view>
            </view>
            <view class="upa-item-3">
              <view class="upa-item-4">设备：</view>
              <view class="upa-item-5">{{ (apply.toolApplyList || []).map(t => t.nodeLink).join('、') }}</view>
            </view>
          </view>

          <image class="upa-item-7"
                 :src="`https://ttd-public.obs.cn-east-3.myhuaweicloud.com/app-img/mine/${apply.approveState == 0 ? 'in-processing-icon' : apply.approveState == 2 ? 'refused-icon' : 'pass-icon11'}.svg`" />

        </view>

        <view class="upa-item-8" v-if="apply.approveState == 2">原因：{{ apply.refusalReason }}</view>
      </view>

      <list-empty v-if="!applyList.length" />
    </view>

    <iphonex-bottom>
      <big-btn :button-text="applyList.length == 0 ? '申请承接方' : '再次申请' " @click="toApply"/>
    </iphonex-bottom>
  </view>
</template>
<script>
import IphonexBottom from "../addressManage/component/iphonexBottom";
import BigBtn from "../addressManage/component/bigBtn";
import ListEmpty from "../../place-order/orderList/listEmpty";
export default {
  name: "undertakePartyApply",
  components: { ListEmpty, BigBtn, IphonexBottom },
  data() {
    return {
			applyList: [],
		};
  },
	onShow() {
		this.queryApplyList();
	},
	methods: {
		queryApplyList() {
			this.$http.post('/b/applyundertaker/queryPageList', {
				pageSize: 100,
				sortInfos: [{
					field: 'addTime',
					sort: 'desc',
				}],
			}, true)
			.then(res => {
				this.applyList = res.dataList;
			})
		},
		toApply() {
			uni.navigateTo({
				url: '/pages/main/apply/apply'
			})
		}
	},
}
</script>
<style lang="scss" scoped>
.upa-list {
  padding-bottom: 200rpx;
  .upa-item {
    background-color: white;
    padding: 32rpx;
    box-sizing: border-box;
    margin-bottom: 32rpx;

    .upa-item-2 {
      font-size: 28rpx;
      font-family: PingFang SC-Regular, PingFang SC;
      font-weight: 400;
      color: #323335;
      line-height: 36rpx;
    }

    .upa-item-1 {
      display: flex;

      .upa-item-31 {
        flex: 1;

        .upa-item-3 {
          margin-top: 16rpx;
          display: flex;

          .upa-item-4 {
            flex-shrink: 0;
            font-size: 24rpx;
            font-family: PingFang SC-Regular, PingFang SC;
            font-weight: 400;
            color: #828282;
            line-height: 36rpx;
          }

          .upa-item-5 {
            font-size: 24rpx;
            font-family: PingFang SC-Regular, PingFang SC;
            font-weight: 400;
            color: #828282;
            line-height: 36rpx;
          }

          .upa-item-9 {
            display: flex;
            flex-flow: row wrap;

            .upa-item-10 {
              margin: 0 8rpx 8rpx 0;
              padding: 0 12rpx;
              @include flexCenter;
              height: 40rpx;
              border-radius: 8rpx;
              border: 2rpx solid #EDEDED;
            }
          }
        }
      }

      .upa-item-7 {
        align-self: center;
        flex-shrink: 0;
        width: 128rpx;
        height: 128rpx;
        margin: 0 16rpx 0 56rpx;
      }
    }

    .upa-item-8 {
      margin-top: 32rpx;
      width: 686rpx;
      box-sizing: border-box;
      padding: 22rpx 32rpx;
      background: #FFFBE8;
      font-size: 28rpx;
      font-family: PingFang SC-Regular, PingFang SC;
      font-weight: 400;
      color: #FF3B30;
      line-height: 36rpx;
    }
  }
}
</style>
<style lang="scss">
@import "../../mine/addressManage/_pageStyle.scss";
</style>

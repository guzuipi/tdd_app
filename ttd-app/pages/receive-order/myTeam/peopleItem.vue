<template>
  <view class="people-item-item" @click="$emit('change')">

    <image v-if="person.headImgUrl" :src="person.headImgUrl" class="people-item-img" />
    <image v-else src="https://ttd-public.obs.cn-east-3.myhuaweicloud.com/app-img/mine/MDicon.png" class="people-item-img" />

    <view class="people-item-right">
      <view class="people-item-right1">
        <view class="people-lkj-1">
          <text class="people-lkj-name">{{ person.userName }}</text>
          <text class="people-lkj-phone">{{ person.phone }}</text>
        </view>
        <view class="people-lkj-2">
          <text class="people-lkj-2-item" v-for="i in skillList" :key="i">{{ i }}</text>
        </view>
      </view>
      <image v-if="checked" src="https://ttd-public.obs.cn-east-3.myhuaweicloud.com/app-img/mine/checkBoxChecked.svg" class="people-item-right3" />
      <image v-else src="https://ttd-public.obs.cn-east-3.myhuaweicloud.com/app-img/mine/checkBoxEmpty.svg" class="people-item-right3" />
    </view>

  </view>
</template>
<script>

export default {
  name: "peopleItem",
  props: {
		person: {
			headImgUrl: '',
			userName: '',
			phone: '',
			skills: '',
		},
    tagList: {
      type: Array,
      default() {
        return ['技能1', '技能2', '技能3']
      }
    },
    checked: {
      type: Boolean,
      default: true
    }
  },
	computed: {
		skillList() {
			if (typeof this.person.skills == 'string') {
				return (this.person.skills || '').split('、').slice(0, 3);
			} else {
				return (this.person.skills || []).slice(0, 3);
			}
		},
	},
}
</script>
<style scoped lang="scss">
.people-item-item {
  padding: 32rpx 0 0 32rpx;
  display: flex;

  .people-item-img {
    width: 96rpx;
    height: 96rpx;
    border-radius: 8rpx;
    padding-right: 20rpx;
  }

  .people-item-right {
    padding-bottom: 32rpx;
    border-bottom: 1rpx solid #EBEDF0;
    flex: 1;
    display: flex;

    .people-item-right1 {
      flex: 1;

      .people-lkj-1 {
        .people-lkj-name {
          font-size: 28rpx;
          font-family: PingFang SC-Regular, PingFang SC;
          font-weight: 400;
          color: #323335;
          line-height: 36rpx;
        }

        .people-lkj-phone {
          padding-left: 16rpx;
          font-size: 24rpx;
          font-family: PingFang SC-Regular, PingFang SC;
          font-weight: 400;
          color: #828282;
          line-height: 36rpx;
        }
      }

      .people-lkj-2 {
        margin-top: 16rpx;
        display: flex;
        flex-flow: wrap row;

        .people-lkj-2-item {
          padding: 0 8rpx;
          height: 32rpx;
          background: #EDEDED;
          border-radius: 8rpx;
          @include flexCenter;
          font-size: 24rpx;
          font-family: PingFang SC-Regular, PingFang SC;
          font-weight: 400;
          color: #4F4F4F;
          margin: 0 8rpx 8rpx 0;
        }
      }
    }

    .people-item-right2 {
      align-self: center;
      width: 80rpx;
      height: 48rpx;
      border-radius: 32rpx;
      border: 2rpx solid #EDEDED;
      font-size: 24rpx;
      font-family: PingFang SC-Regular, PingFang SC;
      font-weight: 400;
      color: #4F4F4F;
      @include flexCenter;
    }

    .people-item-right3 {
      flex-shrink: 0;
      align-self: center;
      width: 48rpx;
      height: 48rpx;
      margin: 0 32rpx 0 16rpx;
    }
  }
}
</style>

<template>
  <view>
    <view class="offer-3">
      {{ label }}
      <red-star v-if="required" />
    </view>

    <view class="offer-4">
      <textarea
          class="offer-6"
          :value="value" @input="input"
          placeholder-class="placeholder-class"
          :placeholder="placeholder"
      />
    </view>
  </view>
</template>
<script>
import RedStar from "./redStar";

export default {
  name: "addRemark",
  components: { RedStar },
  props: {
    value: {
      type: String,
      default: ''
    },
    label: {
      type: String,
      default: '备注：'
    },
    required: Boolean,
    placeholder: {
      type: String,
      default: '可以的话，多少写点！\n 方便工作人员快速排队故障。'
    },
  },
  methods: {
    input(e) {
      this.$emit('input', e.detail.value)
    }
  }
}
</script>
<style scoped lang="scss">
.offer-3 {
  padding: 32rpx 32rpx 16rpx 32rpx;
  font-size: 28rpx;
  font-family: PingFang SC-Regular, PingFang SC;
  font-weight: 400;
  color: #333333;
  line-height: 36rpx;
}

.offer-4 {
  height: 144rpx;
  margin: 0rpx 32rpx;
  padding: 24rpx 32rpx;
  background: #F3F4F5;
  box-sizing: border-box;
  border-radius: 4rpx;

  .offer-6 {
    height: 96rpx;
    width: 622rpx;
    font-size: 28rpx;
    font-family: PingFang SC-Regular, PingFang SC;
    font-weight: 400;
    color: #828282;
    line-height: 36rpx;
    background: #F3F4F5;
  }
}
</style>

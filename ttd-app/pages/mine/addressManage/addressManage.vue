<template>
  <view class="address-manage">
    <custom-navbar title="地址管理" />
    <back-container>
      <view>
        <address-item v-for="(item, index) in addressList" :key="index" :address="item" @onClick="itemClick"
                      @rightClick="rightClick" />

        <list-empty v-if="!addressList.length" />
      </view>
    </back-container>

    <iphonex-bottom>
      <big-btn button-text="新增地址" @click="addAddress"/>
    </iphonex-bottom>

  </view>
</template>

<script>
import BackContainer from './component/backContainer';
import AddressItem from './component/addressItem';
import IphonexBottom from "./component/iphonexBottom";
import BigBtn from "./component/bigBtn";
import ListEmpty from "../../place-order/orderList/listEmpty";

export default {
  components: { ListEmpty, BigBtn, IphonexBottom, AddressItem, BackContainer },
  data() {
    return {
      isSelect: false,
      addressList: []
    };
  },
  onLoad(option) {
    if (option.isSelect) {
      this.isSelect = option.isSelect == '1';
    }
  },
  onReady() {
  },
  onShow() {
    this.queryAddressData();
  },
  methods: {
    itemClick(address) {
      if (this.isSelect) {

        // 传递选择的地址
        const eventChannel = this.getOpenerEventChannel();
        eventChannel.emit('onSelect', address);

        uni.navigateBack({});

      } else {
        uni.navigateTo({ url: `/pages/mine/editAddress/editAddress?id=${address.id}` });
      }
    },
    rightClick(id) {
      uni.navigateTo({ url: `/pages/mine/editAddress/editAddress?id=${id}` });
    },
		addAddress() {
			uni.navigateTo({ url: '/pages/mine/editAddress/editAddress' });
		},
    queryAddressData() {
      this.$http.post('/b/customeraddress/queryPageList', { pageSize: 1000 }, true)
          .then(res => {
            this.addressList = res.dataList;
          })
    }
  }
}
</script>
<style lang="scss">
page {
  background-color: #ECEDF9;
}
</style>

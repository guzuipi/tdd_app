<template>
  <view class="edit-invoice">

    <custom-navbar :title="`${id === 0 ? '添加': '编辑'}发票抬头`" />

    <back-container>
      <template v-slot:headerSlot>
        <blue-tab v-if="id == 0" :active-key="activeKey" :list="tabList" @change="change" />
      </template>


      <!--企业-->
      <view class="edit-in-addr" v-if="activeKey === 0">
        <view class="edit-in-ac-item">
          <view class="edit-in-ac-lable">公司名称</view>
          <input class="edit-in-ac-midle input-sty" :value="name" @input="(e) => onInput(e, 'name')" placeholder="请务必输入" placeholder-class="input-placeholder" />
        </view>

        <view class="edit-in-ac-item">
          <view class="edit-in-ac-lable">统一税号</view>
          <input class="edit-in-ac-midle input-sty" :value="dutyNo" @input="(e) => onInput(e, 'duty')" placeholder="请务必输入" placeholder-class="input-placeholder" />
        </view>

        <view class="edit-in-ac-item">
          <view class="edit-in-ac-lable">单位地址</view>
          <input class="edit-in-ac-midle input-sty" :value="address" @input="(e) => onInput(e, 'address')" placeholder="请输入" placeholder-class="input-placeholder" />
        </view>

        <view class="edit-in-ac-item">
          <view class="edit-in-ac-lable">电话号码</view>
          <input class="edit-in-ac-midle input-sty" :value="phone" @input="(e) => onInput(e, 'phone')" placeholder="请输入" placeholder-class="input-placeholder" />
        </view>
				
				<view class="edit-in-ac-item">
				  <view class="edit-in-ac-lable">邮箱地址</view>
				  <input class="edit-in-ac-midle input-sty" :value="email" @input="(e) => onInput(e, 'email')" placeholder="请输入" placeholder-class="input-placeholder" />
				</view>

        <view class="edit-in-ac-item">
          <view class="edit-in-ac-lable">开户银行</view>
          <input class="edit-in-ac-midle input-sty" :value="openingBank" @input="(e) => onInput(e, 'bank')" placeholder="请输入" placeholder-class="input-placeholder" />
        </view>

        <view class="edit-in-ac-item">
          <view class="edit-in-ac-lable">银行帐户</view>
          <input class="edit-in-ac-midle input-sty" :value="bankAccount" @input="(e) => onInput(e, 'account')" placeholder="请输入" placeholder-class="input-placeholder" />
        </view>

        <view class="up-box">
          <view class="up-box-title">营业执照</view>
          <view class="up-box-area" @click="chooseImage()">
						<view v-if="businessLicense">
							 <image class="up-box-langou" :src="businessLicense" />
						</view>
            <view v-else class="up-box-center">
              <image class="up-box-center-image" src="https://ttd-public.obs.cn-east-3.myhuaweicloud.com/app-img/mine/upload-image-icon.svg" />
              <view class="up-box-center-text">添加照片</view>
            </view>
          </view>
        </view>
      </view>


      <!--个人-->
      <view class="edit-in-addr" v-if="activeKey === 1">
        <view class="edit-in-ac-item">
          <view class="edit-in-ac-lable">姓名</view>
          <input class="edit-in-ac-midle input-sty" :value="name" @input="(e) => onInput(e, 'name')" placeholder="请务必输入" placeholder-class="input-placeholder" />
        </view>

        <view class="edit-in-ac-item">
          <view class="edit-in-ac-lable">身份证</view>
          <input class="edit-in-ac-midle input-sty" :value="dutyNo" @input="(e) => onInput(e, 'duty')" placeholder="请务必输入" placeholder-class="input-placeholder" />
        </view>

        <view class="edit-in-ac-item">
          <view class="edit-in-ac-lable">电话</view>
          <input class="edit-in-ac-midle input-sty" :value="phone" @input="(e) => onInput(e, 'phone')" placeholder="请输入" placeholder-class="input-placeholder" />
        </view>
				
				<view class="edit-in-ac-item">
				  <view class="edit-in-ac-lable">邮箱</view>
				  <input class="edit-in-ac-midle input-sty" :value="email" @input="(e) => onInput(e, 'email')" placeholder="请输入" placeholder-class="input-placeholder" />
				</view>

        <view class="edit-in-ac-item">
          <view class="edit-in-ac-lable">地址</view>
          <input class="edit-in-ac-midle input-sty" :value="address" @input="(e) => onInput(e, 'address')" placeholder="请输入" placeholder-class="input-placeholder" />
        </view>
      </view>

    </back-container>

    <iphonex-bottom>
      <bottom-operate @operateDel="operateDel" @operateSave="operateSave" />
    </iphonex-bottom>
  </view>
</template>

<script>
import BackContainer from "../addressManage/component/backContainer";
import IphonexBottom from "../addressManage/component/iphonexBottom";
import BottomOperate from "../addressManage/component/bottomOperate";
import BlueTab from "../addressManage/component/blueTab";

export default {
  name: 'editInvoice',
  components: { BlueTab, BottomOperate, IphonexBottom, BackContainer },
  data() {
    return {
			id: 0,
			name: '',
			phone: '',
			email: '',
			dutyNo: '',
			address: '',
			openingBank: '',
			bankAccount: '',
			businessLicense: '',
      activeKey: 0,
      tabList: [
        { text: '企业', key: 0 },
        { text: '个人', key: 1 },
      ],
    }
  },
	onLoad(option) {
		if (option.id) { // 编辑地址
		  this.id = Number(option.id);
			this.queryInvoiceInfo(this.id);
		}
	},
  methods: {
    change(data) {
      this.activeKey = data;
    },
		queryInvoiceInfo(id) {
			this.$http
				.post('/b/customerinvoiceinfo/query', { id }, true)
				.then(res => {
					this.activeKey = res.type;
					this.name = res.name;
					this.phone = res.phone;
					this.email = res.email;
					this.dutyNo = res.dutyNo;
					this.address = res.address;
					this.openingBank = res.openingBank;
					this.bankAccount = res.bankAccount;
					this.businessLicense = res.businessLicense;
				})
		},
		onInput(e, type) {
			const text = e.target.value;
			if (type == 'name') {
				this.name = text;
			} else if (type == 'duty') {
				this.dutyNo = text;
			} else if (type == 'address') {
				this.address = text;
			} else if (type == 'phone') {
				this.phone = text;
			} else if (type == 'email') {
				this.email = text;
			} else if (type == 'bank') {
				this.openingBank = text;
			} else if (type == 'account') {
				this.bankAccount = text;
			}
		},
		chooseImage() {
			uni.chooseImage({
			    sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
			    success: (res) => {
							const path = res.tempFilePaths[0];
							this.uploadImage(path);
			    }
			});
		},
		uploadImage(path) {
			const param = {
				file: path,
			};
			this.$http.upload({ path }, true)
			.then(res=>{
				this.businessLicense = res;
			});
		},
		checkParams() {
			if (!this.name) {
				uni.showToast({ title:  `请输入${this.activeKey == 0 ? '公司名称' : '姓名'}`, icon:  'none' });
				return false;
			}
			if (!this.dutyNo) {
				uni.showToast({ title: `请输入${this.activeKey == 0 ? '统一税号' : '身份证'}`, icon:  'none' });
				return false;
			}
			if (!this.address) {
				uni.showToast({ title: `请输入${this.activeKey == 0 ? '单位地址' : '地址'}`, icon:  'none' });
				return false;
			}
			if (!this.phone) {
				uni.showToast({ title:  '请输入电话号码', icon:  'none' });
				return false;
			}
			if (!this.email) {
				uni.showToast({ title:  '请输入邮箱地址', icon:  'none' });
				return false;
			}
			if (this.activeKey == 0 && !this.openingBank) {
				uni.showToast({ title:  '请输入开户银行', icon:  'none' });
				return false;
			}
			if (this.activeKey == 0 && !this.bankAccount) {
				uni.showToast({ title:  '请输入银行帐户', icon:  'none' });
				return false;
			}
			if (this.activeKey == 0 && !this.businessLicense) {
				uni.showToast({ title:  '请选择营业执照', icon:  'none' });
				return false;
			}
			return true;
		},
		// 新增/编辑发票
    operateSave() {

			if (!this.checkParams()) return;

			const params  = {
				id: this.id,
				type: this.activeKey,
				name: this.name,
				phone: this.phone,
				email: this.email,
				dutyNo: this.dutyNo,
				address: this.address,
				openingBank: this.openingBank,
				bankAccount: this.bankAccount,
				businessLicense: this.businessLicense,
			};

			const url = `/b/customerinvoiceinfo/${this.id ? 'update' : 'add'}`;

			this.$http
				.post(url, params, true)
				.then(res => {
					uni.showToast({
						title: '编辑成功',
					  success: () => {
						uni.navigateBack({});
					  },
					});
				})
    },
		// 删除发票信息
    operateDel() {
      uni.showModal({
      	content: '是否删除发票?',
      	success: (res) => {
					if (res.confirm) {
      		  this.deleleInvoice();
					}
      	},
      })
    },
		deleleInvoice() {
			this.$http
				.post('/b/customerinvoiceinfo/delete', { id: this.id }, true)
				.then(res => {
					uni.showToast({
						title: '删除成功',
					  success: () => {
						uni.navigateBack({});
					  },
					});
				})
		},
  }
}
</script>
<style lang="scss" scoped>
@import "../../mine/addressManage/_inputStyle.scss";

.edit-invoice {
  .edit-in-addr {
    box-sizing: border-box;
    padding: 15rpx 0 0 32rpx;

    .edit-in-ac-item {
      display: flex;
      align-items: center;
      height: 88rpx;
      padding-right: 32rpx;

      &:not(:nth-last-child(1)) {
        border-bottom: 1rpx solid #EBEDF0;
      }

      .edit-in-ac-lable {
        flex-shrink: 0;
        font-size: 28rpx;
        font-family: PingFang SC-Regular, PingFang SC;
        font-weight: 400;
        color: #323335;
      }

      .edit-in-ac-midle {
        margin-left: auto;
        font-size: 28rpx;
        font-family: PingFang SC-Regular, PingFang SC;
        font-weight: 400;
        color: #BDBDBD;
        text-align: right;
      }
    }

    .up-box {
      padding-bottom: 48rpx;

      .up-box-title {
        margin: 46rpx 0 32rpx 0;
        font-size: 28rpx;
        font-family: PingFang SC-Regular, PingFang SC;
        font-weight: 400;
        text-align: center;
        color: #323335;
      }

      .up-box-area {
        @include flexCenter;
        width: 480rpx;
        height: 320rpx;
        background: #F3F4F5;
        margin: 0 auto 0 104rpx;

        .up-box-langou {
          width: 480rpx;
          height: 320rpx;
        }

        .up-box-center {
          text-align: center;

          .up-box-center-image {
            width: 48rpx;
            height: 48rpx;
          }

          .up-box-center-text {
            font-size: 24rpx;
            font-family: PingFang SC-Light, PingFang SC;
            font-weight: 300;
            color: #828282;
          }
        }
      }
    }
  }

  /deep/ .input-placeholder {
    font-size: 28rpx;
    font-family: PingFang SC-Regular, PingFang SC;
    font-weight: 400;
    color: #BDBDBD;
  }
}
</style>
<style lang="scss">
@import "../addressManage/_pageStyle.scss";
</style>

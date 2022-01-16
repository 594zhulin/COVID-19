<template>
	<view class="add-form">
		<view class="add-form-item">
			<view class="add-form-label">场所名称</view>
			<view class="add-form-content select">
				<picker mode="selector" :range="regions" range-key="Name" @change="getRegionCode">
					<input class="add-form-input" v-model="regionName" placeholder="请选择场所" disabled />
				</picker>
			</view>
		</view>
		<view class="add-form-item">
			<view class="add-form-label">布点位置</view>
			<view class="add-form-content"><input class="add-form-input" type="text" v-model="positionName" placeholder="请输入布点位置" /></view>
		</view>
		<view class="add-form-item">
			<view class="add-form-label">疫情负责人</view>
			<view class="add-form-content"><input class="add-form-input" type="text" v-model="directorName" placeholder="请输入疫情负责人姓名" /></view>
		</view>
		<view class="add-form-item">
			<view class="add-form-label">手机号</view>
			<view class="add-form-content"><input class="add-form-input" type="text" v-model="directorPhone" placeholder="请输入疫情负责人手机号" /></view>
		</view>
		<view class="add-form-item">
			<view class="add-form-label">验证码</view>
			<view class="add-form-content">
				<input class="add-form-input" type="text" v-model="verifyCode" placeholder="请输入验证码" />
				<view class="add-form-text" v-show="countDown === 60" @click="getVerifyCode">获取验证码</view>
				<view class="add-form-text" v-show="countDown !== 60">{{ countDown }}s</view>
			</view>
		</view>
		<view class="add-form-btn" @click="submit">提交备案</view>
		<device></device>
	</view>
</template>

<script>
import device from './device.vue';
export default {
	components: {
		device
	},
	data() {
		return {
			regions: [],
			directorPhoneReg: /^1[3|4|5|6|7|8|9][0-9]{9}$/,
			timer: null,
			countDown: 60,

			// 提交备案请求需要的参数
			parentCode: '',
			regionName: '',
			positionName: '',
			directorName: '',
			directorPhone: '',
			verifyCode: ''
		};
	},
	onLoad(option) {
		this.getRegionData(option.code || '1001');
	},
	destroyed() {
		clearTimeout(this.timer);
	},
	methods: {
		getRegionData(code) {
			uni.request({
				url: '/regions',
				data: {
					code
				},
				success: res => {
					const { Success, ResData, ErrMsg } = res.data;
					if (Success) {
						this.regions = ResData;
					} else {
						uni.showToast({
							icon: 'none',
							title: ErrMsg
						});
					}
					console.log(res.data);
				},
				fail: err => {
					uni.showToast({
						icon: 'none',
						title: err
					});
				}
			});
		},
		getRegionCode(e) {
			this.regionName = this.regions[e.detail.value].Name;
			this.parentCode = this.regions[e.detail.value].Code;
		},
		validatePhone() {
			if (this.directorPhone === '') {
				uni.showToast({
					icon: 'none',
					title: '请输入疫情负责人手机号'
				});
				return false;
			}
			if (!this.directorPhoneReg.test(this.directorPhone)) {
				uni.showToast({
					icon: 'none',
					title: '请输入正确的手机号'
				});
				return false;
			}
			return true;
		},
		getCountDown() {
			this.countDown = this.countDown - 1;
			this.timer = setTimeout(this.getCountDown, 1000);
			if (this.countDown === 0) {
				this.countDown = 60;
				clearTimeout(this.timer);
			}
		},
		getVerifyCode() {
			const isValidatePass = this.validatePhone();
			if (!isValidatePass) {
				return;
			}
			this.getCountDown();
			uni.showLoading({
				title: '验证码发送中'
			});
			uni.request({
				url: '/verify-code',
				method: 'POST',
				data: this.directorPhone,
				success: res => {
					const { Success, ErrMsg } = res.data;
					if (Success) {
						uni.showToast({
							icon: 'success',
							title: '验证码已发送'
						});
					} else {
						uni.showToast({
							icon: 'none',
							title: ErrMsg
						});
					}
					console.log(res.data);
				},
				fail: err => {
					uni.showToast({
						icon: 'none',
						title: err
					});
				},
				complete: () => {
					uni.hideLoading();
				}
			});
		},
		validate() {
			if (this.regionName === '') {
				uni.showToast({
					icon: 'none',
					title: '请选择场所'
				});
				return false;
			}
			if (this.positionName === '') {
				uni.showToast({
					icon: 'none',
					title: '请输入布点位置'
				});
				return false;
			}
			if (this.directorName === '') {
				uni.showToast({
					icon: 'none',
					title: '请输入疫情负责人姓名'
				});
				return false;
			}
			const isValidatePhonePass = this.validatePhone();
			if (!isValidatePhonePass) {
				return;
			}
			if (this.verifyCode === '') {
				uni.showToast({
					icon: 'none',
					title: '请输入验证码'
				});
				return false;
			}
			return true;
		},
		submit() {
			const isValidatePass = this.validate();
			if (!isValidatePass) {
				return;
			}
			uni.showLoading({
				title: '提交中，请稍后'
			});
			uni.request({
				url: '/position-add',
				method: 'POST',
				data: {
					parent_code: this.parentCode,
					region_name: this.regionName,
					director_name: this.directorName,
					director_phone: this.directorPhone,
					verify_code: this.verifyCode
				},
				success: res => {
					const { Success, ErrMsg } = res.data;
					if (Success) {
						uni.showToast({
							icon: 'success',
							title: '提交成功',
							duration: 5000,
							complete: () => {
								// uni.navigateBack({
								// 	delta: 1
								// });
							}
						});
					} else {
						uni.showToast({
							icon: 'none',
							title: ErrMsg
						});
					}
					console.log(res.data);
				},
				fail: err => {
					uni.showToast({
						icon: 'none',
						title: err
					});
				},
				complete: () => {
					uni.hideLoading();
				}
			});
		}
	}
};
</script>

<style lang="less">
.add-form {
	padding: 24px 16px;
	&-item {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin-bottom: 16px;
	}
	&-label {
		flex-shrink: 0;
		width: 70px;
		height: 20px;
		margin-right: 8px;
		font-size: 14px;
		font-family: PingFangSC-Medium, PingFang SC;
		font-weight: 500;
		color: #555555;
		line-height: 20px;
	}
	&-content {
		display: flex;
		align-items: center;
		justify-content: space-between;
		flex: 1;
		padding: 11px 12px;
		height: 44px;
		background: #fafafa;
		border-radius: 4px;
		border: 1px solid #dedede;
	}
	&-content.select {
		&::after {
			flex-shrink: 0;
			display: block;
			content: '';
			width: 24px;
			height: 24px;
			margin-right: -5px;
			background: url(../../static/arrow.png) no-repeat center;
			background-size: contain;
		}
	}
	&-input {
		flex: 1;
	}
	&-text {
		height: 20px;
		font-size: 14px;
		font-family: PingFangSC-Regular, PingFang SC;
		font-weight: 400;
		color: #4e7fed;
		line-height: 20px;
	}
	&-btn {
		margin-top: 24px;
		height: 44px;
		line-height: 44px;
		background: #4e7fed;
		border-radius: 8px;
		font-size: 18px;
		font-family: PingFangSC-Medium, PingFang SC;
		font-weight: 500;
		color: #ffffff;
		text-align: center;
	}
}
</style>

<template>
	<view class="auth-form">
		<view class="auth-form-item">
			<view class="auth-form-label">场所认证</view>
			<view class="auth-form-content"><input class="auth-form-input" type="text" v-model="regionName" placeholder="请输入场所名称" disabled /></view>
		</view>
		<view class="auth-form-item">
			<view class="auth-form-label">省市区</view>
			<view class="auth-form-content select">
				<picker mode="multiSelector" :range="regions" :value="regionIndex" range-key="name" @columnchange="getRegionData" @change="getRegionCode">
					<input class="auth-form-input" v-model="regionText" placeholder="请选择省市区" disabled />
				</picker>
			</view>
		</view>
		<view class="auth-form-item">
			<view class="auth-form-label">场所类型</view>
			<view class="auth-form-content select">
				<picker mode="selector" :range="regionTypes" range-key="text" @change="getRegionType">
					<input class="auth-form-input" v-model="regionTypeText" placeholder="请选择类型" disabled />
				</picker>
			</view>
		</view>
		<view class="auth-form-item">
			<view class="auth-form-label">场所负责人</view>
			<view class="auth-form-content"><input class="auth-form-input" type="text" v-model="directorName" placeholder="请输入场所负责人姓名" /></view>
		</view>
		<view class="auth-form-item">
			<view class="auth-form-label">手机号</view>
			<view class="auth-form-content"><input class="auth-form-input" type="text" v-model="directorPhone" placeholder="请输入场所负责人手机号" /></view>
		</view>
		<view class="auth-form-item">
			<view class="auth-form-label">验证码</view>
			<view class="auth-form-content">
				<input class="auth-form-input" type="text" v-model="verifyCode" placeholder="请输入验证码" />
				<view class="auth-form-text" v-show="countDown === 60" @click="getVerifyCode">获取验证码</view>
				<view class="auth-form-text" v-show="countDown !== 60">{{ countDown }}s</view>
			</view>
		</view>
		<view class="auth-form-btn" @click="auth">认证</view>
	</view>
</template>

<script>
import { provinces } from '../../utils/provinces.js';
import { cities } from '../../utils/cities.js';
import { areas } from '../../utils/areas.js';
export default {
	data() {
		return {
			regions: [],
			regionIndex: [0, 0, 0],
			regionText: '',
			regionTypes: [
				{
					value: 1,
					text: '街道'
				},
				{
					value: 4,
					text: '社区'
				},
				{
					value: 8,
					text: '广场'
				},
				{
					value: 9,
					text: '小区'
				}
			],
			regionTypeText: '',
			directorPhoneReg: /^1[3|4|5|6|7|8|9][0-9]{9}$/,
			timer: null,
			countDown: 60,

			// 认证请求需要的参数
			parentCode: '',
			regionName: '将军碑社区',
			countryCode: '',
			regionType: 0,
			directorName: '',
			directorPhone: '',
			verifyCode: ''
		};
	},
	onLoad(option) {
		this.parentCode = option.parentCode;
		this.regions[0] = provinces;
		this.regions[1] = cities.filter(item => item.provinceCode === this.regions[0][0].code);
		this.regions[2] = areas.filter(item => item.cityCode === this.regions[1][0].code);
	},
	destroyed() {
		clearTimeout(this.timer)
	},
	methods: {
		getRegionData(e) {
			this.regionIndex[e.detail.column] = e.detail.value;
			const column = e.detail.column;
			switch (column) {
				case 0:
					this.regions[1] = cities.filter(item => item.provinceCode === this.regions[0][e.detail.value].code);
					this.regions[2] = areas.filter(item => item.cityCode === this.regions[1][0].code);
					this.regionIndex.splice(1, 1, 0);
					this.regionIndex.splice(2, 1, 0);
					break;
				case 1:
					this.regions[2] = areas.filter(item => item.cityCode === this.regions[1][e.detail.value].code);
					this.regionIndex.splice(2, 1, 0);
					break;
				default:
					break;
			}
			this.$forceUpdate();
		},
		getRegionCode(e) {
			const provinceText = this.regions[0][e.detail.value[0]].name;
			const cityText = this.regions[1][e.detail.value[1]].name;
			const areaText = this.regions[2][e.detail.value[2]].name;
			this.regionText = [provinceText, cityText, areaText].join('/');
			this.countryCode = this.regions[2][e.detail.value[2]].code;
		},
		getRegionType(e) {
			this.regionType = this.regionTypes[e.detail.value].value;
			this.regionTypeText = this.regionTypes[e.detail.value].text;
		},
		validatePhone() {
			if (this.directorPhone === '') {
				uni.showToast({
					icon: 'none',
					title: '请输入场所负责人手机号'
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
			if (this.regionText === '') {
				uni.showToast({
					icon: 'none',
					title: '请选择省市区'
				});
				return false;
			}
			if (this.regionTypeText === '') {
				uni.showToast({
					icon: 'none',
					title: '请选择场所类型'
				});
				return false;
			}
			if (this.directorName === '') {
				uni.showToast({
					icon: 'none',
					title: '请输入场所负责人姓名'
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
		auth() {
			const isValidatePass = this.validate();
			if (!isValidatePass) {
				return;
			}
			uni.showLoading({
				title: '认证中，请稍后'
			});
			uni.request({
				url: '/region-add',
				method: 'POST',
				data: {
					parent_code: this.parentCode,
					region_name: this.regionName,
					director_name: this.directorName,
					director_phone: this.directorPhone,
					verify_code: this.verifyCode,
					region_type: this.regionType,
					country_code: this.countryCode
				},
				success: res => {
					const { Success, ErrMsg } = res.data;
					if (Success) {
						uni.showToast({
							icon: 'success',
							title: '认证成功',
							duration: 5000,
							complete: () => {
								uni.navigateBack({
									delta: 1
								});
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
.auth-form {
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

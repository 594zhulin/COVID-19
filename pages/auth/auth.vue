<template>
	<view class="auth-form">
		<view class="auth-form-item">
			<view class="auth-form-label">场所认证</view>
			<view class="auth-form-content"><input class="auth-form-input" type="text" v-model="regionName" placeholder="请输入场所名称" /></view>
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
export default {
	data() {
		return {
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
			regionName: '',
			regionType: 0,
			directorName: '',
			directorPhone: '',
			verifyCode: ''
		};
	},
	onLoad(option) {
		this.parentCode = option.parentCode;
	},
	destroyed() {
		clearTimeout(this.timer)
	},
	methods: {
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
			if (this.regionName === '') {
				uni.showToast({
					icon: 'none',
					title: '请输入场所名称'
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
					region_type: this.regionType
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
			background: url(../../assets/arrow.png) no-repeat center;
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

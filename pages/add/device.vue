<template>
	<view class="device">
		<view class="title">推荐设备</view>
		<view class="device-list">
			<view class="device-item" v-for="item in deviceData" :key="item.id">
				<view class="device-pic"></view>
				<view class="device-info">
					<view class="device-name">{{ item.name }}</view>
					<view class="device-desc">{{ item.desc }}</view>
					<view class="device-price">售价{{ item.price }}元</view>
				</view>
			</view>
		</view>
		<view class="device-factory">
			<view class="factory">
				<view class="label">生产厂家</view>
				<view class="name">{{ factory }}</view>
			</view>
			<view class="contact">
				<view class="label">联系人</view>
				<view class="info">
					<view class="name">{{ contact.name }}</view>
					<view class="phone" @click="call">{{ contact.phone }}</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			deviceData: [
				{
					id: '1',
					name: '设备名称',
					desc: '入口消毒一体自动扫码，这里写设备简介',
					price: 15980
				},
				{
					id: '2',
					name: '设备名称',
					desc: '停车场入口款  堵住漏洞，自动看体温，排查健康码',
					price: 6850
				}
			],
			factory: '成都美佳美宜智能制造',
			contact: {
				name: '朱先生',
				phone: '13298018014'
			}
		};
	},
	methods: {
		call() {
			uni.makePhoneCall({
				phoneNumber: this.contact.phone //仅为示例
			});
		}
	}
};
</script>

<style lang="less">
.device {
	padding: 32px 0 12px 0;
	&-item {
		display: flex;
		justify-content: space-between;
		margin-top: 16px;
	}
	&-info {
		flex: 1;
	}
	&-pic {
		margin-right: 24px;
		flex-shrink: 0;
		width: 140px;
		height: 140px;
		background: #d8d8d8;
	}
	&-name {
		margin-top: 8px;
		font-size: 18px;
		font-family: PingFangSC-Medium, PingFang SC;
		font-weight: 500;
		color: #222222;
		line-height: 25px;
	}
	&-desc {
		margin: 8px 0 20px 0;
		font-size: 14px;
		font-family: PingFangSC-Regular, PingFang SC;
		font-weight: 400;
		color: #555555;
		line-height: 20px;
	}
	&-price {
		font-size: 14px;
		font-family: PingFangSC-Medium, PingFang SC;
		font-weight: 500;
		color: #eb4242;
		line-height: 20px;
	}
	&-factory {
		padding: 12px 16px;
		margin-top: 19px;
		background: #f5f8ff;
		border-radius: 8px;
		.factory,
		.contact {
			display: flex;
			align-items: center;
		}
		.label {
			position: relative;
			flex-shrink: 0;
			width: 72px;
			margin-right: 15px;
			font-size: 14px;
			font-family: PingFangSC-Medium, PingFang SC;
			font-weight: 500;
			color: #555555;
			line-height: 20px;
			&::after {
				content: '';
				position: absolute;
				top: 0;
				bottom: 0;
				right: 0;
				width: 1px;
				height: 12px;
				margin: auto;
				background-color: #979797;
			}
		}
		.name {
			font-size: 14px;
			font-family: PingFangSC-Regular, PingFang SC;
			font-weight: 400;
			color: #222222;
			line-height: 20px;
		}
		.contact {
			margin-top: 8px;
			.info {
				display: flex;
				align-items: center;
			}
			.phone {
				margin-left: 8px;
				font-size: 14px;
				font-family: PingFangSC-Medium, PingFang SC;
				font-weight: 500;
				color: #4e7fed;
				line-height: 20px;
			}
		}
	}
}
</style>

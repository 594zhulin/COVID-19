<template>
	<view class="home">
		<view class="banner"></view>
		<view class="community">
			<view class="community-name">{{ communityName }}</view>
			<view class="add-btn" @click="navigateTo('/pages/auth/auth')">+添加场所</view>
		</view>
		<view class="header">
			<view class="title"></view>
			<view class="add-btn" @click="navigateTo('/pages/add/add')">+添加防疫点</view>
		</view>
		<view class="overview">
			<view class="count-list">
				<view class="count-item">
					<view class="count red">{{ redCount }}</view>
					<view class="text">红码人数</view>
				</view>
				<view class="count-item">
					<view class="count yellow">{{ yellowCount }}</view>
					<view class="text">黄码人数</view>
				</view>
				<view class="count-item">
					<view class="count">{{ overTemCount }}</view>
					<view class="text">体温异常人数</view>
				</view>
			</view>
			<view class="count-total">
				<view class="risk-item">
					<view class="text">风险上报总件数</view>
					<view class="count">{{ riskCount }}</view>
				</view>
				<view class="risk-item">
					<view class="text">排查率</view>
					<view class="count">{{ checkRate }}%</view>
				</view>
			</view>
		</view>
		<view class="chart">
			<view class="title">过往人数与扫码人数动态</view>
			<view class="legend">
				<view class="legend-item pass">过往人数</view>
				<view class="legend-item scan">扫码人数</view>
			</view>
			<view class="count-list">
				<view class="count-item">
					<view class="text">过往总人数</view>
					<view class="count">{{ allCount }}</view>
				</view>
				<view class="count-item">
					<view class="text">扫码总人数</view>
					<view class="count">{{ allCount }}</view>
				</view>
				<view class="count-item">
					<view class="text">扫码率</view>
					<view class="count">{{ scanRate }}%</view>
				</view>
			</view>
		</view>
		<view class="region">
			<view class="title">智慧防控布点</view>
			<view class="region-item" v-for="(item, index) in regionData" :key="index">
				<view class="region-name">
					<view class="name">{{ item.region_name }}</view>
					<view class="director">
						<view class="text">防疫负责人</view>
						<view class="info">{{ item.director_name }}/{{ item.director_phone }}</view>
					</view>
				</view>
				<view class="count-list">
					<view class="count-item">
						<view class="text">人流量</view>
						<view class="count">{{ item.all_count }}</view>
					</view>
					<view class="count-item">
						<view class="text">今日扫码率</view>
						<view class="count">{{ item.today_scan_count }}</view>
					</view>
					<view class="count-item">
						<view class="text">智慧布点数量</view>
						<view class="count">{{ item.device_count }}</view>
					</view>
					<view class="count-item">
						<view class="text">累计风险预警排查</view>
						<view class="count">{{ item.risk_count }}</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			code: '',
			communityName: '安宫国际社区',
			redCount: 0,
			yellowCount: 0,
			overTemCount: 0,
			riskCount: 0,
			scanCount: 0,
			allCount: 0,
			regionData: [],
			checkRate: 0,
			scanRate: 0
		};
	},
	onLoad(option) {
		this.code = option.code || '1001';
		this.getHomeData();
	},
	methods: {
		getHomeData(code) {
			uni.request({
				url: '/data',
				data: {
					code: this.code
				},
				success: res => {
					const { Success, ResData, ErrMsg } = res.data;
					if (Success) {
						const { community_name, red_count, yellow_count, over_tem_count, risk_count, scan_count, all_count, month_data, region_data } = ResData;
						this.communityName = community_name || '安宫国际社区健康的JFK健康的减肥扩大解放的疯狂酒店开房间';
						this.redCount = red_count;
						this.yellowCount = yellow_count;
						this.overTemCount = over_tem_count;
						this.riskCount = risk_count;
						this.scanCount = scan_count;
						this.allCount = all_count;
						this.regionData = region_data;
						let check_rate = ((red_count + yellow_count + over_tem_count) / risk_count) * 100;
						let scan_rate = (scan_count / all_count) * 100;
						this.checkRate = check_rate < 100 ? check_rate.toFixed(2) : check_rate;
						this.scanRate = scan_rate < 100 ? scan_rate.toFixed(2) : scan_rate;
					} else {
						uni.showToast({
							icon: 'none',
							title: ErrMsg
						});
					}
				},
				fail: err => {
					uni.showToast({
						icon: 'none',
						title: err
					});
				}
			});
		},
		navigateTo(url) {
			uni.navigateTo({
				url: `${url}?code=${this.code}`
			});
		}
	}
};
</script>

<style lang="less">
.home {
	padding-top: 49px;
	&::before {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		content: '';
		height: 240px;
		background: linear-gradient(180deg, #86b6f7 0%, #4e7fed 100%);
		z-index: -1;
	}
	&::after {
		position: fixed;
		content: '';
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background-color: #f7f7f7;
		z-index: -2;
	}
	.banner {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		height: 240px;
		background: url(../../assets/banner.png) no-repeat center top;
		background-size: cover;
		z-index: -1;
	}
	.community,
	.header,
	.overview,
	.chart {
		margin: 0 16px;
	}
	.community {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin-bottom: 3px;
	}
	.community-name {
		flex: 1;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
		font-size: 18px;
		font-family: PingFangSC-Medium, PingFang SC;
		font-weight: 500;
		color: #ffffff;
	}
	.add-btn {
		flex-shrink: 0;
		width: 94px;
		height: 36px;
		line-height: 36px;
		background: #fffafa;
		border-radius: 22px;
		font-size: 14px;
		font-family: PingFangSC-Medium, PingFang SC;
		font-weight: 500;
		color: #4e7fed;
		text-align: center;
	}
	.header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		.title {
			flex: 1;
			height: 89px;
			background: url(../../assets/title.png) no-repeat center;
			background-position: -6px -6px;
		}
	}
	.overview {
		background: #ffffff;
		border-radius: 12px;
		margin-bottom: 12px;
		.count-list {
			position: relative;
			display: flex;
			align-items: center;
			justify-content: space-between;
			padding: 16px 12px 20px 13px;
			&::after {
				content: '';
				position: absolute;
				left: 20px;
				right: 20px;
				bottom: 0;
				height: 1px;
				opacity: 0.5;
				background-color: #cbcbcb;
			}

			.count {
				margin-bottom: 4px;
				line-height: 33px;
				font-size: 24px;
				font-family: PingFangSC-Semibold, PingFang SC;
				font-weight: 600;
				color: #222222;
				text-align: center;
			}
			.count.red {
				color: #eb4242;
			}
			.count.yellow {
				color: #fec71f;
			}
			.text {
				width: 106px;
				font-size: 14px;
				font-family: PingFangSC-Regular, PingFang SC;
				font-weight: 400;
				color: #555550;
				line-height: 20px;
				text-align: center;
			}
		}
		.count-total {
			display: flex;
			align-items: center;
			justify-content: space-between;
			padding: 16px 20px 20px 20px;
			.risk-item {
				display: flex;
				align-items: center;
			}
			.text {
				margin-right: 12px;
				font-size: 14px;
				font-family: PingFangSC-Regular, PingFang SC;
				font-weight: 400;
				color: #555555;
				line-height: 20px;
			}
			.count {
				font-size: 16px;
				font-family: PingFangSC-Medium, PingFang SC;
				font-weight: 500;
				color: #222222;
				line-height: 22px;
			}
		}
	}
	.chart {
		background: #ffffff;
		border-radius: 12px;
		margin-bottom: 24px;
		.title {
			padding: 19px 0 8px 0;
			font-size: 20px;
			font-family: PingFangSC-Medium, PingFang SC;
			font-weight: 500;
			color: #222222;
			line-height: 28px;
			text-align: center;
		}
		.legend {
			display: flex;
			align-items: center;
			justify-content: space-between;
			margin: 0 101px 0 102px;
		}
		.legend-item {
			display: flex;
			align-items: center;
			font-size: 12px;
			font-family: PingFangSC-Regular, PingFang SC;
			font-weight: 400;
			color: #555550;
			line-height: 17px;
			&::before {
				display: block;
				content: '';
				width: 10px;
				height: 10px;
				margin-right: 5px;
			}
		}
		.legend-item.pass::before {
			background-color: #546dfa;
		}
		.legend-item.scan::before {
			background-color: #2ae0aa;
		}
		.count-list {
			display: flex;
			align-items: center;
			justify-content: space-between;
			padding: 24px 19px 24px 20px;
		}
		.count-item {
			&:last-child {
				.text,
				.count {
					width: 72px;
				}
			}
			.text {
				width: 100px;
				margin-bottom: 4px;
				font-size: 14px;
				font-family: PingFangSC-Regular, PingFang SC;
				font-weight: 400;
				color: #555555;
				line-height: 20px;
			}
			.count {
				width: 100px;
				font-size: 16px;
				font-family: PingFangSC-Medium, PingFang SC;
				font-weight: 500;
				color: #222222;
				line-height: 22px;
			}
		}
	}
	.region {
		padding: 24px 16px 15px 16px;
		background-color: #fff;
		.title {
			margin-bottom: 16px;
			font-size: 18px;
			font-family: PingFangSC-Medium, PingFang SC;
			font-weight: 500;
			color: #222222;
			line-height: 25px;
		}
	}
	.region-item {
		margin-bottom: 12px;
		background: #f7f7f7;
		border-radius: 12px;
		.count-list {
			display: flex;
			align-items: center;
			justify-content: space-between;
			flex-wrap: wrap;
			padding: 2px 16px 20px 20px;
		}
		.count-item {
			display: flex;
			align-items: center;
			margin-top: 14px;
			flex: 0 0 50%;
			.text {
				margin-right: 8px;
				font-size: 14px;
				font-family: PingFangSC-Regular, PingFang SC;
				font-weight: 400;
				color: #555555;
				line-height: 20px;
			}
			.count {
				font-size: 16px;
				font-family: PingFangSC-Medium, PingFang SC;
				font-weight: 500;
				color: #222222;
				line-height: 22px;
			}
		}
	}
	.region-name {
		display: flex;
		align-items: center;
		justify-content: space-between;
		height: 68px;
		padding: 0 16px 0 20px;
		background: linear-gradient(90deg, #86b6f7 0%, #4e7fed 100%);
		border-radius: 12px 12px 0 0;
		.name {
			font-size: 18px;
			font-family: PingFangSC-Medium, PingFang SC;
			font-weight: 500;
			color: #fffafa;
			line-height: 25px;
		}
		.director {
			.text {
				margin-bottom: 2px;
				font-size: 12px;
				font-family: PingFangSC-Regular, PingFang SC;
				font-weight: 400;
				color: #ffffff;
				line-height: 17px;
				opacity: 0.8;
			}
			.info {
				font-size: 14px;
				font-family: PingFangSC-Regular, PingFang SC;
				font-weight: 400;
				color: #ffffff;
				line-height: 20px;
			}
		}
	}
}
</style>

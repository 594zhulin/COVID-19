<script>
export default {
	onLaunch: function() {
		console.log('App Launch');
		uni.addInterceptor('request', {
			invoke(args) {
				// request 触发前拼接 url
				args.url = 'http://175.24.114.188:801/front' + args.url;
			},
			success(args) {
				console.log('interceptor-success', args);
			},
			fail(err) {
				console.log('interceptor-fail', err);
			},
			complete(res) {
				console.log('interceptor-complete', res);
			}
		});
	},
	onShow: function() {
		console.log('App Show');
	},
	onHide: function() {
		console.log('App Hide');
	}
};
</script>

<style>
/*每个页面公共css */
view {
	box-sizing: border-box;
}
.input-placeholder {
	height: 22px;
	font-size: 16px;
	font-family: PingFangSC-Regular, PingFang SC;
	font-weight: 400;
	color: #bebebe;
	line-height: 22px;
}
</style>

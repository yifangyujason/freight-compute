<template>
	<view style="padding: 16px;">
		<view style="text-align: center; font-size: x-large; margin-bottom: 16px;">总运费计算</view>
		<u--form :model="formModel" :rules="formRules" ref="calcForm">
			<!-- 地区 -->
			<u-form-item label="地区" prop="region" borderBottom @click="showRegion = true;">
				<u--input v-model="formModel.region" disabled disabledColor="#ffffff" placeholder="请选择地区"
					border="none"></u--input>
				<u-icon slot="right" name="arrow-right"></u-icon>
			</u-form-item>
			<u-form-item label="运费" prop="freight" borderBottom>
				<u--input v-model="formModel.freight" type="digit" placeholder="请输入运费" border="none"></u--input>
			</u-form-item>
			<u-form-item label="吨数" prop="tons" borderBottom>
				<u--input v-model="formModel.tons" type="digit" placeholder="请输入吨数" border="none"></u--input>
			</u-form-item>
		</u--form>
		<u-button @click="calculate" type="success" style="margin-top: 16px;">计算</u-button>
		<!-- 地区选择 -->
		<u-action-sheet :show="showRegion" :actions="regionOptions" title="请选择地区" @close="showRegion = false"
			@select="regionSelect">
		</u-action-sheet>
		<view style="margin-top: 16px; font-size: large; text-align: center;">
			总运费: {{ totalFreight }}
		</view>
	</view>
</template>

<script>
	import appConfig from '@/env';
	export default {
		data() {
			return {
				formModel: {
					region: '',
					freight: '',
					tons: '',
				},
				formRules: {
					region: {
						type: 'string',
						required: true,
						message: '请输入地区',
						trigger: ['blur', 'change'],
					},
					freight: {
						type: 'number',
						required: true,
						message: '请输入运费',
						trigger: ['blur', 'change'],
					},
					tons: {
						type: 'number',
						required: true,
						message: '请输入吨数',
						trigger: ['blur', 'change'],
					},
				},
				showRegion: false, // 地区选择框显示状态
				regionOptions: [], // 存放动态获取的地区数据
				totalFreight: 0,
			};
		},
		onLoad() {
			// 页面加载时获取地区数据
			this.getRegionOptions();
		},
		methods: {
			// 获取地区数据
			getRegionOptions() {
				uni.request({
					url: `${appConfig.API_URL}/wechat/region`, // 动态获取地区数据
					method: 'GET',
					success: (response) => {
						if (response.statusCode === 200 && response.data.status === 'success') {
							this.regionOptions = response.data.data.map((item) => {
								return {
									name: item.region,
									price: item.price
								}; // 将地区数据映射为 u-select 需要的格式
							});
						} else {
							uni.$u.toast('获取地区数据失败');
							console.error('服务端错误:', response);
						}
					},
					fail: (error) => {
						uni.$u.toast('网络请求失败');
						console.error('请求失败:', error);
					},
				});
			},
			calculate() {
				this.$refs.calcForm
					.validate()
					.then(() => {
						// 提交数据到后端
						uni.request({
							url: `${appConfig.API_URL}/wechat/compute`,
							method: 'POST',
							header: {
								'content-type': 'application/json',
							},
							data: {
								region: this.formModel.region,
								freight: this.formModel.freight,
								tons: this.formModel.tons,
							},
							success: (response) => {
								if (response.statusCode === 200) {
									this.totalFreight = response.data.data;
									uni.$u.toast('计算成功');
								} else {
									uni.$u.toast('计算失败');
									console.error('服务端错误:', response);
								}
							},
							fail: (error) => {
								uni.$u.toast('网络请求失败');
								console.error('请求失败:', error);
							},
						});
					})
					.catch((err) => {
						console.error('表单校验失败:', err);
					});
			},
			// 选择地区
			regionSelect(event) {
				this.formModel.region = event.name; // 更新地区值
				this.formModel.freight = event.price;
				this.showRegion = false; // 关闭弹框
			},
		},
	};
</script>

<style>
</style>
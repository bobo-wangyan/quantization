<template>
 <view>
	<view class="warp">
			<view class="box">
				<t-table @change="change">
					<t-tr font-size="8">
						<t-th>绩效指标</t-th>
						<t-th>绩效指标值</t-th>
					<!-- 	<t-th>普通交易指标</t-th>
						<t-th>普通交易指标值</t-th>
						<t-th>多空交易指标</t-th>
						<t-th>多空交易指标值</t-th> -->
					</t-tr>
					<t-tr font-size="8" v-for="item in tableList" :key="item.id">
						<t-td>{{ item.performance }}</t-td>
						<t-td>{{ item.performanceValue }}</t-td>
					<!-- 	<t-td>{{ item.ordinaryTrade }}</t-td>
						<t-td>{{ item.ordinaryTradeValue }}</t-td>
						<t-td>{{ item.oversoldTrade }}</t-td>
						<t-td>{{ item.oversoldTradeValue }}</t-td> -->
					</t-tr>
				</t-table>
			</view>
		</view>

	<view class="qiun-columns">
		<view class="qiun-charts" >
			<canvas canvas-id="canvasLineA" id="canvasLineA" class="charts" @touchstart="touchLineA"></canvas>
		</view>
	</view>
	
	
	<view>
		<histogram-chart
			:dataAs="histogramData2"
			canvasId="ht1"
			labelKey="label"
			valueKey="value"
			:yAxisAs="{
				formatter: {
					type: 'percent' 
				}
			}"
		/>
     </view>
	 
	<view>
		<line-chart
		 :dataAs="lineData1"
		 canvasId="line1"
		 labelKey="label"
		 valueKey="value"
		 :yAxisAs="{
		 	formatter: {
		 		type: 'number'
		 	}
		 }"
		 />
    </view>
 </view>
</template>

<script>
	import uCharts from '@/components/u-charts/u-charts.js';
	import tTable from '@/components/t-table/t-table.vue';
	import tTh from '@/components/t-table/t-th.vue';
	import tTr from '@/components/t-table/t-tr.vue';
	import tTd from '@/components/t-table/t-td.vue';
	import HistogramChart from '@/components/char/HistogramChart.vue';
	import LineChart from '@/components/char/LineChart.vue';
	var _self;
	var canvaLineA=null;
	export default {
		name: 'Index',
		components: {
					tTable,
					tTh,
					tTr,
					tTd,
					HistogramChart,
					LineChart
				},
		data() {
			return {
				cWidth:'',
				cHeight:'',
				pixelRatio:1,
				tableList: [],
				histogramData2: {},
				lineData1:{}
			}
		},
		onLoad() {
			
			_self = this;
			this.cWidth=uni.upx2px(750);
			this.cHeight=uni.upx2px(500);
		    
			
		},
		mounted() {
		this.getServerData();
		this.getQuotaData();
		 this.getYearRate();
		 this.getHoldWarehouseWorth()
		},
		methods: {
			getServerData(){
				uni.request({
					url: 'http://182.92.122.168:8081/data/accountWorthList',
					data:{
					},
					success: function(res) {
						let LineA={categories:[],series:[]};
						//这里我后台返回的是数组，所以用等于，如果您后台返回的是单条数据，需要push进去
						LineA.categories=res.data.data.categories;
						LineA.series=res.data.data.series;
						_self.showLineA("canvasLineA",LineA);
					},
					fail: () => {
						_self.tips="网络错误，小程序端请检查合法域名";
					},
				});
			},
			showLineA(canvasId,chartData){
				canvaLineA=new uCharts({
					$this:_self,
					canvasId: canvasId,
					type: 'line',
					fontSize:11,
					legend:{show:true},
					dataLabel:false,
					dataPointShape:true,
					background:'#FFFFFF',
					pixelRatio:_self.pixelRatio,
					categories: chartData.categories,
					series: chartData.series,
					animation: true,
					xAxis: {
						type:'grid',
						gridColor:'#CCCCCC',
						gridType:'dash',
						dashLength:8.,
						disabled: true
					},
					yAxis: {
						gridType:'dash',
						gridColor:'#CCCCCC',
						dashLength:8,
						splitNumber:5,
						min:10,
						max:180,
						format:(val)=>{return new Number(val)/1000+'k'}
					},
					width: _self.cWidth*_self.pixelRatio,
					height: _self.cHeight*_self.pixelRatio,
					extra: {
						line:{
							type: 'straight'
						}
					}
				});
				
			},
			touchLineA(e) {
				canvaLineA.showToolTip(e, {
					format: function (item, category) {
						return category + ' ' + item.name + ':' + item.data 
					}
				});
			},
			change(e) {
				console.log(e.detail);
					},
			edit(item) {
				uni.showToast({
					title: item.name,
					icon: 'none'
				});
			},
			getQuotaData(){
				uni.request({
					url: 'http://182.92.122.168:8081/data/quotaList',
					data:{
					},
					success: function(res) {
						_self.tableList = res.data.data
					},
					fail: () => {
						_self.tips="网络错误，小程序端请检查合法域名";
					},
				});
			},
			getYearRate(){
				uni.request({
					url: 'http://182.92.122.168:8081/data/yearRateList',
					data:{
					},
					success: function(res) {
						_self.histogramData2 = res.data.data
					},
					fail: () => {
						_self.tips="网络错误，小程序端请检查合法域名";
					},
				});
			},
			getHoldWarehouseWorth(){
				uni.request({
					url: 'http://182.92.122.168:8081/data/holdWarehouseWorthList',
					data:{
					},
					success: function(res) {
						_self.lineData1 = res.data.data						
					},
					fail: () => {
						_self.tips="网络错误，小程序端请检查合法域名";
					},
				});
			}
		}
	}
</script>

<style>
	/*样式的width和height一定要与定义的cWidth和cHeight相对应*/
	.qiun-charts {
		width: 750upx;
		height: 500upx;
		background-color: #FFFFFF;
	}
	
	.charts {
		width: 750upx;
		height: 500upx;
		background-color: #FFFFFF;
	}
</style>

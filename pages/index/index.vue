<template>
	<view>
		<view class="qiun-charts" >
			<!--#ifdef MP-ALIPAY -->
			<canvas canvas-id="canvasLineA" id="canvasLineA" class="charts" :width="cWidth*pixelRatio" :height="cHeight*pixelRatio" :style="{'width':cWidth+'px','height':cHeight+'px'}" @touchstart="touchLineA" @touchmove="moveLineA" @touchend="touchEndLineA"></canvas>
			<!--#endif-->
			<!--#ifndef MP-ALIPAY -->
			<canvas canvas-id="canvasLineA" id="canvasLineA" class="charts" @touchstart="touchLineA" @touchmove="moveLineA" @touchend="touchEndLineA"></canvas>
			<!--#endif-->
		</view>
		<!--#ifdef H5 -->
		<view class="bg-olive light">
			<view class="flex">
				<view class="flex-sub padding-sm radius">
					<view class="cu-form-group">
						<input placeholder="买入价格" v-model="price.buy" name="input"></input>
					</view>
				</view>
			</view>
			<view class="flex">
				<view class="flex-sub padding-sm radius">
					<view class="cu-form-group">
						<input placeholder="周一上午" v-model="price.MonA" name="input"></input>
					</view>
				</view>
				<view class="flex-sub padding-sm radius">
					<view class="cu-form-group">
						<input placeholder="周一下午" v-model="price.MonP" name="input"></input>
					</view>
				</view>
			</view>
			<view class="flex">
				<view class="flex-sub padding-sm radius">
					<view class="cu-form-group">
						<input placeholder="周二上午" v-model="price.TuesA" name="input"></input>
					</view>
				</view>
				<view class="flex-sub padding-sm radius">
					<view class="cu-form-group">
						<input placeholder="周二下午" v-model="price.TuesP" name="input"></input>
					</view>
				</view>
			</view>
			<view class="flex">
				<view class="flex-sub padding-sm radius">
					<view class="cu-form-group">
						<input placeholder="周三上午" v-model="price.WedA" name="input"></input>
					</view>
				</view>
				<view class="flex-sub padding-sm radius">
					<view class="cu-form-group">
						<input placeholder="周三下午" v-model="price.WedP" name="input"></input>
					</view>
				</view>
			</view>
			<view class="flex">
				<view class="flex-sub padding-sm radius">
					<view class="cu-form-group">
						<input placeholder="周四上午" v-model="price.ThurA" name="input"></input>
					</view>
				</view>
				<view class="flex-sub padding-sm radius">
					<view class="cu-form-group">
						<input placeholder="周四下午" v-model="price.ThurP" name="input"></input>
					</view>
				</view>
			</view>
			<view class="flex">
				<view class="flex-sub padding-sm radius">
					<view class="cu-form-group">
						<input placeholder="周五上午" v-model="price.FriA" name="input"></input>
					</view>
				</view>
				<view class="flex-sub padding-sm radius">
					<view class="cu-form-group">
						<input placeholder="周五下午" v-model="price.FriP" name="input"></input>
					</view>
				</view>
			</view>
			<view class="flex">
				<view class="flex-sub padding-sm radius">
					<view class="cu-form-group">
						<input placeholder="周六上午" v-model="price.SatA" name="input"></input>
					</view>
				</view>
				<view class="flex-sub padding-sm radius">
					<view class="cu-form-group">
						<input placeholder="周六下午" v-model="price.SatP" name="input"></input>
					</view>
				</view>
			</view>
			<view class="padding-sm flex flex-direction">
				<button class="cu-btn bg-green lg" @tap="changeData()">提交</button>
			</view>
			<view class="padding-sm flex flex-direction">
				<button class="cu-btn bg-red lg" @tap="deleteData()">清空缓存和表单</button>
			</view>
		</view>
		<!--#endif-->
	</view>
</template>

<script>
	import uCharts from '@/components/u-charts/u-charts.js';
	import  { isJSON } from '@/common/checker.js';
	var _self;
	var canvaLineA=null;
	var lastMoveTime=null;//最后执行移动的时间戳
	export default {
		data() {
			return {
				cWidth:'',
				cHeight:'',
				pixelRatio:1,
				textarea:'',
				Interactive:'',//交互显示的数据
				defaultDate: {
				  'categories': ['买入', '1A', '1P', '2A', '2P', '3A', '3P', '4A', '4P', '5A', '5P', '6A', '6P'],
				  'series': [{
					'name': '价格',
					'data': []
				  }]
				},
				price: {
					buy: '',
					MonA: '',
					MonP: '',
					TuesA: '',
					TuesP: '',
					WedA: '',
					WedP: '',
					ThurA: '',
					ThurP: '',
					FriA: '',
					FriP: '',
					SatA: '',
					SatP: ''
				}
			}
		},
		onLoad() {
			_self = this;
			//#ifdef MP-ALIPAY
			uni.getSystemInfo({
				success: function (res) {
					if(res.pixelRatio>1){
						//正常这里给2就行，如果pixelRatio=3性能会降低一点
						//_self.pixelRatio =res.pixelRatio;
						_self.pixelRatio =2;
					}
				}
			});
			//#endif
			this.cWidth=uni.upx2px(750);
			this.cHeight=uni.upx2px(500);
			this.getServerData();
		},
		methods: {
			getServerData(){
				let LineA={categories:[],series:[]};
				//这里我后台返回的是数组，所以用等于，如果您后台返回的是单条数据，需要push进去
				LineA.categories = this.defaultDate.categories;
				LineA.series = this.defaultDate.series;
				// 初始化数据缓存(如果有)
				uni.getStorage({
				    key: 'echartdata',
				    success: function (res) {
						if (res.data) {
							LineA.series = res.data;
							// _self.price = res.data.data
						}
				    }
				})
				uni.getStorage({
				    key: 'price',
				    success: function (res) {
						if (res.data) {
							_self.price = res.data
						}
				    }
				})
				_self.textarea = JSON.stringify(this.defaultDate);
				_self.showLineA("canvasLineA",LineA);
			},
			showLineA(canvasId,chartData){
				canvaLineA=new uCharts({
					$this:_self,
					canvasId: canvasId,
					type: 'line',
					colors:['#facc14', '#f04864', '#8543e0', '#90ed7d'],
					fontSize:11,
					padding:[5,5,0,5],
					legend:{
						show:true,
						padding:5,
						lineHeight:11,
						margin:0,
					},
					dataLabel:true,
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
						dashLength:8,
						boundaryGap:'justify'
						//disableGrid:true
					},
					yAxis: {
						gridType:'dash',
						gridColor:'#CCCCCC',
						dashLength:8,
						min:0,
						max:1000
					},
					width: _self.cWidth*_self.pixelRatio,
					height: _self.cHeight*_self.pixelRatio,
					extra: {
						line:{
							type: 'curve'
						}
					}
				});
				
			},
			touchLineA(e) {
				lastMoveTime=Date.now();
			},
			moveLineA(e){
				let currMoveTime = Date.now();
				let duration = currMoveTime - lastMoveTime;
				if (duration < Math.floor(1000 / 60)) return;//每秒60帧
				lastMoveTime = currMoveTime;
				
				let currIndex=canvaLineA.getCurrentDataIndex(e);
				if(currIndex>-1&&currIndex<canvaLineA.opts.categories.length){
					let riqi=canvaLineA.opts.categories[currIndex];
					let leibie=canvaLineA.opts.series[0].name;
					let shuju=canvaLineA.opts.series[0].data[currIndex];
					this.Interactive=leibie+':'+riqi+'-'+shuju+'元';
				}
				canvaLineA.showToolTip(e, {
					format: function (item, category) {
						return category + ' ' + item.name + ':' + item.data 
					}
				});
			},
			touchEndLineA(e){
				let currIndex=canvaLineA.getCurrentDataIndex(e);
				if(currIndex>-1&&currIndex<canvaLineA.opts.categories.length){
					let riqi=canvaLineA.opts.categories[currIndex];
					let leibie=canvaLineA.opts.series[0].name;
					let shuju=canvaLineA.opts.series[0].data[currIndex];
					this.Interactive=leibie+':'+riqi+'-'+shuju+'元';
				}
				canvaLineA.touchLegend(e);
				canvaLineA.showToolTip(e, {
					format: function (item, category) {
						return category + ' ' + item.name + ':' + item.data 
					}
				});
			},
			changeData(){
				const { buy, MonA, MonP, TuesA, TuesP, WedA, WedP, ThurA, ThurP, FriA, FriP, SatA, SatP } = this.price
				let newdata = [{
					'name': '价格',
					'data': [buy, MonA, MonP, TuesA, TuesP, WedA, WedP, ThurA, ThurP, FriA, FriP, SatA, SatP]
				}]
				// set in storage
				uni.setStorage({
					key: 'echartdata',
					data: newdata,
					success: function () {
						console.log('success');
					}
				})
				uni.setStorage({
					key: 'price',
					data: this.price,
					success: function () {
						console.log('success');
					}
				})
				canvaLineA.updateData({
					series: newdata,
				})
			},
			deleteData(){
				uni.removeStorage({
				    key: 'echartdata',
				    success: function (res) {
				        console.log('success');
				    }
				})
				uni.removeStorage({
				    key: 'price',
				    success: function (res) {
				        console.log('success');
				    }
				})
				let newdata = [{
					'name': '价格',
					'data': []
				}]
				canvaLineA.updateData({
					series: newdata,
				})
				this.price = {
					buy: '',
					MonA: '',
					MonP: '',
					TuesA: '',
					TuesP: '',
					WedA: '',
					WedP: '',
					ThurA: '',
					ThurP: '',
					FriA: '',
					FriP: '',
					SatA: '',
					SatP: ''
				}
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

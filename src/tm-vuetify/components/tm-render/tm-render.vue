<template>
	<view class="tm-render" @touchmove.stop.prevent="">
		
			<!-- #ifdef MP-WEIXIN || MP-QQ || MP-KUAISHOU  -->
			
			<canvas :style="{width:`${c_w}px`,height:`${c_h}px`}" @mouseup="touchend" @mousedown="touchstart"
				@mousemove="touchmove" @touchend="touchend" @touchmove="touchmove" @touchstart="touchstart" id="exid"
				canvas-id="exid" type="2d"></canvas>
				
			<!-- #endif -->
			
			<!-- #ifndef MP-WEIXIN || MP-QQ || MP-KUAISHOU -->
			<canvas :style="{width:`${c_w}px`,height:`${c_h}px`}" @mouseup="touchend" @mousedown="touchstart"
				@mousemove="touchmove" @touchend="touchend" @touchmove="touchmove" @touchstart="touchstart" id="exid"
				canvas-id="exid" ></canvas>
			<!-- #endif -->
	</view>
</template>

<script>
	import CRender from '@/tm-vuetify/tool/function/crender/class/crender.class'
	// import Graph from '@/tm-vuetify/tool/function/crender/class/graph.class'
	// import {CRender,GRAPHS} from '@/tm-vuetify/tool/function/render/index.js'

	let render = null;
	export default {
		name: "tm-render",
		props: {
			width: {
				type: Number,
				default: 0
			},
			height: {
				type: Number,
				default: 600
			}
		},
		computed: {
			c_w: {
				get: function() {
					return this.cavan_width;
				},
				set: function(val) {
					this.cavan_width = val;
				},

			},
			c_h: function() {
				return uni.upx2px(this.height);
			}
		},
		created() {
			const dpr = uni.getSystemInfoSync().pixelRatio
			this.dpr = dpr;
			this.c_w = uni.upx2px(this.width);
		},
		data() {
			return {
				cavan_width: 0,
				canvaConfig: null,
				dragGrpahId: '',//当前正在拖动或者点击的项目id.
				old_x: 0,
				old_y: 0,
				isDrag: false,
				
				dpr: 1
			};
		},
		mounted() {
			// let t = this;
			// setTimeout(function() {
				
			// 	uni.canvasGetImageData({
			// 		canvasId:'exid',x:0,y:0,width:300,height:300,
			// 		success(e){
			// 			console.log(e)
			// 		},
			// 		fail(e){
			// 			console.log(e)
			// 		}
			// 	},t)
			// }, 3000);
			
			// return;
			// this.diaryCalComponent = this.selectComponent('#mychart-bar');
			this.$nextTick(function() {

				this.inits();
			})
		},
		destroyed() {
			render = null;
			clearTimeout(555)
		},
		methods: {
			async inits() {


				let t = this;

				let res = await this.$Querey('.tm-render', this).catch(e => {})
				let p = res[0];
				t.c_w = p.width || 300;
				t.canvaConfig = p;

				// console.log(ctx);

				//#ifdef MP-WEIXIN || MP-QQ || MP-KUAISHOU
				uni.createSelectorQuery().in(t).select('#exid').fields({
					node: true,
					context: true,
				}, function(res) {
					let canvas = res.node;

					let ctx = canvas.getContext('2d')
					
					ctx['dpr'] = t.dpr;
					ctx['scaledpr'] = 10 / (10 * t.dpr);
					const w = ctx['width'] = t.c_w
					const h = ctx['height'] = t.c_h

					// canvas.width = res[0].width * dpr  
					// canvas.height = res[0].height * dpr  
					const dpr = uni.getSystemInfoSync().pixelRatio

					canvas.width = w * dpr
					canvas.height = h * dpr

					// 设置 canvas 坐标原点
					ctx.translate(0, 0);
					ctx.scale(dpr, dpr)
					render = new CRender(ctx, t, canvas)

					t.$nextTick(function() {
						t.$emit('render', render.area);
					})

				}).exec()

				//#endif
				

				//#ifndef MP-WEIXIN || MP-QQ || MP-KUAISHOU
				let ctx = uni.createCanvasContext('exid', t)
				
				ctx['dpr'] = t.dpr;
				ctx['scaledpr'] = 10 / (10 * t.dpr);
				const w = ctx['width'] = t.c_w
				const h = ctx['height'] = t.c_h

				render = new CRender(ctx, t)

				t.$nextTick(function() {
					t.$emit('render', render.area);
				})

				//#endif


			},
			wait(time) {
				return new Promise(resolve => setTimeout(resolve, time))
			},
			getTextWidthAndPos(shape) {
				if (!render) return [0, 0, 0, 0];
				let {
					content,
					position,
					maxWidth,
					rowGap
				} = shape.shape

				const {
					textBaseline,
					fontSize,
					textAlign
				} = shape.style


				let [x, y] = position

				content = content.split('\n')
				const rowNum = content.length
				const lineHeight = fontSize + rowGap
				const allHeight = rowNum * lineHeight - rowGap
				const twidth = render.ctx.measureText(content + "").width;
				if (textBaseline === 'middle') {

					y -= allHeight * rowNum + fontSize / 2
				}

				if (textBaseline === 'bottom') {

					y += fontSize
				}

				if (textAlign === 'center') {
					x -= twidth / 2
					y += fontSize
				}
				return [x, y, twidth, allHeight]
				// measureText
			},
			getRender() {
				return render;
			},
			async addGraph(obj) {
				let t = this;
				let pf = obj;
				if (typeof obj == 'object' && Array.isArray(obj)) {
					let c = obj.filter(el => {
						return {
							...el,
							tmid: uni.$tm.guid()
						};
					})
					pf = c;
				} else if (typeof obj == 'object' && !Array.isArray(obj)) {
					pf = [{
						...pf,
						tmid: uni.$tm.guid()
					}]
				}

				let graphs = pf.map(config => render.add(config))

				graphs.forEach((graph, i) => {
					const config = pf[i]
					t.updateGraphConfigByKey(graph, config)
				})

				await render.launchAnimation()
				//释放内存。
				// graphs = []
				return graphs.length == 1 ? graphs[0] : graphs;
			},
			//添加完毕需要更新下，才会显示。
			updateGraphConfigByKey(graph, config) {
				const keys = Object.keys(config)

				keys.forEach(async key => {
					if (key === 'shape' || key === 'style') {
						// graph.animation('shape', {x:config.shape.x+5}, true)

						await graph.animation(key, config[key], true)
					} else {
						graph[key] = config[key]
					}
				})
			},
			touchend(event) {
				let t = this;
				let evx = 0;
				let evy = 0;
				//触摸
				if (event.type.indexOf('mouse') == -1 && event.changedTouches.length == 1) {
					evx = event.changedTouches[0].x
					evy = event.changedTouches[0].y
					//电脑端。
				} else {
					evx = event.pageX - this.canvaConfig.left
					evy = event.pageY - this.canvaConfig.top;

				}

				let x = evx;
				let y = evy;


				this.dragGrpahId = "";
				this.isDrag = false
				//触发画板的事件。
				this.$emit('touchend', {
					x: x,
					y: y
				})
				//在那个元素上离开的。
				let gps = render.graphs;
				let isClickGrpahs = gps.filter((el, index) => {
					if (el.name == 'text') {
						let rect = t.getTextWidthAndPos(el);
						el.hoverRect = rect
					}

					return el.hoverCheck([x, y], el) || el.hoverCheckProcessor([x, y], el);
				});

				if (isClickGrpahs.length > 0) {
					let nowgap = isClickGrpahs[0];

					// 执行元素上绑定的事件。
					if (nowgap[event.type]) nowgap[event.type].call(nowgap, {
						x: x,
						y: y
					});
				}
			},
			touchmove(event) {
				let t = this;
				let evx = 0;
				let evy = 0;
				let isPc = false
				//触摸
				if (event.type.indexOf('mouse') == -1 && event.changedTouches.length == 1) {
					evx = event.changedTouches[0].x
					evy = event.changedTouches[0].y
					isPc = false
					//电脑端。
				} else {

					evx = event.pageX - this.canvaConfig.left
					evy = event.pageY - this.canvaConfig.top;
					isPc = true;
				}

				let movex = evx - this.old_x;
				let movey = evy - this.old_y;
				let x = evx;
				let y = evy;
				// 触发发画板的事件
				this.$emit('touchmove', {
					x: x,
					y: y
				})
				if(this.isDrag==false) return;
				//在哪个元素移动的。
				let gps = render.graphs;
				let isClickGrpahs = gps.filter((el, index) => {
					return (el.hoverCheck([x, y], el) || el.hoverCheckProcessor([x, y], el))&&el.tmid==t.dragGrpahId;
				});

				if (isClickGrpahs.length > 0) {
					
					
					let nowgap = isClickGrpahs[0];
					if (isPc) {
						movex = evx - this.old_x;
						movey = evy - this.old_y;
					}
					if ((nowgap.drag === true && this.isDrag == true) || (nowgap.drag === true && isPc == false)) {
						
						if (nowgap.name == "circle" || nowgap.name == "ellipse" ||
							nowgap.name == "ring" || nowgap.name == "arc" || nowgap.name == "regPolygon") {
							nowgap.attr('shape', {
								rx: movex,
								ry: movey
							})
						} else if (nowgap.name == "rect" ||nowgap.name ==  'rectRound'|| nowgap.name == "path"|| nowgap.name == "image"|| nowgap.name == "star" || nowgap.name =='arrow') {
							nowgap.attr('shape', {
								x: movex,
								y: movey
							})
						} else if (nowgap.name == "text") {
							nowgap.attr('shape', {
								position: [movex, movey]
							})
						}

						// 执行元素上绑定的事件。
						if (nowgap[event.type]) nowgap[event.type].call(nowgap, {
							x: movex,
							y: movey
						});
						// if(nowgap['mousemove']||nowgap['touchmove']){

						// 	if (nowgap['mousemove']) nowgap.mousemove.call(nowgap,{x:movex,y:movey})
						// 	if (nowgap['touchmove']) nowgap.touchmove.call(nowgap,{x:movex,y:movey})
						// }
					}
					//配置不允许拖出边界。
					// if(this.dragGrpahId === nowgap.tmid 
					// && movex+nowgap.shape.w<this.canvaConfig.width
					// && movey+nowgap.shape.h<this.canvaConfig.height
					// && x>=0&&y>=0&&movex>=0&&movey>=0
					// ){


					// }
					// this.$emit('shape:touchmove',{x:x,y:y,shape:nowgap})
				}

			},
			touchstart(event) {
				let t = this;
				let evx = 0;
				let evy = 0;
				let isPc = false
				//触摸
				if (event.type.indexOf('mouse') == -1 && event.changedTouches.length == 1) {
					evx = event.changedTouches[0].x
					evy = event.changedTouches[0].y
					isPc = false
					//电脑端。
				} else {
					evx = event.pageX - this.canvaConfig.left
					evy = event.pageY - this.canvaConfig.top
					isPc = true;
				}

				let x = evx
				let y = evy

				let gps = render.graphs;



				//点中了哪些图片，第一个是最顶层的，依次类推。
				let isClickGrpahs = gps.filter((el, index) => {
					// if (el.name == 'text') {
					// 	let rect = t.getTextWidthAndPos(el);
					// 	el.hoverRect = rect
						
					// }
					// 要判断谁的层级高就是先托动谁。
					return el.hoverCheck([x, y], el) || el.hoverCheckProcessor([x, y], el);
				});

				if (isClickGrpahs.length > 0) {
					var indexOfMax = 0;
					var max = isClickGrpahs.reduce( (a,c,i) => c.index > a ? (indexOfMax = i,c.index) : a, 0)
					let nowgap = isClickGrpahs[indexOfMax];
					if (nowgap.drag === true) {
						this.dragGrpahId = nowgap.tmid;
						let gapPos = [];
						if (nowgap.name == "circle" || nowgap.name == "ellipse" ||
							nowgap.name == "ring" || nowgap.name == "arc" || nowgap.name == "regPolygon"
							
						) {
							gapPos = [nowgap.shape.rx, nowgap.shape.ry]
						} else if (nowgap.name == "rect" ||nowgap.name ==  'rectRound'|| nowgap.name == "path"|| nowgap.name == "image"|| nowgap.name == "star"|| nowgap.name =='arrow') {
							gapPos = [nowgap.shape.x, nowgap.shape.y]
						} else if (nowgap.name == "text") {
							gapPos = nowgap.shape.position
						} 
						if (isPc) {
							this.old_x = evx - gapPos[0]
							this.old_y = evy - gapPos[1];
						} else {
							this.old_x = x - gapPos[0];
							this.old_y = y - gapPos[1];
						}

						this.isDrag = true
						
					}
					// 执行元素上绑定的事件。

					if (nowgap[event.type]) nowgap[event.type].call(nowgap, {
						x: x,
						y: y
					});
					// if(nowgap['mousedwon']||nowgap['touchstart']||nowgap['mouseenter']){

					// 	if (nowgap['mousedwon']) nowgap.mousedwon.call(nowgap,{x:x,y:y})
					// 	if (nowgap['touchstart']) nowgap.touchstart.call(nowgap,{x:x,y:y})
					// 	if (nowgap['mouseenter']) nowgap.mouseenter.call(nowgap,{x:x,y:y})
					// }
				} else {
					this.dragGrpahId = ""
				}

				this.$emit('touchstart', {
					x: x,
					y: y
				})
			},

		},

	}
</script>

<style lang="scss">
	body {}
</style>

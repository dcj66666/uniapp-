<template>
	<view class="tm-calendar d-inline-block">
		<view  @click.stop.prevent="openPoup"><slot></slot></view>
		<tm-poup @change="toogle" clss-style="tm-calendar-wk" :black="black_tmeme" ref="pop" height="1040" v-model="show">
			<view class="tm-calendar-wk">
				<view class="shadow-10">
					<view class="tm-calendar-title pa-32 pb-16">
						<view class="text-size-g text-align-center text-grey">{{title}}</view>
						<view class="tm-calendar-close  rounded flex-center"  :class="black_tmeme?'grey-darken-3':'grey-lighten-3'">
							<tm-icons @click="$refs.pop.close()" name="icon-times" size="24" :color="black_tmeme?'white':'grey'"></tm-icons>
						</view>
						
					</view>
					<view class="pa-32 flex-center" style="width: 96%;margin: auto;">
						<view>
							<tm-icons dense @click="preYear" name="icon-angle-double-left" color="grey-lighten-1"></tm-icons>
							
							<text class="px-16"></text>
							<tm-icons dense @click="preMonth" name="icon-angle-left" color="grey-lighten-1"></tm-icons>
							
							<!-- <text class="px-8 "></text> -->
						</view>
						<view class="text-size-n text-weight-b px-32">{{titleValue}}</view>
						<view>
							<!-- <text class="px-8"></text> -->
							
							<tm-icons dense @click="nextMonth" name="icon-angle-right" color="grey-lighten-1"></tm-icons>
							
							<text class="px-16"></text>
							
							<tm-icons dense @click="nextYear" name="icon-angle-double-right" color="grey-lighten-1"></tm-icons>
							
						</view>
					</view>

				</view>
			</view>
			
			<view class="tm-calendar-body">
				<view class="tm-calendar-bg flex-center">
					<text class="text tm-calendar-bg-center-text" :class="[black_tmeme?' opacity-5 ':'']">{{centerValue}}</text>
				</view>
				<view class="tm-calendar-content ">
					<view style="height: 32upx;"></view>
					<tm-row>
						<!-- #ifdef MP-->
						<tm-col color="" align="middle" width="14.28%" v-for="(item,index) in ['一','二','三','四','五','六','日']" 
						:key="index"><text class="text-size-s py-16">{{item}}</text>
						</tm-col>
						<!-- #endif -->
						
						<!-- #ifdef H5  || APP-VUE || APP-PLUS -->
						<tm-col color="" align="middle" width="14.28%" v-for="(item,index) in ['一','二','三','四','五','六','日']" 
						:key="index+'_acv'"><text class="text-size-s py-16">{{item}}</text>
						</tm-col>
						<!-- #endif -->
						<tm-col :round="(item.start||item.end||item.checked)?4:0" @click="day_danxuanclick(item,index)"  
						:color="item.beginEnd?(item.checked===true||item.start||item.end?color_tmeme+(black_tmeme?' bk ':''):(item.guocheng?color_tmeme+' text  opacity-7 '+(black_tmeme?'bk':''):'')):''" 
						:custom-class="isSelectedDateClass(item)"
						align="middle" width="14.28%" v-for="(item,index) in nowData"
						:key="index">
						
							<view  class="tm-calendar-col flex-center flex-col py-4" :class="[black_tmeme&&!item.beginEnd?' opacity-2 ':'']">
								
								<text class="text-size-n "
								:class="[
									!item.nowMonth&&!item.guocheng&&!item.checked&&!item.start&&!item.end?(black_tmeme?'text-grey-darken-3':'text-grey-lighten-1'):'',
									item.checked||item.start||item.end?'text-white':'',
									item.guocheng?' text text-'+color_tmeme:'',
									!item.beginEnd?'text-grey-lighten-3':''
								]"
								>{{item.day}}</text>
								<!-- 
								 !item.nowMonth&&!item.guocheng&&!item.checked&&!item.start&&!item.end?(black?'text-grey-darken-3':'text-grey-lighten-1'):'',
								 item.checked||item.start||item.end?'text-white':(item.guocheng?'':'text-grey-lighten-1'),
								 item.guocheng?'text-'+color:'',
								 !item.beginEnd?'text-grey-lighten-3':'',
								 -->
								<view class="text-size-xs  text_bl"
								>
								<block v-if="item.start">
									开始
								</block>
								<block v-if="item.end">
									结束
								</block>
								
								<block v-if="!item.start&&!item.end">
									{{item.text?item.text:(showNongli?item.nongli.day:'')}}
								</block>
								</view>
							</view>
						</tm-col>
					</tm-row>
				</view>
				<view class="" style="height: 60rpx;"></view>
				<view class="text-align-center px-32   text-grey ">{{selectedVal}}</view>
				<view class="pa-32">
					<tm-button :black="black_tmeme"  @click="confirm" block itemeClass="round-24" :theme="btnColor?btnColor:color_tmeme" fontSize="32">{{btnText}}</tm-button>
				</view>
			</view>
		</tm-poup>
	</view>
</template>

<script>
	/**
	 * 日历
	 * @description 日历组件，提供节气、农历公历显示，时间范围选择等功能。
	 * @property {String} mode = [day|rang] 默认：day单选日期，rang范围选择日期。 
	 * @property {String} title = [] 弹层层标题
	 * @property {String} btn-text = [] 底部按钮确认的文字
	 * @property {String} btn-color = [primary|green|orange|red|blue|bg-gradient-blue-lighten] 默认：bg-gradient-blue-lighten底部按钮确认的背景颜色仅支持主题色名称
	 * @property {String} color = [primary|green|orange|red|blue] 主题默认:primary，提供是请写主题色名称
	 * @property {String} default-value = [] 默认时间默认:当前时间，格式：'2021-7-21'
	 * @property {String} bing-end = [] 时间格式：'2021-7-21',范围结束时间当mode=rang时有效
	 * @property {String} bing-start = [] 时间格式：'2021-7-21',范围结束时间当mode=rang时有效
	 * @property {Boolean|String} disabled = [true|false] 是否禁用，只读，默认false
	 * @property {Function} confirm 按钮点击确认时会发送当前选中的数据。如果数据为空，则不触发。
	 * @property {Function} rang-start 日历为rang时间范围模式时，当设定开始时间触发，并返回开始时间和结束时间(空)的数据。
	 * @property {Function} rang-end 日历为rang时间范围模式时，当设定结束时间触发，并返回开始时间和结束时间的数据。
	 * @property {String} time-start = [] 日历时间可选范围开始日期格式'2021-7-21'
	 * @property {String} time-end = [] 日历时间可选范围结束日期格式'2021-7-21'
	 * @property {Array<object>} txt = [] 设置日期下标文本；格式[{date:'2021-7-21',text:"测试"}]
	 * @property {Array<object>} selected-date-class = [] 设置指定日期的样式，格式[{date:'2021-7-21',class:"类名"}]
	 * @property {Boolean|String} black = [true|false] 是否开启暗黑模式
	 * @property {Boolean} show-nongli = [true|false] 是否显示农历
	 * @property {Array} formart = [] 默认['年','月','日']，时间的分割符。
	 * @example <tm-calendar :txt="bbc" ref='calendar' mode="rang" time-start="2021-7-1" time-end="2021-7-31" @confirm="next" v-model="tts" ></tm-calendar>
	 */
	import tmIcons from "@/tm-vuetify/components/tm-icons/tm-icons.vue"
	import tmCol from "@/tm-vuetify/components/tm-col/tm-col.vue"
	import tmRow from "@/tm-vuetify/components/tm-row/tm-row.vue"
	import tmButton from "@/tm-vuetify/components/tm-button/tm-button.vue"
	import tmPoup from "@/tm-vuetify/components/tm-poup/tm-poup.vue"
	export default {
		components:{tmIcons,tmCol,tmRow,tmButton,tmPoup},
		name: "tm-calendar",
		props:{
			black:{
				type:Boolean|String,
				default:null
			},
			disabled:{
				type:Boolean|String,
				default:false
			},
			// 默认时间。
			defaultValue:{
				type:String,
				default:''
			},
			mode:{
				type:String,
				default:'day'
			},
			bingStart:{
				type:String,
				default:null
			},
			timeStart:{
				type:String,
				default:null
			},
			timeEnd:{
				type:String,
				default:null
			},
			bingEnd:{
				type:String,
				default:null
			},
			// 顶部标题。
			title:{
				type:String,
				default:'选择时间' 
			},
			// 底部按钮文件
			btnText:{
				type:String,
				default:'确认' 
			},
			// 底部按钮背景主题色名称
			btnColor:{
				type:String,
				default:'' 
			},
			// 主题色。
			color:{
				type:String,
				default:'primary' 
			},
			
			value:{
				type:Boolean,
				default:false 
			},
			
			txt:{
				type:Array,
				default:()=>{ return []}
			},
			// 指定日期的样式类格式同txt
			selectedDateClass:{
				type:Array,
				default:()=>{ return []}
			},
			// 是否显示家历
			showNongli:{
				type:Boolean,
				default:false
			},
			// 跟随主题色的改变而改变。
			fllowTheme:{
				type:Boolean|String,
				default:true
			},
			formart:{
				type:Array,
				default:()=>{
					return ['年','月','日'];
				}
			}
		},
		model:{
			prop:'value',
			event:'input'
		},
		watch:{
			bingStart:function(){
				this.watchRangeTime();
			},
			bingEnd:function(){
				this.watchRangeTime();
			},
			value:function(val){
				this.show = val;
			},

			txt:function(val){
				if(this.cal){
					this.cal.setTimeArrayText(this.txt);
				}
			},
			defaultValue:function(val,oldVal){
				if(!this.cal) return;
				// 自动更新默认的显示时间。
				if(this.mode=='day'){
					let d = new Date().toLocaleDateString().replace(/\//g,'-');
					if(this.defaultValue){
						d = this.defaultValue;
					}
					if(!oldVal||oldVal==''){
						this.cal = new this.$tm.calendar({value:d,start:this.timeStart,end:this.timeEnd})
						this.nowData = this.cal.getData();
					}
					this.cal.setValue(d)
					let index = this.findItemToindexByDate(this.cal.now_day_month)
					if(index>-1){
						this.nowData = this.cal.getData();
						let item = this.nowData[index];
						item.checked = true;
						this.selectedDay = item;
						this.fanxuanDate();
					}
				}
			}
		},
		data() {
			return {
				show: false,
				nowData:[],//当前月份数据。
				cal:null,//日历对象数据。
				selectedDay:null,
				start:null,
				end:null,
				fanwei:[]
			};
		},
		computed:{
			black_tmeme: function() {
				if (this.black !== null) return this.black;
				return this.$tm.vx.state().tmVuetify.black;
			},
			color_tmeme:function(){
				if(this.$tm.vx.state().tmVuetify.color!==null&&this.$tm.vx.state().tmVuetify.color && this.fllowTheme){
					return this.$tm.vx.state().tmVuetify.color;
				}
				return this.color;
			},
			titleValue:function(){
				if(! this.cal) return ''

				return this.cal.value.getFullYear()+(this.formart[0]??'年')+(this.cal.value.getMonth()+1)+(this.formart[1]??'月');
			},
			centerValue:function(){
				if(! this.cal) return ''
				return this.cal.value.getMonth()+1;
			},
			selectedVal:function(){
				if(this.mode=='day'&&this.selectedDay){
	
					return this.selectedDay.year+(this.formart[0]??'年')+this.selectedDay.month+(this.formart[1]??'月')+this.selectedDay.day+(this.formart[2]??'');
				}
				if(this.mode=='rang'&&this.start&&!this.end){
					let p = this.start.year+(this.formart[0]??'年')+this.start.month+(this.formart[1]??'月')+this.start.day+(this.formart[2]??'');
					let et = '' ;
					return p + " ~ " + et;
				}
				if(this.mode=='rang'&&this.start&&this.end){
					let p = this.start.year+(this.formart[0]??'年')+this.start.month+(this.formart[1]??'月')+this.start.day+(this.formart[2]??'日');
					let p2 = this.end.year+(this.formart[0]??'年')+this.end.month+(this.formart[1]??'月')+this.end.day+(this.formart[2]??'日');
				
					
					return p + " ~ " +p2;
				}
			}
			
		},
		mounted() {
			this.watchRangeTime();
			
			this.show = this.value;
		},
		methods: {
			watchRangeTime(){
				let d = new Date().toLocaleDateString().replace(/\//g,'-');
				if(this.defaultValue){
					d = this.defaultValue;
				}
				this.cal = new this.$tm.calendar({value:d,start:this.timeStart,end:this.timeEnd})
				if(this.txt){
					this.cal.setTimeArrayText(this.txt);
				}
				this.nowData = this.cal.getData();
				if(this.mode=='day'){
					let index = this.findItemToindexByDate(this.cal.now_day_month)
					if(index>-1){
						let item = this.nowData[index];
						item.checked = true;
						this.selectedDay = item;
						this.fanxuanDate();
					}
					// 反选选区。
				}else if(this.mode=='rang'){
					if(this.bingStart&&this.bingEnd){
						this.fanxuanxuanwuBydate(
						new Date(this.bingStart.replace(/-/g,'/')),
						new Date(this.bingEnd.replace(/-/g,'/')),
						)
					}
				}
			},
			openPoup(){
				if(this.disabled==true) return;
				this.show=!this.show
			},
			isGuochengBg(item){
				
				if(!item.beginEnd) return '';
				if(item.checked===true||item.start||item.end) return this.color_tmeme;
				if(item.guocheng) return this.color_tmeme + ' text' ;
				if(item.guocheng) return '';
				
			},
			isSelectedDateClass(date){
				let rangeL = 'round-l-4 round-tr-0 round-br-0'
				let rangeR = 'round-r-4 round-tl-0 round-bl-0'
				let index = this.selectedDateClass.findIndex((item)=>{
					let val = date.year+'-'+date.month+'-'+date.day;
					return val == item.date;
				})
				let pc = '';
				
				if(date.start&&!date.end){
					pc = ` ${rangeL} `;
				}else if(date.end&&!date.start){
					pc = ` ${rangeR} `;
				}else if(date.start&&date.end){
					pc = ` `
				}
				
				if(index >-1){
					
					return this.selectedDateClass[index].class + pc
				}
				
				return pc;
			},
			close(){
				this.$refs.pop.close();
			},
			toogle(e){
				this.$emit('input',e);
				this.$emit('update:value',e);
			},
			// 获取当前月份的数据。
			getNowMonthData(){
				return this.cal.getData();
			},
			// 设置时间文本数据。
			setNowMonthData(e){
				if(!Array.isArray(e)) return;
				this.cal.setTimeArrayText(e);
				this.nowData = this.cal.getData();
				let index = this.findItemToindexByDate(this.cal.now_day_month)
				if(index>-1){
					let item = this.nowData[index];
					item.checked = true;
					this.selectedDay = item;
					
				}
				this.fanxuanDate()
			},
			confirm(){
				if(this.mode=='day'){
					if(this.selectedDay){
						this.$emit('confirm',this.selectedDay);
					}
				}else if(this.mode=='rang'){
					if(this.start&&this.end){
						this.$emit('confirm',this.fanwei);
					}
				}
				
				this.close();
				
				
			},
			preMonth(){
				if(!this.cal) return;
				this.nowData = this.cal.prevMonth().getData();
				this.$nextTick(function(){
					this.fanxuanDate();
				})
			},
			nextMonth(){
				if(!this.cal) return;
				this.nowData = this.cal.nextMonth().getData();
				this.$nextTick(function(){
					this.fanxuanDate();
				})
			},
			preYear(){
				if(!this.cal) return;
				this.nowData = this.cal.prevYear().getData();
				this.$nextTick(function(){
					this.fanxuanDate();
				})
			},
			nextYear(){
				if(!this.cal) return;
				this.nowData = this.cal.nexYear().getData();
				this.$nextTick(function(){
					this.fanxuanDate();
				})
			},

			fanxuanDate(){
				if(this.mode=='day'){
					if(this.selectedDay){
						let index = this.findItemToindex(this.selectedDay);
						if(index>-1){
							this.nowData.splice(index,1,this.selectedDay);
						}
					}
				}else if(this.mode=='rang'){
					
					if(this.start&&!this.end){
						let index = this.findItemToindex(this.start);
						if(index>-1){
							this.nowData.splice(index,1,this.start);
						}
						return;
					}
					if(this.start&&this.end){
						let index = this.findItemToindex(this.start);
						if(index>-1){
							this.nowData.splice(index,1,this.start);
						}
						index = this.findItemToindex(this.end);
						if(index>-1){
							this.nowData.splice(index,1,this.end);
						}
						
					
						for(let i=0;i<this.fanwei.length;i++){
							let idx = this.fanwei[i];
							index = this.findItemToindex(idx);
							if(index>-1){
								this.nowData.splice(index,1,idx);
							}
						}
						
						
						return;
					}
				}
			},
			isSelected:function(item){
				let index = this.selectedDay.indexOf(idx=>{
					return idx.year===item.year&&idx.month===item.month&&idx.day===item.day;
				})
				if(index===-1) return false;
				return true;
			},
			// 查找选中的
			findChecked(){
				let xz = [];
				this.nowData.forEach((item,index)=>{
					if(item.checked===true){
						xz.push({
							item:item,
							index:index
						})
					}
				})
				
				return xz;
			},
			// 查找index
			findItemToindex(item){
				let index=-1;
				for(let i=0;i<this.nowData.length;i++){
					let idx = this.nowData[i];
					if(idx.year==item.year&&idx.month==item.month&&idx.day==item.day){
						index=i;
						break;
					}
				}
				return index;
			},
			// 根据时间选取查找。
			findItemToindexByDate(date){
				let index=-1;
				for(let i=0;i<this.nowData.length;i++){
					let idx = this.nowData[i];
					if(idx.year==date.getFullYear()&&idx.month==(date.getMonth()+1)&&idx.day==date.getDate()){
						index=i;
						break;
					}
				}
				return index;
			},
			// 清除当前日历中选中的部分。
			clearCheckedNowDay(){
				let xz = [];
				this.nowData.forEach((item,index)=>{
					item.checked = false;
					xz.push(item);
				})
				this.nowData = xz;
			},
			// 清除选区的内容。
			clearRangeNowDay(){
				let xz = [];
				let dqdata = this.cal.value;
				for(let i=0;i<this.nowData.length;i++){
					let item = this.nowData[i];
					let index = i;
					item.start = false;
					item.end = false;
					item.guocheng = false;
					item.checked = false;
					
					if(item.year==dqdata.getFullYear()&&item.month==dqdata.getMonth()+1){
						
						item.nowMonth =true;
					}else{
						item.nowMonth =false;
					}
					xz.push(item);
				}
				
				
				this.nowData = xz;
			},
			// 通过外围 时间默认的选中
			fanxuanxuanwuBydate(start,end){
				if(!start||!end) return;
				
				this.$nextTick(function(){
					if(start.getTime()>end.getTime()) return;
					// 获取开始月份的数据。
					let sobj = new this.$tm.calendar({value:start.toLocaleDateString().replace(/\//g,'-')});
					// 获取结束月份的数据。
					let eobj = new this.$tm.calendar({value:end.toLocaleDateString().replace(/\//g,'-')});
					function findItemToindex_only(obj,type){
						let item=null;
						for(let i=0;i<obj.length;i++){
							let idx = obj[i];
							if(idx.nowDay==true){
								item = idx;
								break;
							}
						}
						if(type == 'start'&&item){
							item.start=true;
							item.end=false;
							item.checked=false;
							item.guocheng=false;
						}else if(type == 'end'&&item){
							item.start=false;
							item.end=true;
							item.checked=false;
							item.guocheng=false;
						}
						
						return item;
					}
					let sodata = sobj.getData();
					let eodata = eobj.getData();
					
					let start_obj = null;
					let end_obj = null;
					start_obj = findItemToindex_only(sodata,'start')
					let s_index = this.findItemToindex(start_obj)
					if(s_index>-1){
						this.nowData.splice(s_index,1,start_obj)
					}
					end_obj = findItemToindex_only(eodata,'end')
					let e_index = this.findItemToindex(end_obj)
					if(e_index>-1){
						this.nowData.splice(e_index,1,end_obj)
					}
					this.start = start_obj
					this.end = end_obj
					// 如果结束和开始相等。
					if(start.getTime()==end.getTime()){
						this.start = {...this.start,start:true,end:true}
						this.end = {...this.end,start:true,end:true}
					}
					// 计算包含的时间 区域。
					let start_time = new Date(this.start.year+"/"+this.start.month+"/"+this.start.day).getTime()
					let start_bdm = new Date(this.start.year,this.start.month-1).getTime()
					let end_time = new Date(this.end.year+"/"+this.end.month+"/"+this.end.day).getTime()
					let end_bdm = new Date(this.end.year,this.end.month-1).getTime()
					this.fanwei=[];
					let m=[];
					let testc = new this.$tm.calendar({value:this.start.year+"/"+this.start.month+"/"+this.start.day})
					testc.setTimeArrayText(this.txt);
					function findItemToindex_only_nob(item,obj){
						let istrue = false;
						for(let i=0;i<obj.length;i++){
							let idx = obj[i];
							if(item.year==idx.year&&item.month==idx.month&&item.day==idx.day){
								istrue = true;
								break;
							}
						}
						return istrue;
					}
					
					
					for(let j=0;j<1000;j++){
						let pds  = new Date(testc.value.getFullYear(),testc.value.getMonth()).getTime();
						let testod = testc.getData();
						if(pds<=end_bdm&&pds>=start_bdm){
							
							for(let k=0;k<testod.length;k++){
								if(!findItemToindex_only_nob(testod[k],m)){
									m.push(testod[k]);
								}
							}
							testc.nextMonth()
							
						}else{
							break;
						}
					}
					
					
					for(let i=0;i<m.length;i++){
						let dod = {...m[i]};
						let npds = new Date(dod.year+"/"+(dod.month)+"/"+dod.day);
						let dq = npds.getTime()
						if(dq > start_time && dq < end_time){
							dod.start=false;
							dod.end=false;
							dod.checked=false;
							dod.guocheng=true;
							
							let isindex =this.findItemToindex(dod);
							if(isindex>-1){
								this.nowData.splice(isindex,1,dod);
							}
							this.fanwei.push(dod)
						}
					}
					
					
					// 开始反选。
					this.fanxuanDate();
				})
				
				
			},
			day_danxuanclick(e,index) {
				// 是否禁用。
				if(this.disabled||this.disabled=='true'){
					this.$tm.toast("已被禁用")
					return;
				}
				if(!e.beginEnd){
					this.$tm.toast("不可选！")
					return;
				};
				if(this.mode=='day'){
					this.clearCheckedNowDay();
					let p = {...e};
					p.checked = true;
					this.selectedDay = p;
					this.nowData.splice(index,1,p);
				}else{

					let p = {...e};
					if(!this.start&&!this.end){
						this.clearRangeNowDay();
						
						p.start = true;
						p.end = false;
						p.guocheng = false;
						this.start = p
						this.nowData.splice(index,1,p);
						//发布选中开始的事件。
						this.$emit("rang-start",{start:p,end:null})
						return;
					}
					if(this.start&&this.end){
						this.clearRangeNowDay();
						
						p.start = true;
						p.end = false;
						p.guocheng = false;
						this.start = p
						this.nowData.splice(index,1,p);
						this.end=null;
						//发布选中开始的事件。
						this.$emit("rang-start",{start:p,end:null})
						return;
					}
					if(this.start&&!this.end){
						
						this.$nextTick(function(){
							let selected = new Date(e.year,e.month,e.day).getTime();
							let selectedStart = new Date(this.start.year,this.start.month,this.start.day).getTime()
							if(selected <= selectedStart)
							{
								
								// this.clearRangeNowDay();
								let enjh = uni.$tm.deepClone(this.start);
								enjh.start = selected<selectedStart?false:true;
								enjh.end = true;
								enjh.guocheng = false;
								this.end = enjh
								let index_check =-1;
								for(let ix =0 ; ix <this.nowData.length;ix++){
									let item_check = this.nowData[ix]
									if( item_check.month==enjh.month&&item_check.year==enjh.year&&item_check.day==enjh.day){
										index_check=ix;
										break;
									}
								}
								
								
								if(index_check>-1){
									this.nowData.splice(index_check,1,this.end);
								}
								p.start = true;
								p.end =  selected<selectedStart?false:true;
								p.guocheng = false;
								this.start = p
								
							}else if(selected > selectedStart)
							{
								p.start = false;
								p.end = true;
								p.guocheng = false;
								this.end = p
							
							}
							
							this.nowData.splice(index,1,p);
							
							//发布选中开始的事件。
							this.$emit("rang-start",{start:p,end:this.end})
							//发布选中开始的事件。
							this.$emit("rang-end",{start:this.start,end:this.end})
							
							// 计算包含的时间 区域。
							let start_time = new Date(this.start.year+"/"+this.start.month+"/"+this.start.day).getTime()
							let start_bdm = new Date(this.start.year,this.start.month-1).getTime()
							let end_time = new Date(this.end.year+"/"+this.end.month+"/"+this.end.day).getTime()
							let end_bdm = new Date(this.end.year,this.end.month-1).getTime()
							this.fanwei=[];
							let m=[];
							
							
							let testc = new this.$tm.calendar({value:this.start.year+"-"+this.start.month+"-"+this.start.day})
							
							
							testc.setTimeArrayText(this.txt);
							function findItemToindex_only(item,obj){
								let istrue = false;
								for(let i=0;i<obj.length;i++){
									let idx = obj[i];
									if(item.year==idx.year&&item.month==idx.month&&item.day==idx.day){
										istrue = true;
										break;
									}
								}
								return istrue;
							}
							
							
							for(let j=0;j<1000;j++){
								let npsDate =  new Date(testc.value.getFullYear(),testc.value.getMonth());
								let pds  = npsDate.getTime();
								let testod = testc.getData();
								
								if(pds<=end_bdm&&pds>=start_bdm){
									
									for(let k=0;k<testod.length;k++){
										if(!findItemToindex_only(testod[k],m)){
											m.push(testod[k]);
										}
									}
									testc.nextMonth()
									
								}else{
									break;
								}
							}
							
							for(let i=0;i<m.length;i++){
								let dod = {...m[i]};
								let npds = new Date(dod.year+"/"+(dod.month)+"/"+dod.day);
								let dq = npds.getTime()
								if(dq > start_time && dq < end_time){
									dod.start=false;
									dod.end=false;
									dod.checked=false;
									dod.guocheng=true;
								
									let isindex =this.findItemToindex(dod);
									if(isindex>-1){
										this.nowData.splice(isindex,1,dod);
										
									}
									this.fanwei.push(dod)
								}
							}
							
															
						})
						
						
						
						
					}
					
					
					
				}
				
				
			}
		},
	}
</script>

<style lang="scss" scoped>
	.tm-calendar-col{
		width: 100%;
		// height: 80upx;
		min-height: 80upx;
		
		// text-align: center;
		// line-height: 80upx;
		line-height: inherit;
		position: relative;
		.text_bl{
			// position: absolute;
			bottom: 14upx;
		}
	}
	.textOn{
		color:#1976d2 !important;
	}
	.tm-calendar-wk {
		width: 100%;
		.tm-calendar-title {
			position: relative;

			.tm-calendar-close {
				position: absolute;
				top: 32upx;
				right: 32upx;
				height: 50upx;
				width: 50upx;
			}
		}

		
	}

	.tm-calendar-body {
		position: relative;
	
		.tm-calendar-bg {
			height: 570upx;
			.text {
				font-size: 400upx !important;
				color: rgba(225, 225, 225, 0.4);
				&.tm-calendar-bg-center-text span{
					font-size: 400upx !important;
				}
			}
		}
	
		.tm-calendar-content {
			width: 100%;
			position: absolute;
			top: 0;
			left: 0;
		}
	}
	
	
</style>

<template>
	<div class="date-picker form-inline form-control"
	     :class="{'isdisabled' : isdisabled == 'true'}" 
	     v-on-clickaway="close">
		<div class="form-horizontal has-feedback">
			<input class="hasclear"
				   :class="{'isdisabled' : isdisabled == 'true'}" 
                   v-bind:value="value | displayFormatter(selectedDate, displaylang, valueformat, displayformat, dateType)" 
                   @click="toggleState"
                   @mouseenter="showCancel = true" 
                   @mouseleave="showCancel = false" readonly>
            <span id="searchclear" v-show="panelState && value != null && value != ''" @click="clear" class="fa fa-times"></span>
		</div>
		
		 <transition name="toggle">
			<div v-show="panelState" style="z-index:9; position: absolute; top: 27px;">
				<!-- 一般模式 -->
				<div class="panel panel-success" 
				     v-show="tmpType == 'date' || tmpType == 'datetime'">
					<div class="panel-heading form-inline">
						<button type="button"
						        class="btn btn-default col-md-push-8 btn-options" 
						        @click="lastMonth()">
							<i class="fa fa-chevron-left"></i>
						</button>
						<div class="form-group">
							<div class="text-center">
								<div class="btn"> 
									<span @click="tmpType = 'year'"> {{ displaylang === 'zh' ? tmpYear - 1911 : tmpYear }} </span>
								</div> / 
								<div class="btn"> 
									<span @click="tmpType = 'month'"> {{ language === 'zh' ? tmpMonth : month(tmpMonth, language) }} </span>
								</div>
							</div>
						</div>
						<button type="button" 
						        class="btn btn-default pull-right btn-options" 
						        @click="nextMonth()">
							<i class="fa fa-chevron-right"></i>
						</button>
						<button type="button" 
						        class="btn btn-info pull-right btn-options"
						        v-show="type == 'datetime'"
						        id="changeDateType-time"
						        @click="tmpType = 'time'">
							<i class="fa fa-clock-o "></i>
						</button>
						
						<tooltip text="選擇時間" target="#changeDateType-time"/>
					</div>
					<div class="panel-body">
						<table>
							  <colgroup>
								<col span="7" style="width: 14%">
							  </colgroup>
							<tr>
								<td class="text-center" v-for="item in weekList" :key="item">{{week(item, language)}}</td>
							</tr>
							<tr v-for="n in (dateList.length / 7)">
								<td class="nav-item" v-for="(item, index) in getDateList(n)" :key="index">
									<div>
										<button type='button' 
												:class="{'picker-button': true, 'btn btn-default': true, preMonth: item.previousMonth && !item.isSelected, nextMonth: item.nextMonth && !item.isSelected, 'btn-primary': item.isSelected, 'btn-success': item.isToday && !item.isSelected, 'btn-default': item.isMonthday && !item.isSelected, 'alert-danger': item.isHoliday && !item.isSelected}"
												@click="selectDate(item);"
												v-text="item.value">
										</button>
									</div>
								</td>
							</tr>
						</table>
					</div>
				</div>
				<!-- 年份模式 -->
				<div class="panel panel-success" v-show="tmpType === 'year'">
					<div class="panel-heading form-inline">
						<button type="button" 
						        class="btn btn-default col-md-push-8 btn-options" 
						        @click="tmpYear -= 12">
							<i class="fa fa-chevron-left"></i>
						</button>
						<div class="form-group text-center">
							<div class="btn"> 
								<span> {{ (tmpYear - (tmpYear % 12)) - (displaylang == 'zh' ? 1911 : 0) }} ~
								       {{ (tmpYear - (tmpYear % 12) + 11) - (displaylang == 'zh' ? 1911 : 0) }} </span>
							</div>
						</div>
						<button type="button" 
						        class="btn btn-default pull-right btn-options" 
						        @click="tmpYear += 12">
							<i class="fa fa-chevron-right"></i>
						</button>
					</div>
					<div class="panel-body">
						<table class="table col-md-12">
							<colgroup>
								<col span="4" style="width: 25%">
							</colgroup>
							<tr v-for="n in 3">
								<td class="nav-item" v-for="(item, index) in getYearList(n)" :key="index">
									<button type='button' class="btn btn-default picker-button-1"
									      v-text="item.value - (displaylang == 'zh' ? 1911 : 0)"
										  @click="selectYear(item)"
										  ></button>
								</td>
							</tr>
						</table>
					</div>
				</div>
				<!-- 月份模式 -->
				<div class="panel panel-success" v-show="tmpType === 'month'">
					<div class="panel-heading form-inline">
						<button type="button" 
						        class="btn btn-default col-md-push-8 btn-options" 
						        @click="tmpYear = tmpYear - 1">
							<i class="fa fa-chevron-left"></i>
						</button>
						<div class="text-center form-group">
							<div class="btn"> 
								<span @click="tmpType = 'year'"> {{ displaylang === 'zh' ? ((tmpYear) - 1911) : tmpYear }} </span>
							</div>
						</div>
						<button type="button"
						        class="btn btn-default pull-right btn-options" 
						        @click="tmpYear = tmpYear + 1">
							<i class="fa fa-chevron-right"></i>
						</button>
					</div>
					<div class="panel-body">
						<table class="table col-md-12">
							<colgroup>
								<col span="4" style="width: 25%">
							</colgroup>
							<tr v-for="n in 3">
								<td class="nav-item" v-for="(item, index) in getMonthList(n)" :key="index">
									<button type='button' 
									      class="btn btn-default picker-button-1"
										  @click="selectMonth(item)"
										  >{{month(item.value, language)}}</button>
								</td>
							</tr>
						</table>
					</div>
				</div>
				<!-- 時間模式 -->
				<div class="panel panel-success" v-show="tmpType === 'time'" style="width: 200px">
					<div class="panel-heading form-inline">
						<button type="button" 
						        class="btn btn-info btn-options" 
						        v-show="type == 'datetime'"
						        id="changeDateType-date1"
						        @click="tmpType = 'datetime'">
							<i class="fa fa-calendar"></i>
						</button>
						
						<tooltip text="選擇日期" target="#changeDateType-date1"/>
					</div>
					<div class="panel-body">
						<table style="width: 100%">
							<colgroup>
								<col span="4" style="width: 25%">
							</colgroup>
							<tr>
								<td></td>
								<td class="text-center">
									<button type="button"
									        class="btn btn-default btn-options"
									        @click="selectedTime('hour', 1)">
										<i class="fa fa-plus"></i>
									</button>
								</td>
								<td class="text-center">
									<button type="button" 
									        class="btn btn-default btn-options"
									        @click="selectedTime('min', 1)">
										<i class="fa fa-plus"></i>
									</button>
								</td>
								<td></td>
							</tr>
							<tr>
								<td></td>
								<td class="text-center">
									<button type="button" 
											@click="tmpType = 'hour'"
									        class="btn btn-default btn-options" >
										{{selectedDate ? ((selectedDate.getHours() < 10 ? '0' : '') + selectedDate.getHours()) : '00'}}
									</button>
								</td>
								<td class="text-center">
									<button type="button" 
											@click="tmpType = 'min'"
									        class="btn btn-default btn-options" >
										{{selectedDate ? ((selectedDate.getMinutes() < 10 ? '0' : '') + selectedDate.getMinutes()) : '00'}}
									</button>
								</td>
								<td></td>
							</tr>
							<tr>
								<td></td>
								<td class="text-center">
									<button type="button" 
									        class="btn btn-default btn-options"
									        @click="selectedTime('hour', -1)">
										<i class="fa fa-minus"></i>
									</button>
								</td>
								<td class="text-center">
									<button type="button" 
									        class="btn btn-default btn-options"
									        @click="selectedTime('min', -1)">
										<i class="fa fa-minus"></i>
									</button>
								</td>
								<td></td>
							</tr>
						</table>
					</div>
				</div>
				<!-- 小時選擇模式 -->
				<div class="panel panel-success" v-show="tmpType === 'hour'">
					<div class="panel-heading form-inline">
						<button type="button" 
						        class="btn btn-info btn-options" 
						        v-show="type == 'datetime'"
						        id="changeDateType-date2"
						        @click="tmpType = 'datetime'">
							<i class="fa fa-calendar"></i>
						</button>
						<tooltip text="選擇日期" target="#changeDateType-date2"/>
						<button type="button" 
						        class="btn btn-info btn-options" 
						        id="changeDateType-time2"
						        @click="tmpType = 'time'">
							<i class="fa fa-clock-o "></i>
						</button>
						<tooltip text="選擇時間" target="#changeDateType-time2"/>
					</div>
					
					<div class="panel-body">
						<table class="table col-md-12">
							<colgroup>
								<col span="6">
							</colgroup>
							<tr v-for="n in 4">
								<td class="nav-item" v-for="(item, index) in getHourList(n)" :key="index">
									<button type='button' 
											:class="{'btn-primary': selectedDate && item.value == selectedDate.getHours()}"
									      	class="btn btn-default picker-button"
							        		@click="selectedDetailTime('hour', item.value)"
										  >{{item.value}}</button>
								</td>
							</tr>
						</table>
					</div>
				</div>
				<!-- 分鐘選擇模式 -->
				<div class="panel panel-success" v-show="tmpType === 'min'">
					<div class="panel-heading form-inline">
						<button type="button" 
						        class="btn btn-info btn-options"
						        v-show="type == 'datetime'" 
						        id="changeDateType-date3"
						        @click="tmpType = 'datetime'">
							<i class="fa fa-calendar"></i>
						</button>
						<tooltip text="選擇日期" target="#changeDateType-date3"/>
						<button type="button" 
						        class="btn btn-info btn-options" 
						        id="changeDateType-time3"
						        @click="tmpType = 'time'">
							<i class="fa fa-clock-o "></i>
						</button>
						<tooltip text="選擇時間" target="#changeDateType-time3"/>
					</div>
					
					<div class="panel-body">
						<table class="table col-md-12">
							<colgroup>
								<col span="10">
							</colgroup>
							<tr v-for="n in 6">
								<td class="nav-item" v-for="(item, index) in getMinList(n)" :key="index">
									<button type='button' 
											:class="{'btn-primary': selectedDate && item.value == selectedDate.getMinutes()}"
									      	class="btn btn-default picker-button"
							        		@click="selectedDetailTime('min', item.value)"
										  >{{item.value}}</button>
								</td>
							</tr>
						</table>
					</div>
				</div>
			</div>
		</transition>
	</div>
</template>
<script>
module.exports = {
	mixins: [VueClickaway.mixin], //重要
	props : {
		// 元件顯示語言
	    language: {
	        type: [String],
	        default: 'en'
	    },
	    // 顯示值的年制設定(en, zh)
	    displaylang: {
	        type: [String],
	        default: 'en'
	    },
	    // 回傳值的年制設定(en, zh)
	    returnlang: {
	        type: [String],
	        default: 'en'
	    },
	    // 顯示值
	    value: {
	        type: [String, Array, Date],
	    },
	    // display格式
	    displayformat: {
	        type: [String],
	        default: 'YYYY-MM-DD'
	    },
	    // 值的格式
	    valueformat: {
	        type: [String],
	        default: 'YYYY.MM.DD'
	    },
	    // 選擇type:normal, year, month, time
	    type : {
	        default: 'date',
	    },
	    isdisabled : {
	        default: false,
	    }
	},
	data: function() {
		let now = new Date();
		let startDate, endDate, selected, tempDateList;
		
		var tmpValue;
		if(this.value) {
			// 資料類型為月份時，補日期
		    if(this.type == 'month') {
		        tmpValue = this.value.replace(/[&\|\\\*^%$#@\-]/g, '').replace(/\//g, '').replace('.', '') + '01';
		    } 
			// 資料類別為年份時，補-01-01
		    else if(this.type == 'year') {
		        tmpValue = this.value.replace(/[&\|\\\*^%$#@\-]/g, '').replace(/\//g, '').replace('.', '') + '-01-01';
		    } 
			// 資料類別為年份時，補-01-01
		    else if(this.type == 'year') {
		        tmpValue = this.value.replace(/[&\|\\\*^%$#@\-]/g, '').replace(/\//g, '').replace('.', '') + '-01-01';
		    } 
		    // 資料類別設定為normal時，不處理字串
		    else {
		        tmpValue = this.value;
		    }

			selected = moment(tmpValue, this.valueformat).toDate();
		}
		else {
			selected = new Date(now.getFullYear(), now.getMonth(), now.getDate());
		}	
		startDate = endDate = this.value ? moment(tmpValue, this.valueformat).toDate() : now
		tempDateList = this.dateLists(this.value ? moment(tmpValue, this.valueformat).toDate() : now);
		return {
		    initialized: false,
			startDate: startDate,
			endDate: endDate,
			selectedDate: selected,
			showCancel : false,
			panelState: false,
			tmpYear: startDate.getFullYear(),
			tmpMonth: startDate.getMonth() + 1,
			tmpDay: startDate.getDate(),
			tmpValue: undefined,
			weekList: [0, 1, 2, 3, 4, 5, 6],
			dateList: tempDateList,
			dateType: this.type,
			tmpType: this.type,
		}
	},
	filters: {
		// 處理畫面顯示年制
	    displayFormatter: function(value, selected, language, valueformat, displayformat, type) {
	    	if(value === undefined || value == null || value == '') {
	    		return '';
	    	}
			var tmpValue = '';
			// 資料類型為月份時，補日期
		    if(type == 'month') {
		        tmpValue = value.replace(/[&\|\\\*^%$#@\-]/g, '').replace(/\//g, '').replace('.', '') + '01';
		    } 
			// 資料類別為年份時，補-01-01
		    else if(type == 'year') {
		        tmpValue = value.replace(/[&\|\\\*^%$#@\-]/g, '').replace(/\//g, '').replace('.', '') + '-01-01';
		    } 
		    // 資料類別設定為normal時，不處理字串
		    else {
		        tmpValue = value;
		    }
			selected = moment(tmpValue, valueformat).toDate();
	        if(language == 'zh') {
	        	if(type != 'time') {
	        		var returnVal = moment(selected).subtract(1911, 'years')
	        							.format(displayformat || 'YYYY-MM-DD')
	        							.replace(/^[0]+/, '');
	        		if((displayformat || 'YYYY-MM-DD').split('Y').length - 1 == 2 &&
	        		   (displayformat || 'YYYY-MM-DD').length != returnVal.length) {
	        			return "0" + returnVal;
	        		}else
						return returnVal;
	        	}
	            else
	            	return moment(selected).subtract(1911, 'years').format(displayformat || 'YYYY-MM-DD');
	        }
	        return moment(selected).format(displayformat || 'YYYY-MM-DD');
	    }
	},
	methods: {
		// 轉換月份顯示
		month: function(index, lang) {
			var item = {};
			switch (lang) {
				case 'en':
					item = {1: 'Jan', 2: 'Feb', 3: 'Mar', 4: 'Apr', 5: 'May', 6: 'Jun',
					 7: 'Jul', 8: 'Aug', 9: 'Sep', 10: 'Oct', 11: 'Nov', 12: 'Dec'}
					break;
				case 'zh':
					item = {1: '一月', 2: '二月', 3: '三月', 4: '四月', 5: '五月', 6: '六月',
					 7: '七月', 8: '八月', 9: '九月', 10: '十月', 11: '十一月', 12: '十二月'}
					break;
			}
			return item[index]
		},
		// 轉換週期顯示
		week: function(index, lang) {
			var item = {};
			switch (lang) {
				case 'en':
					item =  {0: 'Sun', 1: 'Mon', 2: 'Tue', 3: 'Wed', 4: 'Thu', 5: 'Fri', 6: 'Sat'}
					break;
				case 'zh':
					item =  {0: '日', 1: '一', 2: '二', 3: '三', 4: '四', 5: '五', 6: '六'}
					break;
			}
			return item[index]
		},
		// 依rowIndex顯示datelist
		getDateList: function(index) {
			return this.dateList.slice(((index - 1) * 7), 7 * index);
		},
		// 依rowIndex顯示yearlist
		getYearList: function(index) {
			return this.yearList.slice(((index - 1) * 4), 4 * index);
		},
		// 依rowIndex顯示monthlist
		getMonthList: function(index) {			
			return this.monthList.slice(((index - 1) * 4), 4 * index);
		},
		// 依rowIndex顯示hourlist
		getHourList: function(index) {			
			return this.hourList.slice(((index - 1) * 6), 6 * index);
		},
		// 依rowIndex顯示minlist
		getMinList: function(index) {			
			return this.minList.slice(((index - 1) * 10), 10 * index);
		},
		
		
		// 確認是否為今日
		checkToday: function(day) {
			var now = new Date();
			
			return this.tmpYear == now.getFullYear() &&
			   this.tmpMonth == now.getMonth() + 1 && day == now.getDate();
		},
		// 確認是否為假日
		checkHoliday: function(index,startDay) {
			//return 1 != 1;
			return (index + startDay) % 7 == 0 || (index + startDay) % 7 == 6;
		},
		// 確認是否為假日
		checkMonthday: function() {
			return 1 == 1;
		},
		// 顯示下個月
		nextMonth: function() {
            //終止預設行為
			if(this.tmpMonth + 1 > 12) {this.tmpYear += 1; this.tmpMonth = 1;}
			else { this.tmpMonth += 1; }
			this.dateList = this.dateLists(new Date(this.tmpYear, this.tmpMonth - 1, 1));
		},
		// 顯示上個月
		lastMonth: function() {
			if(this.tmpMonth - 1 < 1) {this.tmpYear -= 1; this.tmpMonth = 12;}
			else { this.tmpMonth -= 1; }
			this.dateList = this.dateLists(new Date(this.tmpYear, this.tmpMonth - 1, 1));
		},
		// 選擇日期
		selectDate: function(day) {
		    this.clearSelected();
			this.tmpDay = day.value;
			this.tmpMonth = day.month + 1;
			this.tmpYear = day.year;
			//console.log('selectedDate: ' + moment(moment(this.selectedDate)
			//.toDate())
			//.format("YYYY-MM-DD HH:mm"));
			
			let value = new Date(day.year, day.month, this.tmpDay, 
			                             this.selectedDate.getHours(),
			                             this.selectedDate.getMinutes());
			// 將值丟入input, 並且parse為輸出值格式
			this.$emit('input', this.parse(value));
			day.isSelected = true;
			// 關閉panel
			if(this.type == 'date') {
				this.panelState = false;
			} else 
				this.tmpType = 'time';
		},
		// 選擇月份
		selectMonth: function(month){
		    this.tmpMonth = month.value;
			// 若選擇模式為month
		    if(this.dateType == 'month') {
		    	// 回傳值
				let value = new Date(this.tmpYear, this.tmpMonth-1, 1, 
			                             this.selectedDate.getHours(),
			                             this.selectedDate.getMinutes());
				// 將值丟入input, 並且parse為輸出值格式
                this.$emit('input', this.parse(value));
                // 關閉panel
                this.panelState = false;
		    } else {
		    	this.dateList = this.dateLists(new Date(this.tmpYear, this.tmpMonth - 1, 1));
		    	this.tmpType = 'date';
		    }	
		},
		// 選擇年份
		selectYear: function(year){
			this.tmpYear = year.value;
		    if(this.dateType == 'year') {
				let value = new Date(this.tmpYear, 0, 1, 
			                             this.selectedDate.getHours(),
			                             this.selectedDate.getMinutes());
				// 將值丟入input, 並且parse為輸出值格式
                this.$emit('input', this.parse(value));
                // 關閉panel
                this.panelState = false;
		    } else 
				this.tmpType = 'month';
		},
		selectedTime: function(status, val) {
			let value = new Date(this.selectedDate.getFullYear(), 
										 this.selectedDate.getMonth(), 
			                             this.selectedDate.getDate(), 
			                             this.selectedDate.getHours() + (status == 'hour' ? val : 0),
			                             this.selectedDate.getMinutes() + (status == 'min' ? val : 0));
			// 將值丟入input, 並且parse為輸出值格式
			this.$emit('input', this.parse(value));
		},
		selectedDetailTime: function(status, val) {
			let value = new Date(this.selectedDate.getFullYear(), 
										 this.selectedDate.getMonth(), 
			                             this.selectedDate.getDate(), 
			                            (status == 'hour' ? val : this.selectedDate.getHours()),
			                            (status == 'min' ? val : this.selectedDate.getMinutes()));
			// 將值丟入input, 並且parse為輸出值格式
			this.$emit('input', this.parse(value));
		},
		// 確認是否選取
		checkSelected: function(year, month, day) {
			var tmpDate;
			if (this.selectedDate)
				tmpDate = this.selectedDate;
			else
				tmpDate = moment(this.value).toDate();
			return tmpDate && 
			       year == tmpDate.getFullYear() &&
			       month == tmpDate.getMonth() &&
			       day == tmpDate.getDate();
		},
		// 組dateList
		dateLists: function(date) {
			// 取得本月從週期幾開始
			let startDay = new Date(date.getFullYear(), date.getMonth(), 1).getDay();
			
			//console.log('selectedDate: ' + moment(moment(this.selectedDate)
			//.toDate())
			//.format("YYYY-MM-DD HH:mm"));
			
			//console.log('tmpDate: ' + moment(moment(date)
			//.toDate())
			//.format("YYYY-MM-DD HH:mm"));
			
			// 取得本月天數
		    let currentMonthLength = new Date(date.getFullYear(), date.getMonth() + 1, 0).getDate();
			let vm = this;
			
			// 組成本月天數資料
			let temp = Array.apply(null, new Array(currentMonthLength)).map(function (val, index) {
				return {
					year: date.getFullYear(),
					month:  date.getMonth(),
					currentMonth: true, // 設定為本月日期
					value: index + 1,
					isToday: vm.checkToday(index + 1),
					isHoliday: vm.checkHoliday(index,startDay),
					isMonthday: vm.checkMonthday(),
					isSelected: vm.checkSelected(date.getFullYear(), date.getMonth(), index + 1)
				}
			})
			
			// 取得上月天數
			let previousMongthLength = new Date(date.getFullYear(), date.getMonth(), 0).getDate();
			var preMonthDate = new Date(date.getFullYear(), date.getMonth()-1, 1);
			// 處理上月日期資料
			for(let i = 0, len = startDay; i < len; i++){
			    temp = [{
					year: preMonthDate.getFullYear(),
					month: preMonthDate.getMonth(),
					previousMonth: true, 
					value: previousMongthLength - i}]
					.concat(temp)
			}
			
			// 處理下月日期資料，補到42天
			for(let i = temp.length, item = 1; i < 42; i++, item++) {
			    temp[temp.length] = {
					year: new Date(date.getFullYear(), date.getMonth() + 1, 1).getFullYear(),
					month: new Date(date.getFullYear(), date.getMonth() + 1, 1).getMonth(),
					nextMonth: true, 
					value: item
				}
			}
			
			return temp;
		},
		// 清空選擇
		clearSelected: function() {
		    $.each(this.dateList.filter(function (item) {
			    return item.isSelected;
			}), function(index, item) {
			    item.isSelected = false;
			});
		},
		// 剖析為valueformat
		parse: function(value) {
		    if(this.returnlang == 'zh') {
		    	// 處理民國年顯示，-1911年，並且將字首0去除
		    	if(this.dateType != 'time')
	            	return moment(value).subtract(1911, 'years').format(this.valueformat || 'YYYY-MM-DD')
	            						.replace(/^[0]/, '') ;
	            else {
	            	return moment(value).subtract(1911, 'years').format(this.valueformat || 'YYYY-MM-DD');
	            }
		    }
		    return moment(value).format(this.valueformat || 'YYYY-MM-DD');
		},
		
		close: function() {
			this.panelState = false;
		},
		clear: function() {
			this.clearSelected();
        	return this.$emit('input', "");
      	},
      	toggleState: function() {
      		console.log(this.isdisabled);
      		if(!this.isdisabled)
      			this.panelState = !this.panelState;
      	}
	},
	computed: {
		// 計算月份資料
		monthList: function() {
			var now = new Date();
			
			var monthList = Array.apply(null, new Array(12)).map(function (val, index) {
				return {
					value: index + 1,
					isToday: this.tmpYear == now.getFullYear() && index == now.getMonth(),
					isMonthday: this.tmpYear == now.getFullYear() && index == now.getMonth(),
					isHoliday: false,
				}
			})
			return monthList;
		},
		// 計算年份資料
		yearList: function() {
			var tmpYear = this.tmpYear;
			var now = new Date();
			var yearList = Array.apply(null, new Array(12)).map(function (val, index) {
				return {
					value: (tmpYear - (tmpYear % 12)) + index,
					isToday: this.tmpYear == now.getFullYear(),
					isMonthday: this.tmpYear == now.getFullYear(),
					isHoliday: false,
				}
			});
			return yearList;
		},
		// 小時清單
		hourList: function() {
			return Array.apply(null, new Array(24)).map(function (val, index) { 
						return {
						value: (index < 10 ? '0' : '') + index
						}
			});
		},
		// 分鐘清單
		minList: function() {
			return Array.apply(null, new Array(60)).map(function (val, index) { 
						return {
						value: (index < 10 ? '0' : '') + index
						}
			});
		},
	},
	watch: {
		'type' : function(val, oldVal) {
			// 動態設定type時，元件內的dateType應跟著連動
			if(val != this.dateType)
				this.dateType = val;
		},
		'value' : function(val, oldVal) {
			// 動態設定value時，元件內的selectedDate應跟著連動
			if(this.value) {
				// 資料類型為月份時，補日期
			    if(this.type == 'month') {
			        tmpValue = this.value.replace(/[&\|\\\*^%$#@\-]/g, '').replace(/\//g, '').replace('.', '') + '01';
			    } 
				// 資料類別為年份時，補-01-01
			    else if(this.type == 'year') {
			        tmpValue = this.value.replace(/[&\|\\\*^%$#@\-]/g, '').replace(/\//g, '').replace('.', '') + '-01-01';
			    } 
				// 資料類別為年份時，補-01-01
			    else if(this.type == 'year') {
			        tmpValue = this.value.replace(/[&\|\\\*^%$#@\-]/g, '').replace(/\//g, '').replace('.', '') + '-01-01';
			    } 
			    // 資料類別設定為normal時，不處理字串
			    else {
			        tmpValue = this.value;
			    }
	
				selected = moment(tmpValue, this.valueformat).toDate();
				this.selectedDate = selected;
			} else {
				this.selectedDate = new Date();
			}
		},
		'panelState' : function(val, oldVal) {
			// 開啟日期選單時，重新初始化
			if (val) {
				this.tmpType = this.dateType;
				
				var tmpDate;
				if (this.selectedDate)
					tmpDate = this.selectedDate;
				else
					tmpDate = moment(this.value).toDate();
				
				this.tmpYear = tmpDate.getFullYear();
				this.tmpMonth = tmpDate.getMonth() + 1;
				this.tmpDate = tmpDate.getDate();
				
				this.dateList = this.dateLists(new Date(this.tmpYear, this.tmpMonth - 1, 1));
			}
		}
	},
	mounted: function() {
	},
    beforeDestroy: function() {
    }
}
</script>
<style>
	.preMonth, .nextMonth {
		color: #ccc;
	}
	.date-picker {
		position: relative;
	},
	
	.toggle-enter, .toggle-leave-active {
		opacity: 0;
		transform: translateY(-10px);
	}
	.toggle-enter-active, .toggle-leave-active {
		transition: all ease .2s;
	}
	.fade-enter, .fade-leave-active {
		opacity: 0;
		transform: scale3d(0, 0, 0);
	}
	.fade-enter-active, .fade-leave-active {
		transition: all ease .1s;
	}
	.picker-button-1 {
		width: 55px;
	}
	
	.picker-button {
		width: 100%;
		border-radius: 100%;
	}
	
	.picker-button-1:hover, .picker-button:hover {
	    background-color: #4286f4;
	    color: white;
	}
	
    #searchclear {
        position: absolute;
        right: 5px;
        top: 0;
        bottom: 0;
        height: 14px;
        margin: auto;
        font-size: 14px;
        cursor: pointer;
        color: #ccc;
    }
    
    .hasclear {
	    background-color: white !important;
	    border: none;
	    width: 100%
    }
    .isdisabled {
    	background-color: #eee !important;
    }
    
     .btn-options {
     	width: 30px;
     }
</style>

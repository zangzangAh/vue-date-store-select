<template>
  <transition name="el-zoom-in-top"  :close-on-click-modal	=false :close-on-press-escape="false" :show-close="false">
    <div class="el-picker-panel el-date-picker el-popper"  style="width: 452px;margin:0; box-shadow: 0 0 0 0 white;">
      <div class="el-picker-panel__body-wrapper" style="margin:0">
        <slot name="sidebar" class="el-picker-panel__sidebar"></slot>
        <div class="el-picker-panel__sidebar" v-if="shortcuts">
          <button
                  type="button"
                  class="el-picker-panel__shortcut"
                  v-for="(shortcut, key) in shortcuts"
                  :key="key"
                  @click="handleShortcutClick(shortcut)">{{ shortcut.text }}</button>
        </div>
        <div class="el-picker-panel__body">
          <div class="el-date-picker__time-header" v-if="showTime">
            <span class="el-date-picker__editor-wrap">
              <el-input
                      :placeholder="t('el.datepicker.selectDate')"
                      :value="visibleDate"
                      size="small"
                      @input="val => userInputDate = val"
                      @change="handleVisibleDateChange" />
            </span>
            <span class="el-date-picker__editor-wrap" v-clickoutside="handleTimePickClose">
              <el-input
                      ref="input"
                      @focus="timePickerVisible = true"
                      :placeholder="t('el.datepicker.selectTime')"
                      :value="visibleTime"
                      size="small"
                      @input="val => userInputTime = val"
                      @change="handleVisibleTimeChange" />
            </span>
          </div>
          <div
                  class="el-date-picker__header"
                  :class="{ 'el-date-picker__header--bordered': currentView === 'year' || currentView === 'month' }"
                  v-show="currentView !== 'time'">
            <button
                    type="button"
                    @click="prevYear"
                    :aria-label="t(`el.datepicker.prevYear`)"
                    class="el-picker-panel__icon-btn el-date-picker__prev-btn el-icon-d-arrow-left">
            </button>
            <button
                    type="button"
                    @click="prevMonth"
                    v-show="currentView === 'date'"
                    :aria-label="t(`el.datepicker.prevMonth`)"
                    class="el-picker-panel__icon-btn el-date-picker__prev-btn el-icon-arrow-left">
            </button>
            <span
                    role="button"
                    class="el-date-picker__header-label">{{ yearLabel }}</span>
            <span
                    v-show="currentView === 'date'"
                    role="button"
                    class="el-date-picker__header-label"
                    :class="{ active: currentView === 'month' }">{{t(`el.datepicker.month${ month + 1 }`)}}</span>
            <button
                    type="button"
                    @click="nextYear"
                    :aria-label="t(`el.datepicker.nextYear`)"
                    class="el-picker-panel__icon-btn el-date-picker__next-btn el-icon-d-arrow-right">
            </button>
            <button
                    type="button"
                    @click="nextMonth"
                    v-show="currentView === 'date'"
                    :aria-label="t(`el.datepicker.nextMonth`)"
                    class="el-picker-panel__icon-btn el-date-picker__next-btn el-icon-arrow-right">
            </button>
          </div>
          <div class="el-picker-panel__content" style="width: 428px">
            <table cellspacing="0"
                   cellpadding="0"
                   class="el-date-table"
                   @mousemove="handleMouseMove"
                   :class="{ 'is-week-mode': selectionMode === 'week' }" >
              <tbody>
              <tr   v-on:click="handleClick"  >
                <th v-if="showWeekNumber" >{{ t('el.datepicker.week') }}</th>
                <th v-for="(week, key) in WEEKS" :key="key">{{ t('el.datepicker.weeks.' + week) }}</th>
              </tr>
              <tr
                      class="el-date-table__row"
                      v-for="(row, key) in rows"
                      :class="{ current: isWeekActive(row[1]) }" style="width:60px;"
                      :key="key"   v-on:click="handleClick">
                <td  v-for="(cell, key) in row" :class="getCellClasses(cell)" :key="key"  >
                  <div style="height: 25px">
                    <span v-if="cell.type === 'today' " class="is-today" style="width:12px;height:4px;left: 50%;top: 28px;position:absolute " ></span>
                    <span v-if="ifHave(row,cell)" class="is-event" style=" width:33px;height:33px;left: 50%; top: 5px;  z-index: 1;position:absolute"></span>
                    <span style="text-align: left;z-index: 2;position:absolute;margin-left: 8%;">
                      {{ cell.text }}
                    </span>
                  </div>
                  <div style="font-size: 8px; ">
                    {{ title }}:{{ cell.remainNum }}
                  </div>
                </td>
              </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script type="text/babel">
    import {
        formatDate,
        parseDate,
        getWeekNumber,
        isDate,
        modifyDate,
        modifyTime,
        modifyWithTimeString,
        clearMilliseconds,
        clearTime,
        prevYear,
        nextYear,
        prevMonth,
        nextMonth,
        extractDateFormat,
        extractTimeFormat,
        getFirstDayOfMonth,
        getDayCountOfMonth,
        getStartDateOfMonth,
        nextDate,
        clearTime as _clearTime
    } from './elementDate';
    import Clickoutside from 'element-ui/src/utils/clickoutside';
    import Locale from 'element-ui/src/mixins/locale';
    import ElInput from 'element-ui/packages/input';
    import ElButton from 'element-ui/packages/button';
    import YearTable from 'element-ui/packages/date-picker/src/basic/year-table';
    import MonthTable from 'element-ui/packages/date-picker/src/basic/month-table';
    import DateTable from 'element-ui/packages/date-picker/src/basic/date-table';

    import { arrayFindIndex, arrayFind, coerceTruthyValueToArray } from 'element-ui/src/utils/util';

    const WEEKS = ['sun', 'mon', 'tue', 'wed', 'thu', 'fri', 'sat'];
    const getDateTimestamp = function(time) {
        if (typeof time === 'number' || typeof time === 'string') {
            return _clearTime(new Date(time)).getTime();
        } else if (time instanceof Date) {
            return _clearTime(time).getTime();
        } else {
            return NaN;
        }
    };

    const removeFromArray = function(arr, pred) {
        const idx = typeof pred === 'function' ? arrayFindIndex(arr, pred) : arr.indexOf(pred);
        return idx >= 0 ? [...arr.slice(0, idx), ...arr.slice(idx + 1)] : arr;
    };
    export default {
        name: 'date-store-page',
        mixins: [Locale],
        directives: { Clickoutside },
        props: {
            title:{
                type: String,
                default:'库存' ,
                required: false
            },
            disabled:{
                type: Boolean,
                default:false ,
                required: false
            },
            todayDisabled:{
                type: Boolean,
                default:false ,
                required: false
            },

            storeList : {
                type: Array,
                required: true
            },
            storeNum: {
                type: Number,
                required: true
            },
            selectList: {
                type: Array,
                required: true
            },
            defaultValue: {
                validator(val) {
                    // either: null, valid Date object, Array of valid Date objects
                    return val === null || isDate(val) || (Array.isArray(val) && val.every(isDate));
                }
            },

            disabledDate: {},
            minDate: {},
            maxDate: {},
            rangeState: {
                default() {
                    return {
                        endDate: null,
                        selecting: false
                    };
                }
            }
        },
        components: {
            YearTable, MonthTable, DateTable, ElInput, ElButton
        },

        data() {
            return {
                value: {},
                tableRows: [ [], [], [], [], [], [] ],
                lastRow: null,
                lastColumn: null,
                date: new Date(),
                popperClass: '',
                defaultTime: null,
                showTime: false,
                selectionMode: 'day',
                shortcuts: '',
                visible: true,
                currentView: 'date',
                firstDayOfWeek: 7,
                showWeekNumber: false,
                timePickerVisible: false,
                format: '',
                arrowControl: false,
                userInputDate: null,
                userInputTime: null
            };
        },
        watch: {
            showTime(val) {
                /* istanbul ignore if */
                if (!val) return;
                this.$nextTick(_ => {
                    const inputElm = this.$refs.input.$el;
                    if (inputElm) {
                        this.pickerWidth = inputElm.getBoundingClientRect().width + 10;
                    }
                });
            },

            value(val) {
                if (this.selectionMode === 'dates' && this.value) return;
                if (isDate(val)) {
                    this.date = new Date(val);
                } else {
                    this.date = this.getDefaultValue();
                }
            },

            defaultValue(val) {
                if (!isDate(this.value)) {
                    this.date = val ? new Date(val) : new Date();
                }
            },


            selectionMode(newVal) {
                if (newVal === 'month') {
                    /* istanbul ignore next */
                    if (this.currentView !== 'year' || this.currentView !== 'month') {
                        this.currentView = 'month';
                    }
                } else if (newVal === 'dates') {
                    this.currentView = 'date';
                }
            },
            'rangeState.endDate'(newVal) {
                this.markRange(this.minDate, newVal);
            },

            minDate(newVal, oldVal) {
                if (getDateTimestamp(newVal) !== getDateTimestamp(oldVal)) {
                    this.markRange(this.minDate, this.maxDate);
                }
            },

            maxDate(newVal, oldVal) {
                if (getDateTimestamp(newVal) !== getDateTimestamp(oldVal)) {
                    this.markRange(this.minDate, this.maxDate);
                }
            },
            selectList:{
                handler: function (val,oldVal) {
                    this.selectList = val ;
                },
                deep: true
            },
            storeList:{
                handler: function (val,oldVal) {
                    this.storeList = val;
                },
                deep: true
            },
            storeNum:{
                handler: function (val,oldVal) {
                    this.storeNum = val;
                },
                deep: true
            },
        },

        methods: {
            handleClear() {
                this.date = this.getDefaultValue();
                this.$emit('pick', null);
            },

            emit(value, ...args) {
                if (!value) {
                    this.$emit('pick', value, ...args);
                } else if (Array.isArray(value)) {
                    const dates = value.map(date => this.showTime ? clearMilliseconds(date) : clearTime(date));
                    this.$emit('pick', dates, ...args);
                } else {
                    this.$emit('pick', this.showTime ? clearMilliseconds(value) : clearTime(value), ...args);
                }
                this.userInputDate = null;
                this.userInputTime = null;
            },

            prevMonth() {
                this.date = prevMonth(this.date);
            },

            nextMonth() {
                this.date = nextMonth(this.date);
            },

            prevYear() {
                if (this.currentView === 'year') {
                    this.date = prevYear(this.date, 10);
                } else {
                    this.date = prevYear(this.date);
                }
            },

            nextYear() {
                if (this.currentView === 'year') {
                    this.date = nextYear(this.date, 10);
                } else {
                    this.date = nextYear(this.date);
                }
            },

            handleShortcutClick(shortcut) {
                if (shortcut.onClick) {
                    shortcut.onClick(this);
                }
            },

            handleTimePick(value, visible, first) {
                if (isDate(value)) {
                    const newDate = this.value
                        ? modifyTime(this.value, value.getHours(), value.getMinutes(), value.getSeconds())
                        : modifyWithTimeString(this.getDefaultValue(), this.defaultTime);
                    this.date = newDate;
                    this.emit(this.date, true);
                } else {
                    this.emit(value, true);
                }

            },
            handleDatePick(value) {
                if (this.selectionMode === 'day') {
                    this.date = this.value
                        ? modifyDate(this.value, value.getFullYear(), value.getMonth(), value.getDate())
                        : modifyWithTimeString(value, this.defaultTime);
                    this.emit(this.date, this.showTime);
                } else if (this.selectionMode === 'week') {
                    this.emit(value.date);
                } else if (this.selectionMode === 'dates') {
                    this.emit(value, true); // set false to keep panel open
                }
            },



            changeToNow() {
                // NOTE: not a permanent solution
                //       consider disable "now" button in the future
                if (!this.disabledDate || !this.disabledDate(new Date())) {
                    this.date = new Date();
                    this.emit(this.date);
                }
            },



            resetView() {
                if (this.selectionMode === 'month') {
                    this.currentView = 'month';
                } else if (this.selectionMode === 'year') {
                    this.currentView = 'year';
                } else {
                    this.currentView = 'date';
                }
            },

            handleEnter() {
                document.body.addEventListener('keydown', this.handleKeydown);
            },

            handleLeave() {
                this.$emit('dodestroy');
                document.body.removeEventListener('keydown', this.handleKeydown);
            },

            handleKeydown(event) {
                const keyCode = event.keyCode;
                const list = [38, 40, 37, 39];
                if (this.visible && !this.timePickerVisible) {
                    if (list.indexOf(keyCode) !== -1) {
                        this.handleKeyControl(keyCode);
                        event.stopPropagation();
                        event.preventDefault();
                    }
                    if (keyCode === 13 && this.userInputDate === null && this.userInputTime === null) { // Enter
                        this.emit(this.date, false);
                    }
                }
            },

            handleKeyControl(keyCode) {
                const mapping = {
                    'year': {
                        38: -4, 40: 4, 37: -1, 39: 1, offset: (date, step) => date.setFullYear(date.getFullYear() + step)
                    },
                    'month': {
                        38: -4, 40: 4, 37: -1, 39: 1, offset: (date, step) => date.setMonth(date.getMonth() + step)
                    },
                    'week': {
                        38: -1, 40: 1, 37: -1, 39: 1, offset: (date, step) => date.setDate(date.getDate() + step * 7)
                    },
                    'day': {
                        38: -7, 40: 7, 37: -1, 39: 1, offset: (date, step) => date.setDate(date.getDate() + step)
                    }
                };
                const mode = this.selectionMode;
                const year = 3.1536e10;
                const now = this.date.getTime();
                const newDate = new Date(this.date.getTime());
                while (Math.abs(now - newDate.getTime()) <= year) {
                    const map = mapping[mode];
                    map.offset(newDate, map[keyCode]);
                    if (typeof this.disabledDate === 'function' && this.disabledDate(newDate)) {
                        continue;
                    }
                    this.date = newDate;
                    this.$emit('pick', newDate, true);
                    break;
                }
            },

            handleVisibleTimeChange(value) {
                const time = parseDate(value, this.timeFormat);
                if (time) {
                    this.date = modifyDate(time, this.year, this.month, this.monthDate);
                    this.userInputTime = null;
                    this.$refs.timepicker.value = this.date;
                    this.timePickerVisible = false;
                    this.emit(this.date, true);
                }
            },

            handleVisibleDateChange(value) {
                const date = parseDate(value, this.dateFormat);
                if (date) {
                    if (typeof this.disabledDate === 'function' && this.disabledDate(date)) {
                        return;
                    }
                    this.date = modifyTime(date, this.date.getHours(), this.date.getMinutes(), this.date.getSeconds());
                    this.userInputDate = null;
                    this.resetView();
                    this.emit(this.date, true);
                }
            },

            isValidValue(value) {
                return value && !isNaN(value) && (
                    typeof this.disabledDate === 'function'
                        ? !this.disabledDate(value)
                        : true
                );
            },

            getDefaultValue() {
                // if default-value is set, return it
                // otherwise, return now (the moment this method gets called)
                return this.defaultValue ? new Date(this.defaultValue) : new Date();
            },
            cellMatchesDate(cell, date) {
                const value = new Date(date);
                return this.year === value.getFullYear() &&
                    this.month === value.getMonth() &&
                    Number(cell.text) === value.getDate();
            },
            ifHave(row ,cell){
                for( let select of this.selectList){
                    if( select.date == cell.date){
                        return true;
                    }
                }
                return false;
            },

            getCellClasses(cell) {
                const selectionMode = this.selectionMode;
                const defaultValue = this.defaultValue ? Array.isArray(this.defaultValue) ? this.defaultValue : [this.defaultValue] : [];

                let classes = [];
                if ((cell.type === 'normal' || cell.type === 'today') && !cell.disabled) {
                    classes.push('available');
                    if (cell.type === 'today') {
                        classes.push('today');
                    }
                } else {
                    classes.push(cell.type);
                }

                if (cell.type === 'normal' && defaultValue.some(date => this.cellMatchesDate(cell, date))) {
                    classes.push('default');
                }

                if (selectionMode === 'day' && (cell.type === 'normal' || cell.type === 'today') && this.cellMatchesDate(cell, this.value)) {
                    classes.push('current');
                }

                if (cell.inRange && ((cell.type === 'normal' || cell.type === 'today') || this.selectionMode === 'week')) {
                    classes.push('in-range');

                    if (cell.start) {
                        classes.push('start-date');
                    }

                    if (cell.end) {
                        classes.push('end-date');
                    }
                }

                if(this.disabled) classes.push('disabled');
                else {
                    var oDate1 =  formatDate(cell.date) ;
                    var today =  formatDate(new Date());
                    if( oDate1  < today ){ classes.push('disabled');  }//今天之前的日期不能选择
                    else if( oDate1 ==today  &&  this.todayDisabled ) { classes.push('disabled');  }
                }

                if(cell.remainNum==0){ classes.push('disabled')}//剩余数为0时不能选择
                if (cell.selected) {
                    classes.push('selected');
                }
                if(cell.column === 6){
                    // let  style="border:solid #e4e7ed; border-width:0px 0px 1px 0px;"
                    classes.push('lastColumnStyle');
                }else{
                    // let  style="border:solid #e4e7ed; border-width:0px 1px 1px 0px;"
                    classes.push('normalColumnStyle');
                }

                return classes.join(' ');
            },

            getDateOfCell(row, column) {
                const offsetFromStart = row * 7 + (column - (this.showWeekNumber ? 1 : 0)) - this.offsetDay;
                return nextDate(this.startDate, offsetFromStart);
            },

            isWeekActive(cell) {
                if (this.selectionMode !== 'week') return false;
                const newDate = new Date(this.year, this.month, 1);
                const year = newDate.getFullYear();
                const month = newDate.getMonth();

                if (cell.type === 'prev-month') {
                    newDate.setMonth(month === 0 ? 11 : month - 1);
                    newDate.setFullYear(month === 0 ? year - 1 : year);
                }

                if (cell.type === 'next-month') {
                    newDate.setMonth(month === 11 ? 0 : month + 1);
                    newDate.setFullYear(month === 11 ? year + 1 : year);
                }

                newDate.setDate(parseInt(cell.text, 10));

                const valueYear = isDate(this.value) ? this.value.getFullYear() : null;
                return year === valueYear && getWeekNumber(newDate) === getWeekNumber(this.value);
            },

            markRange(minDate, maxDate) {
                minDate = getDateTimestamp(minDate);
                maxDate = getDateTimestamp(maxDate) || minDate;
                [minDate, maxDate] = [Math.min(minDate, maxDate), Math.max(minDate, maxDate)];

                const startDate = this.startDate;
                const rows = this.rows;
                for (let i = 0, k = rows.length; i < k; i++) {
                    const row = rows[i];
                    for (let j = 0, l = row.length; j < l; j++) {
                        if (this.showWeekNumber && j === 0) continue;

                        const cell = row[j];
                        const index = i * 7 + j + (this.showWeekNumber ? -1 : 0);
                        const time = nextDate(startDate, index - this.offsetDay).getTime();

                        cell.inRange = minDate && time >= minDate && time <= maxDate;
                        cell.start = minDate && time === minDate;
                        cell.end = maxDate && time === maxDate;
                    }
                }
            },

            handleMouseMove(event) {
                if (!this.rangeState.selecting) return;

                let target = event.target;
                if (target.tagName === 'SPAN') {
                    target = target.parentNode.parentNode;
                }
                if (target.tagName === 'DIV') {
                    target = target.parentNode;
                }
                if (target.tagName !== 'TD') return;

                const row = target.parentNode.rowIndex - 1;
                const column = target.cellIndex;

                // can not select disabled date
                if (this.rows[row][column].disabled) return;

                // only update rangeState when mouse moves to a new cell
                // this avoids frequent Date object creation and improves performance
                if (row !== this.lastRow || column !== this.lastColumn) {
                    this.lastRow = row;
                    this.lastColumn = column;
                    this.$emit('changerange', {
                        minDate: this.minDate,
                        maxDate: this.maxDate,
                        rangeState: {
                            selecting: true,
                            endDate: this.getDateOfCell(row, column)
                        }
                    });
                }
            },

            handleClick(event) {

                let target = event.target;
                if (target.tagName === 'SPAN') {
                    target = target.parentNode.parentNode;
                }
                if (target.tagName === 'DIV') {
                    target = target.parentNode;
                }

                if (target.tagName !== 'TD') return;

                const row = target.parentNode.rowIndex - 1;
                const column = this.selectionMode === 'week' ? 1 : target.cellIndex;
                const cell = this.rows[row][column];

                if(this.disabled) return ;
                else{
                    var oDate1 =  formatDate(cell.date) ;
                    var today =  formatDate(new Date());
                    if( oDate1  <  today )  return ; //今天之前的日期不可操作
                    else if( oDate1 ==today &&  this.todayDisabled )  return   ;
                }

                if(cell.remainNum==0) return ;//剩余小于0 不可操作
                if (cell.disabled || cell.type === 'week') return;

                const newDate = this.getDateOfCell(row, column);

                if (this.selectionMode === 'range') {
                    if (!this.rangeState.selecting) {
                        this.$emit('pick', {minDate: newDate, maxDate: null});
                        this.rangeState.selecting = true;
                    } else {
                        if (newDate >= this.minDate) {
                            this.$emit('pick', {minDate: this.minDate, maxDate: newDate});
                        } else {
                            this.$emit('pick', {minDate: newDate, maxDate: this.minDate});
                        }
                        this.rangeState.selecting = false;
                    }
                } else if (this.selectionMode === 'day') {
                    this.$emit('pick', newDate);
                    if(this.selectList.size==0){
                        let select = { date:formatDate(newDate),num:0,remainNum:cell.remainNum };
                        this.selectList.push(select );
                    }else {
                        var addNew = true;
                        for (var i = 0; i < this.selectList.length; i++) {
                            if (formatDate(newDate)==this.selectList[i].date) {
                                addNew = false;
                                this.selectList.splice(i, 1);
                            }
                        }
                        if(addNew){
                            let select = { date:formatDate(newDate),num:0,remainNum:cell.remainNum };
                            this.selectList.push(select );
                        }
                    }
                } else if (this.selectionMode === 'week') {
                    const weekNumber = getWeekNumber(newDate);
                    const value = newDate.getFullYear() + 'w' + weekNumber;
                    this.$emit('pick', {
                        year: newDate.getFullYear(),
                        week: weekNumber,
                        value: value,
                        date: newDate
                    });
                } else if (this.selectionMode === 'dates') {
                    const value = this.value || [];
                    const newValue = cell.selected
                        ? removeFromArray(value, date => date.getTime() === newDate.getTime())
                        : [...value, newDate];
                    this.$emit('pick', newValue);
                }
            }
        },



        computed: {
            WEEKS() {
                const week = this.firstDayOfWeek;
                return WEEKS.concat(WEEKS).slice(week, week + 7);
            },
            year() {
                if(this.date){
                    return this.date.getFullYear();
                }else{
                    return new Date().getFullYear();
                }
            },

            month() {
                if(this.date){
                    return this.date.getMonth();
                }else{
                    return new Date().getMonth();
                }
            },

            week() {
                return getWeekNumber(this.date);
            },

            monthDate() {
                return this.date.getDate();
            },



            visibleTime() {
                if (this.userInputTime !== null) {
                    return this.userInputTime;
                } else {
                    return formatDate(this.value || this.defaultValue, this.timeFormat);
                }
            },

            visibleDate() {
                if (this.userInputDate !== null) {
                    return this.userInputDate;
                } else {
                    return formatDate(this.value || this.defaultValue, this.dateFormat);
                }
            },

            yearLabel() {
                const yearTranslation = this.t('el.datepicker.year');
                if (this.currentView === 'year') {
                    const startYear = Math.floor(this.year / 10) * 10;
                    if (yearTranslation) {
                        return startYear + ' ' + yearTranslation + ' - ' + (startYear + 9) + ' ' + yearTranslation;
                    }
                    return startYear + ' - ' + (startYear + 9);
                }
                return this.year + ' ' + yearTranslation;
            },

            timeFormat() {
                if (this.format) {
                    return extractTimeFormat(this.format);
                } else {
                    return 'HH:mm:ss';
                }
            },

            dateFormat() {
                if (this.format) {
                    return extractDateFormat(this.format);
                } else {
                    return 'yyyy-MM-dd';
                }
            },
            offsetDay() {
                const week = this.firstDayOfWeek;
                // 周日为界限，左右偏移的天数，3217654 例如周一就是 -1，目的是调整前两行日期的位置
                return week > 3 ? 7 - week : -week;
            },
            startDate() {
                return getStartDateOfMonth(this.year, this.month);
            },

            rows() {
                // TODO: refactory rows / getCellClasses
                const date = new Date(this.year, this.month, 1);
                let day = getFirstDayOfMonth(date); // day of first day
                const dateCountOfMonth = getDayCountOfMonth(date.getFullYear(), date.getMonth());
                const dateCountOfLastMonth = getDayCountOfMonth(date.getFullYear(), (date.getMonth() === 0 ? 11 : date.getMonth() - 1));

                day = (day === 0 ? 7 : day);
                const offset = this.offsetDay;
                const rows = this.tableRows;
                let count = 1;
                let firstDayPosition;

                const startDate = this.startDate;
                const disabledDate = this.disabledDate;
                const selectedDate = this.selectionMode === 'dates' ? coerceTruthyValueToArray(this.value) : [];
                const now = getDateTimestamp(new Date());
                for (let i = 0; i < 6; i++) {
                    const row = rows[i];

                    if (this.showWeekNumber) {
                        if (!row[0]) {
                            row[0] = { type: 'week', text: getWeekNumber(nextDate(startDate, i * 7 + 1)) };
                        }
                    }

                    for (let j = 0; j < 7; j++) {
                        let cell = row[this.showWeekNumber ? j + 1 : j];
                        if (!cell) {
                            cell = { row: i, column: j, type: 'normal', inRange: false, start: false, end: false ,time:'',date:'',remainNum:this.storeNum};
                        }
                        cell.type = 'normal';

                        const index = i * 7 + j;
                        const time = nextDate(startDate, index - offset).getTime();
                        cell.inRange = time >= getDateTimestamp(this.minDate) && time <= getDateTimestamp(this.maxDate);
                        cell.start = this.minDate && time === getDateTimestamp(this.minDate);
                        cell.end = this.maxDate && time === getDateTimestamp(this.maxDate);
                        cell.time = time ;
                        cell.date = formatDate(nextDate(startDate, index - offset)) ;
                        cell.remainNum = this.storeNum ;
                        for( let store of this.storeList){
                            if( formatDate(new Date(store.date)) ==cell.date){
                                cell.remainNum = store.remainNum;
                            }
                        }
                        const isToday = time === now;

                        if (isToday) {
                            cell.type = 'today';
                        }

                        if (i >= 0 && i <= 1) {
                            if (j + i * 7 >= (day + offset)) {
                                cell.text = count++;
                                if (count === 2) {
                                    firstDayPosition = i * 7 + j;
                                }
                            } else {
                                cell.text = dateCountOfLastMonth - (day + offset - j % 7) + 1 + i * 7;
                                cell.type = 'prev-month';
                            }
                        } else {
                            if (count <= dateCountOfMonth) {
                                cell.text = count++;
                                if (count === 2) {
                                    firstDayPosition = i * 7 + j;
                                }
                            } else {
                                cell.text = count++ - dateCountOfMonth;
                                cell.type = 'next-month';
                            }
                        }

                        let cellDate = new Date(time);
                        cell.disabled = typeof disabledDate === 'function' && disabledDate(cellDate);
                        cell.selected = arrayFind(selectedDate, date => date.getTime() === cellDate.getTime());

                        this.$set(row, this.showWeekNumber ? j + 1 : j, cell);
                    }

                    if (this.selectionMode === 'week') {
                        const start = this.showWeekNumber ? 1 : 0;
                        const end = this.showWeekNumber ? 7 : 6;
                        const isWeekActive = this.isWeekActive(row[start + 1]);

                        row[start].inRange = isWeekActive;
                        row[start].start = isWeekActive;
                        row[end].inRange = isWeekActive;
                        row[end].end = isWeekActive;
                    }
                }
                rows.firstDayPosition = firstDayPosition;

                return rows;
            }
        }
    };
</script>
<style >
  .normalColumnStyle {border:solid #e4e7ed; border-width:0px 1px 1px 0px;}
  .lastColumnStyle  {border:solid #e4e7ed; border-width:0px 0px 1px 0px;}
  .is-today{
    content: '';
    background-color: #f29543;
    border-radius: 50%;
    opacity: .8;
    width: 12px;
    height: 4px;
    position: absolute;
    top: 30px;
    z-index: 2;
    margin-left: -6px;
    margin-top: 8px;
  }
  .is-event{
    content: '';
    border: 1px solid #f29543;
    background-color: #fff;
    border-radius: 50%;
    width: 36px;
    height: 36px;
    position: absolute;
    left: 5px;
    top: 30px;
    z-index: 1;
    margin-left: -18px;
    margin-top: -19px;
  }
</style>

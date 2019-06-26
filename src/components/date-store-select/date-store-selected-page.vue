<template>
    <div class="events-wrapper"    >
        <div class="cal-events" >
            <table style="width:100%">
                <div v-for="(item, index) in selectList" class="event-item" :key="index" >

                    <tr style="text-align: left; font-size: 12px" >
                        <td   colspan="2">
                            <span style="padding-right:5px">{{item.date}}</span>剩余{{ title }}：<span style="font-weight: bold">{{item.remainNum}} </span>
                        </td>
                        <td   >
                            <i :class="selectAllIcon(index, item)" :title="selectAllTitle(index, item)"  @click="handleSelectAll(index, item)"   v-if="!ifDisabled(index, item)" style="margin-left:5px;color:dodgerblue;cursor:pointer" ></i>
                        </td>
                    </tr>
                    <tr style="font-size: 12px;text-align: left" >
                        <td  > 预定  </td>
                        <td  >
                            <el-input-number :disabled="ifDisabled(index, item)" :min="1" :max="item.remainNum"    v-model="item.num"  label="订购轮播：" size="mini" style="font-size: 12px;width: 110px" >
                            </el-input-number>
                        </td>
                        <td   >
                            <i class="el-icon-delete" title="删除"  @click="handleDelete(index, item)"   v-if="!ifDisabled(index, item)" style="margin-left:5px;color:orangered;cursor:pointer" ></i>
                        </td>
                    </tr>
                </div>
            </table>

        </div>
    </div>
</template>

<script>
    import { formatDate } from './elementDate';
    export default {
        name: 'date-store-selected-page',
        data () {
            return {

            }
        },
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
            selectList: {
                type: Array,
                required: true
            }
        },
        computed: {

        },
        watch:{
            selectList:{
                handler: function (val,oldVal) {
                    this.selectList = val;
                },
                deep: true
            },
        },
        methods: {
            ifDisabled:function(index,item){
                if(this.disabled) return true ;
                else {
                    var date =  formatDate(item.date) ;
                    var today =  formatDate(new Date());
                    if( date  < today ){ return true ;  }//今天之前的日期不能选择
                    else if (date==today) return this.todayDisabled ;
                    else return false ;
                }
            },
            selectAllIcon:function(index,item){
                if(item.num == item.remainNum) return 'el-icon-minus';
                else return 'el-icon-plus';
            },
            selectAllTitle:function(index,item){
                if(item.num == item.remainNum) return '取消全部预定';
                else return '全部预定';
            },
            handleSelectAll: function (index,item) {
                if(item.num == item.remainNum) item.num=1;
                else item.num = item.remainNum;
            },
            handleDelete: function (index,item) {
                for (var i = 0; i < this.selectList.length; i++) {
                    if (item.date==this.selectList[i].date) {
                        this.selectList.splice(i, 1);
                        return ;
                    }
                }
            },

        },
        mounted(){
        }
    }
</script>
<style>
    .events-wrapper{
        width: 180px;
        height:496px;
        border: 1px solid #e4e7ed;
    }
    .cal-events{
        height: 98%;
        overflow-y: auto;
        padding: 0 2px;
        margin: 2px 0;
    }
    .event-item{
        padding: 2px 2px;
        margin-top: 5px;
        border:1px solid rgba(0,0,0,.1);
        background-color: #fff;
        border-radius: 5px;
        color: #323232;
        position: relative;

    }
</style>

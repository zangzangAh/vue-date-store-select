<template>
    <div style="width:640px;height:500px;padding:10px;border: 1px solid #e4e7ed;   box-shadow: 0 2px 12px 0 rgba(0,0,0,.1);">
        <div style="float:left;width:460px">
            <date-store-page :title="title" :disabled="disabled"  :today-disabled="todayDisabled"  :store-list="storeList" :select-list="selectList"  :store-num="storeNum"  >
            </date-store-page>
        </div>
        <div style="float:left;width:180px;">
            <date-store-selected-page :title="title"   :disabled="disabled"  :today-disabled="todayDisabled"   :select-list="selectList" >
            </date-store-selected-page>
        </div>
        </div>
</template>

<script>
    import dateStorePage from './date-store-page';
    import dateStoreSelectedPage from './date-store-selected-page';

    export default {
        name: 'date-store-select',
        components: { dateStorePage, dateStoreSelectedPage },
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
            storeList: {
                type: Array,
                required: true
            },
            selectList: {
                type: Array,
                required: true
            },
            storeNum: {
                type: Number,
                required: true
            }
        },
        computed: {
        },
        watch:{
            selectList:{
                handler: function (val,oldVal) {
                    this.selectList = val;
                    this.$emit("listenSelectedEvent",this.selectList);
                },
                deep: true
            },
        },
        mounted(){
        }
    }
</script>
<style>

</style>

<template>
  <div style="margin:0 auto;width:50%;">
    <h3 style="color:#20A0FF">VUE——日库存预定选择组件</h3>
    <hr>
    <div style="margin:0 auto;padding-top:40px;width:600px;height:550px;text-align: left">
      <date-sotre-select title="库存" :today-disabled= todayDisabled    :store-num="dateStore" :store-list="storeList" :select-list="selectedList"    @listenSelectedEvent="bindSelectedEvent"></date-sotre-select>
    </div>
    <hr>
    <div style="text-align: left;font-size: 10px" >

      <ul>
        <li style="color:red">  基于VUE-ELEMENT组件 </li>
        <li>应用于根据每日库存预定数据</li>
      </ul>
      <h3>参数说明：</h3>
      <table >
        <tr>
          <th>参数</th>
          <th>类型</th>
          <th>默认</th>
          <th>说明</th>
        </tr>
        <tr>
          <td>title</td>
          <td>string</td>
          <td>库存</td>
          <td>显示标题</td>
        </tr>
        <tr>
          <td>disabled</td>
          <td>boolean</td>
          <td>false</td>
          <td>是否可操作</td>
        </tr>
        <tr>
          <td>todayDisabled</td>
          <td>boolean</td>
          <td>false</td>
          <td>今日是否可操作</td>
        </tr>

        <tr>
          <td>dateStore</td>
          <td>number</td>
          <td>0</td>
          <td>每日库存</td>
        </tr>
        <tr>
          <td>storeList</td>
          <td>array</td>
          <td>[]</td>
          <td>指定每日库存数据。数据格式：[{"date":"日期","storeNum":"总库存","remainStore":"剩余库存"}]</td>
        </tr>
        <tr>
          <td>selectedList</td>
          <td>array</td>
          <td>[]</td>
          <td>选择库存数据。数据格式：[{"date":"日期","num":"选择数","remainStore":"原始剩余库存"}]</td>
        </tr>
      </table>
    </div>

  </div>
</template>

<script>
  import dateSotreSelect from './date-store-select/date-store-select';
export default {
  name: 'HelloWorld',
  components: { dateSotreSelect },
  data () {
    return {
        todayDisabled:false,
      dateStore:10,
      storeList:[{"date":"2019-06-20","storeNum":5,"remainNum":5},{"date":"2019-06-26","storeNum":10,"remainNum":9},{"date":"2019-06-29","storeNum":10,"remainNum":0}],
      selectedList:[{"date":"2019-06-20","num":5,"remainNum":5},{"date":"2019-06-22","num":5,"remainNum":10},{"date":"2019-06-26","num":5,"remainNum":9}],
      msg: 'Welcome to Your Vue.js App'
    }
  },
  methods:{
    bindSelectedEvent:function(data){
      console.log("选择的数据:"+JSON.stringify(data));
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin-left: 25px;font-weight: normal;
}
li { padding:5px;}
table, td, th {   border: 1px solid lightgray;    }

  table {margin:10px 10px 10px 50px;  border-collapse: collapse;  width: 80%;  }
td {  padding:5px;  }
  th {  height: 20px; padding:5px;   }
</style>

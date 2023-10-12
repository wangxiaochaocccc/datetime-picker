<template>
    <div class='container'>
      <datetime-picker width="300px" v-model="val"/>
      <div>111</div>
      <date-range v-model="val2"/>
      <button @click="getTime" >获取时间</button>
    </div>
</template>

<script>
import datetimePicker from './libs/datetime-picker.vue';
import dateRange from './libs/date-range.vue'
export default {
  data () {
    return {
      val: '',
      val2:[]
    }
  },
  components: {
    datetimePicker,
    dateRange
  },
  methods: {
    getTime () {
      console.log(this.val2);
    }
  }
}
</script>
<style lang='scss'>
.container {
  width: 800px;
  height: 400px;
  background-color:pink;
  margin: 100px auto 0;
}
</style>
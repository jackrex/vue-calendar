<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <button v-on:click="showPop">Click</button>
    <DatePicker v-model="sendVal" title="选择日期" :parentTime="currentTime" v-on:cancel="clickCancel"></DatePicker>
  </div>
</template>

<script>

import DatePicker from './DatePicker.vue'
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      sendVal: false,
      currentTime: this.simpleDateFormat(new Date(), 'yyyy/MM/dd')
    }
  },
  components: {
    DatePicker
  },
  methods:{
    showPop: function() {
      this.msg = 'Haha'
      this.sendVal = true
    },
    clickCancel: function() {
    },
         simpleDateFormat(time, format){     // 格式化日期，ios下建议'2017/11/15',斜杠'/'代替横线'-'
                var t = new Date(time);
                var tf = function(i) {
                    return (i < 10 ? '0' : '') + i
                };
                return format.replace(/yyyy|MM|dd|HH|mm|ss/g, function(a) {
                    switch (a) {
                        case 'yyyy':
                            return tf(t.getFullYear());
                        case 'MM':
                            return tf(t.getMonth() + 1);
                        case 'mm':
                            return tf(t.getMinutes());
                        case 'dd':
                            return tf(t.getDate());
                        case 'HH':
                            return tf(t.getHours());
                        case 'ss':
                            return tf(t.getSeconds());
                    }
                })
            }    
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>

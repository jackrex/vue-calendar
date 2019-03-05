<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <button v-on:click="showPop">Click</button>
    <DatePicker v-model="sendVal" :unavailableDuration="duration" @dateClick="dateClickHandle" v-on:cancel="clickCancel"></DatePicker>
  </div>
</template>

<script>
import DatePicker from "./DatePicker.vue";
export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  data() {
    return {
      sendVal: false,
      duration: 50
    };
  },
  components: {
    DatePicker
  },
  methods: {
    showPop: function() {
      this.msg = "Haha";
      this.sendVal = true;
    },
     dateClickHandle (day, month) {
            console.log("ssss" + day + ' ' + month);
        },
    clickCancel: function() {}
  }
};
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

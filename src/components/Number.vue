<template>
  <span>
    {{current}}{{this.suffix}}
    <sup :style="'color: '+supColor" v-if="diff">+{{diff}}</sup>
  </span>
</template>

<script>
export default {
  name: "Number",
  data() {
    return {
      current: null,
      diff: null
    };
  },
  props: {
    number: [Number, String],
    suffix: String,
    speed: Number,
    color: String
  },
  mounted() {
    this.current = this.number;
  },
  watch: {
    number: function(newVal, oldVal) {
      if (!this.current) {
        this.current = newVal;
        this.diff = null;
        return;
      }
      let diff = newVal - oldVal;
      if (diff > 0) {
        this.diff = diff;
      }
      let speed = this.speed ? this.speed : 100;
      const timeValue = setInterval(() => {
        this.current++;
        if (newVal <= this.current) {
          clearInterval(timeValue);
        }
      }, speed);
    }
  },
  computed: {
    supColor: function() {
      return this.color ? this.color : "white";
    }
  }
};
</script>



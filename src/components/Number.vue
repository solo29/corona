<template>
  <span>{{current}}{{this.suffix}}</span>
</template>

<script>
export default {
  name: "Number",
  data() {
    return {
      current: null
    };
  },
  props: {
    number: [Number, String],
    suffix: String,
    speed: Number
  },
  mounted() {
    this.current = this.number;
  },
  watch: {
    number: function(newVal) {
      if (!this.current) {
        this.current = newVal;
        return;
      }
      let speed = this.speed ? this.speed : 1000;
      const timeValue = setInterval(() => {
        this.current++;
        if (newVal <= this.current) {
          clearInterval(timeValue);
        }
      }, speed);
    }
  }
};
</script>



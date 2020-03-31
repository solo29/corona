<template>
  <div class="text-gray-500">
    <div class="text-center" @click="reload">{{updated}}</div>
    <div>
      <div class="board">
        <ul class="stats">
          <li>
            <p class="stats--info">Total Infected</p>
            <Number color="yellow" :number="totalInfected" />
          </li>
          <li>
            <p class="stats--info">Total Death</p>
            <Number color="red" :number="totalDeath" />
          </li>
          <li>
            <p class="stats--info">Total Recovered</p>
            <Number color="#69c569" :number="totalRecovered" />
          </li>

          <li>
            <p class="stats--info">Total Death Rate</p>
            {{rate(totalRecovered, totalDeath)}}%
          </li>
        </ul>
      </div>
    </div>

    <div class="px-2 mt-2">
      <input
        class="uppercase text-center bg-white focus:outline-none focus:shadow-outline border border-gray-300 rounded-lg py-2 px-4 block w-full appearance-none leading-normal"
        type="text"
        @keyup="filterItems"
        v-model="filterVal"
      />
    </div>

    <div class="board" v-for="(item, i) in filteredData" :key="i+''+item.name">
      <p class="flex mt-3">
        <img
          width="30px"
          :src="require(`../assets/countries/${$options.filters.getLocale(item.name)}.png`)"
        />
        <span class="ml-2">{{item.name}}</span>
      </p>

      <ul class="stats">
        <li>
          <p class="stats--info">Infected</p>
          <Number color="yellow" :number="item.infected" />
        </li>
        <li>
          <p class="stats--info">Death</p>
          <Number color="red" :number="item.death" />
        </li>
        <li>
          <p class="stats--info">Recovered</p>
          <Number color="#69c569" :number="item.recovered" />
        </li>
        <li>
          <p class="stats--info">Death Rate</p>
          {{rate(item.recovered, item.death) }}%
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import Number from "./Number.vue";
import countries from "../countries";
export default {
  name: "Main",
  components: {
    Number
  },
  data() {
    return {
      filterVal: null,
      updated: null,
      data: null,
      filteredData: null,
      countries: null
    };
  },
  computed: {
    totalInfected: function() {
      if (this.data) return this.sum(this.data.map(x => +x.infected));
      else return null;
    },
    totalRecovered: function() {
      if (this.data) return this.sum(this.data.map(x => +x.recovered));
      else return null;
    },
    totalDeath: function() {
      if (this.data) return this.sum(this.data.map(x => +x.death));
      else return null;
    }
  },
  methods: {
    reload() {
      this.getData();
    },
    sum(arr) {
      return arr.reduce((prev, curr) => prev + curr, 0);
    },
    filterItems() {
      if (!this.data) return;
      let val = this.filterVal ? this.filterVal.toUpperCase() : "";

      this.filteredData = this.data.filter(item => item.name.startsWith(val));
    },
    parseHtml(html) {
      return html
        .match(/<td(|\s+[^>]*)>(.*?)<\/td\s*>/g)
        .map(m => m.replace(/<\/?[^>]+(>|$)/g, ""));
    },
    rate(recovered, death) {
      let sum = +recovered + +death;
      return ((+death * 100) / sum).toFixed(2);
    },
    toJson(arr) {
      // let date = arr[0];

      let result = [];
      for (let i = 6; i < arr.length; i++) {
        result.push({
          name: arr[i],
          infected: arr[++i],
          death: arr[++i],
          recovered: arr[++i]
        });
        ++i;
      }
      return result.sort(this.compare);
      // return result;
    },
    compare(a, b) {
      if (+a.infected < +b.infected) {
        return 1;
      }
      if (+a.infected > +b.infected) {
        return -1;
      }
      return 0;
    },
    getData() {
      this.axios
        .get(
          "https://docs.google.com/spreadsheets/d/e/2PACX-1vQuDj0R6K85sdtI8I-Tc7RCx8CnIxKUQue0TCUdrFOKDw9G3JRtGhl64laDd3apApEvIJTdPFJ9fEUL/pubhtml?gid=0&single=true"
        )
        .then(x => {
          let html = this.parseHtml(x.data);
          let json = this.toJson(html);

          this.updated = html[0];
          console.log("loaded");

          this.data = json.filter(item => item.name && item.name.length > 0);
          this.filteredData = this.data;
          this.filterItems();
        });
    }
  },
  filters: {
    getLocale: function(value) {
      if (!value) return "";
      if (countries[value]) {
        return countries[value].toLowerCase();
      }
      value = value.toString().toLowerCase();
      value = value
        .split(" ")
        .map(value => value.charAt(0).toUpperCase() + value.slice(1))
        .join(" ");

      if (countries[value]) {
        return countries[value].toLowerCase();
      }
      return "git";
    }
  },
  mounted() {
    this.getData();

    setInterval(() => {
      this.getData();
    }, 60000);
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.stats {
  @apply flex flex-row justify-between px-4 py-2 rounded font-semibold bg-gray-800 text-white;

  &--info {
    font-size: 9px;
  }
}

.board {
  max-width: 500px;
  margin: auto;
}
</style>

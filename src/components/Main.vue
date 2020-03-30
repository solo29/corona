<template>
  <div class="text-gray-500">
    <div class="text-center" @click="reload">{{updated}}</div>
    <div>
      <table class="mx-auto table-auto text-center text-xs">
        <thead>
          <tr>
            <th class="sm:px-4 py-2">Total Infected</th>
            <th class="sm:px-4 py-2">Total Death</th>
            <th class="sm:px-4 py-2">Total Recovered</th>
            <th class="sm:px-4 py-2">Total Death Rate</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td class="border sm:px-4 py-2">
              <Number :number="totalInfected" />
            </td>
            <td class="border sm:px-4 py-2">
              <Number :number="totalDeath" />
            </td>
            <td class="border sm:px-4 py-2">
              <Number :number="totalRecovered" />
            </td>
            <td class="border sm:px-4 py-2">{{rate(totalRecovered, totalDeath)}}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="px-2 mt-2">
      <input
        class="uppercase text-center bg-white focus:outline-none focus:shadow-outline border border-gray-300 rounded-lg py-2 px-4 block w-full appearance-none leading-normal"
        type="text"
        @keyup="filterItems"
        v-model="filterVal"
      />
    </div>
    <table class="mx-auto table-fixed text-center text-xs">
      <thead>
        <tr>
          <th class="sm:px-4 py-2">Countries</th>
          <th class="sm:px-4 py-2">Infected</th>
          <th class="sm:px-4 py-2">Death</th>
          <th class="sm:px-4 py-2">Recovered</th>
          <th class="sm:px-4 py-2">Rate</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in filteredData" :key="item.name">
          <td class="w-1/4 border sm:px-4 py-2">{{item.name}}</td>
          <td class="w-1/4 border sm:px-4 py-2">
            <Number :number="item.infected" />
          </td>
          <td class="w-1/4 border sm:px-4 py-2">
            <Number :number="item.death" />
          </td>
          <td class="w-1/4 border sm:px-4 py-2">
            <Number :number="item.death" />
          </td>
          <td class="w-1/4 border sm:px-4 py-2">{{rate(item.recovered, item.death) }}%</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import Number from "./Number.vue";
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
      filteredData: null
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

          this.data = json;
          this.filteredData = this.data;
          this.filterItems();
        });
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
</style>

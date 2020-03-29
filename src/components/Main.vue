<template>
  <div>
    <div class="text-center">{{updated}}</div>
    <div class="px-2">
      <input
        class="bg-white focus:outline-none focus:shadow-outline border border-gray-300 rounded-lg py-2 px-4 block w-full appearance-none leading-normal"
        type="text"
        @keyup="filterItems"
      />
    </div>
    <table class="mx-auto table-auto text-center">
      <thead>
        <tr>
          <th class="px-4 py-2">Countries</th>
          <th class="px-4 py-2">Infected</th>
          <th class="px-4 py-2">Death</th>
          <th class="px-4 py-2">Recovered</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in filteredData" :key="item.name">
          <td class="border px-4 py-2">{{item.name}}</td>
          <td class="border px-4 py-2">{{item.infected}}</td>
          <td class="border px-4 py-2">{{item.death}}</td>
          <td class="border px-4 py-2">{{item.recovered}}</td>
        </tr>
      </tbody>
    </table>
    <!-- <ul>
      <li v-for="item in filteredData" :key="item.name">{{item}}</li>
    </ul>-->
  </div>
</template>

<script>
export default {
  name: "Main",

  data() {
    return {
      updated: null,
      data: null,
      filteredData: null
    };
  },
  methods: {
    filterItems(e) {
      if (!this.data || !e.target) return;
      let val = e.target.value.toUpperCase();
      console.log(val);
      this.filteredData = this.data.filter(item => item.name.startsWith(val));
    },
    parseHtml(html) {
      return html
        .match(/<td(|\s+[^>]*)>(.*?)<\/td\s*>/g)
        .map(m => m.replace(/<\/?[^>]+(>|$)/g, ""));
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
    }
  },
  mounted() {
    this.axios
      .get(
        "https://docs.google.com/spreadsheets/d/e/2PACX-1vQuDj0R6K85sdtI8I-Tc7RCx8CnIxKUQue0TCUdrFOKDw9G3JRtGhl64laDd3apApEvIJTdPFJ9fEUL/pubhtml?gid=0&single=true"
      )
      .then(x => {
        let html = this.parseHtml(x.data);
        let json = this.toJson(html);

        this.updated = html[0];
        console.log(json);

        this.data = json;
        this.filteredData = json;
      });
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>

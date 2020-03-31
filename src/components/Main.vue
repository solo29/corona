<template>
  <div class="text-gray-500">
    <div class="time">Update {{time}}s ago</div>

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

    <div class="w-1/2 mx-auto px-2 mt-3 flex">
      <button class="reload" @click="reload">
        <svg
          :class="loading ? 'loading': ''"
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 24 24"
          fill="white"
          width="58px"
          height="58px"
        >
          <path
            d="M17.65 6.35C16.2 4.9 14.21 4 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08c-.82 2.33-3.04 4-5.65 4-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"
          />
          <path d="M0 0h24v24H0z" fill="none" />
        </svg>
      </button>
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
          height="20px"
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
      loading: true,
      filterVal: null,
      updated: null,
      data: null,
      filteredData: null,
      countries: null,
      time: 0
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
      // alert(22);
      if (!this.loading) this.getData();
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
      this.loading = true;
      this.axios.defaults.headers["Cache-Control"] = "no-store";
      this.axios

        .get(
          "https://docs.google.com/spreadsheets/d/e/2PACX-1vQuDj0R6K85sdtI8I-Tc7RCx8CnIxKUQue0TCUdrFOKDw9G3JRtGhl64laDd3apApEvIJTdPFJ9fEUL/pubhtml?gid=0&single=true&nocache=" +
            new Date().getTime()
        )
        .then(x => {
          let html = this.parseHtml(x.data);
          let json = this.toJson(html);

          this.updated = new Date().toTimeString();
          console.log("...loading");

          this.data = json.filter(item => item.name && item.name.length > 0);
          this.filteredData = this.data;
          this.filterItems();
          this.loading = false;
          this.time = 0;
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
      this.time++;
      if (this.time >= 30) {
        this.getData();
        this.time = 0;
      }
    }, 1000);
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
  img {
    width: 30px;
    height: 25px;
  }
}
.time {
  font-size: 9px;
}
.reload {
  position: absolute;
  margin: 0;
  margin-right: 13px;
  right: 0;
  margin-top: -8px;
  // svg {
  //   transform: rotate(360deg);
  //   transition: all ease 2s;
  // }

  .loading {
    -webkit-animation: rotation 2s infinite linear;
  }

  @-webkit-keyframes rotation {
    from {
      -webkit-transform: rotate(0deg);
    }
    to {
      -webkit-transform: rotate(359deg);
    }
  }
}
</style>

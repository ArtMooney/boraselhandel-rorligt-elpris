<style scoped>
@import "./css/normalize.css";
@import "./css/components.css";
@import "./css/rorligt-elpris.css";
</style>

<template>
  <div class="rorligt-elpris-app">
    <div class="rorligt-elpris-wrapper">
      <div
        :class="[elomrade === 0 ? 'elomrade left selected' : 'elomrade left']"
      >
        <div class="elpris-rubrik">Elområde 1</div>
        <div class="elpris-wrapper">
          <div class="rorligt-elpris">{{ pricesArea1.priceAverage }}</div>
          <div class="elpris-kwh">öre/kWh</div>
        </div>
      </div>
      <div :class="[elomrade === 1 ? 'elomrade selected' : 'elomrade']">
        <div class="elpris-rubrik">Elområde 2</div>
        <div class="elpris-wrapper">
          <div class="rorligt-elpris">{{ pricesArea2.priceAverage }}</div>
          <div class="elpris-kwh">öre/kWh</div>
        </div>
      </div>
      <div :class="[elomrade === 2 ? 'elomrade selected' : 'elomrade']">
        <div class="elpris-rubrik">Elområde 3</div>
        <div class="elpris-wrapper">
          <div class="rorligt-elpris">{{ pricesArea3.priceAverage }}</div>
          <div class="elpris-kwh">öre/kWh</div>
        </div>
      </div>
      <div
        :class="[elomrade === 3 ? 'elomrade right selected' : 'elomrade right']"
      >
        <div class="elpris-rubrik">Elområde 4</div>
        <div class="elpris-wrapper">
          <div class="rorligt-elpris">{{ pricesArea4.priceAverage }}</div>
          <div class="elpris-kwh">öre/kWh</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",

  data() {
    return {
      spotPeriodWebhook: "https://api.ngine.se/webhook/elhandel-getareas",
      userName: "BorasElhandel",
      userPass: "kjzsdnfkjdsf87324",
      elomrade: null,
      spotPeriodPrices: [],
      pricesArea1: "",
      pricesArea2: "",
      pricesArea3: "",
      pricesArea4: "",
    };
  },

  async created() {
    this.spotPeriodPrices = await this.getApiData(this.spotPeriodWebhook);
    this.processPrices();
  },

  methods: {
    getApiData(urlEndpoint) {
      return new Promise((resolve, reject) => {
        var requestOptions = {
          method: "GET",
          headers: {
            Authorization: "Basic " + btoa(this.userName + ":" + this.userPass),
          },
          redirect: "follow",
        };

        fetch(urlEndpoint, requestOptions)
          .then((response) => {
            if (!response.ok) throw new Error();
            return response.json();
          })
          .then((result) => {
            resolve(result);
          })
          .catch((error) => {
            console.log(error);
            reject(error);
          });
      });
    },

    processPrices() {
      for (const area of this.spotPeriodPrices) {
        if (area.searchArea === "SE1") this.pricesArea1 = area;
        if (area.searchArea === "SE2") this.pricesArea2 = area;
        if (area.searchArea === "SE3") this.pricesArea3 = area;
        if (area.searchArea === "SE4") this.pricesArea4 = area;
      }
    },
  },
};
</script>

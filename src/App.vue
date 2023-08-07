<style scoped>
@import "./css/normalize.css";
@import "./css/components.css";
@import "./css/rorligt-elpris.css";
</style>

<template>
  <div class="rorligt-elpris-app">
    <div class="postal-form w-form">
      <form class="postal-wrapper">
        <input
          v-model="postalInput"
          type="number"
          class="postal-input w-input"
          maxlength="256"
          placeholder="Postnummer"
        />
        <p v-if="showErrorMessage" class="error-message">{{ errorMessage }}</p>
      </form>
    </div>

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
      elomraden: {
        e0: "91534-98499",
        e1: "79023-79699,80596-80598,81192-81195,81630-81631,81690-81695,81730-81730,81740-81740,81791-81794,82010-82011,82020-82029,82040-82043,82046-82046,82050-82051,82060-82065,82070-82078,82130-82136,82140-82143,82150-82151,82158-82158,82191-82195,82198-82198,82230-82231,82240-82240,82291-82292,82299-82299,82330-82330,82391-82393,82430-82434,82440-82443,82450-82452,82491-82494,82530-82532,82591-82596,82630-82640,82650-82650,82660-82662,82670-82670,82691-82695,82730-82740,82791-82795,82830-82891,82894-82894,82869-84079,84081-84084,84086-84093,84095-84298,84300-89693,90130-91532,91597-92275,92278-92399,93027-93592",
        e2: "11115-19793,33230-33236,41101-51260,51264-51463,51492-57063,57131-57197,57331-57495,57531-57895,58128-59091,59094-79022,79024-79069,79071-79089,79092-79497,80250-81629,81632-81695,81791-81791,81830-81834,81841-81843,81891-81892,81930-81966,82892-82892,82895-82895,84080-84080,84085-84085,84094-84094,84299-84299",
        e3: "20522-20522,21112-39591,57072-57091,56013-56013,56025-56025,56291-56291,56891-57012,57015-57016,57230-57296,57474-57492,57496-57498,57692-57692,57738-57738,57791-57791,57793-57793,57930-57993",
      },
      errorMessage: "Error",
      showErrorMessage: false,
      postalInput: null,
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

    getElomrade(postal) {
      const elomraden = this.getFormattedAreas();
      let postalArea = null;

      for (const area of Object.entries(elomraden)) {
        for (const postalRange of area[1]) {
          if (postal >= postalRange[0]) {
            if (postal <= postalRange[1]) {
              postalArea = area[0];
            }
          }
        }
      }

      return postalArea ? parseInt(postalArea.replace("e", "")) : null;
    },

    getFormattedAreas() {
      let formattedAreas = {};

      for (const omrade of Object.entries(this.elomraden)) {
        let elomradeArray = [];
        const areaList = omrade[1].split(",");

        for (const areaRanges of areaList) {
          const areaRange = areaRanges.split("-");
          let newAreaRange = [];

          for (const area of areaRange) {
            newAreaRange.push(parseInt(area));
          }

          elomradeArray.push(newAreaRange.sort((a, b) => a - b));
        }

        formattedAreas[omrade[0]] = elomradeArray;
      }

      return formattedAreas;
    },
  },

  watch: {
    postalInput() {
      this.elomrade = this.getElomrade(this.postalInput);
    },
  },
};
</script>

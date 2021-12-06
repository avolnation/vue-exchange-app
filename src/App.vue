<template>
  <div id="parent">
    <div id="header">
      <button
        type="button"
        class="btn"
        @click="showModal"
        title="Список доступных валют"
      >
        <img
          src="https://img.icons8.com/material-outlined/48/000000/info.png"
        />
      </button>
    </div>
    <div id="info">
      <h2>Welcome to the Currency Exchange app</h2>
      <p class="steps">
        1. At first, click on info button at the top left to see list of
        available currencies.
      </p>
      <p class="steps">2. Choose From currency and To currency</p>
      <p class="steps">3. Write amount which you want to exchange</p>
    </div>

    <AvailableCurrenciesModal
      v-show="isModalVisible"
      @close="closeModal"
      :exchangeData="exchangeData"
    />

    <div id="CurrenciesExchangeWrapper">
      <p class="currex">
        <img
          class="coinExchange"
          src="./dollar-euro-money-exchange-svgrepo-com.svg"
          alt="dollar-euro-money-exchange-svgrepo-com"
        />
        Convert
      </p>
      <hr />
      <div id="currenciesInputs">
        <div class="column">
          <label>Amount</label>
          <input
            class="currexBlock"
            type="number"
			min="0"
            default=" "
            v-model="currAmount"
          />
        </div>
        <div class="column">
          <label>From</label>
          <SelectCurrencies
            id="firstSelector"
            :exchangeData="exchangeData"
            @sendData="pushAbbrData"
          />
        </div>
        <button @click="swapCurrencies" class="swapButton">Swap</button>
        <div class="column">
          <label>To</label>
          <SelectCurrencies
            id="secondSelector"
            :exchangeData="exchangeData"
            @sendData="pushAbbrData"
          />
        </div>
      </div>
      <hr />
      <transition name="fade">
        <div id="converted">
          <p
            v-if="
              currAmount == 0 || currAmount < 0 || firstCurrAbbr == '' || secondCurrAbbr == ''
            "
            class="infoo"
          >
            Choose From and To Currency, and write amount(>0)
          </p>
          <template v-else>
            <p id="CurrAmount1">{{ currAmount }} {{ firstCurrAbbr }} =</p>
            <p id="CurrAmount2">
              {{ (tweenedNumber * convRate).toFixed(2) }}
              {{ secondCurrAbbr }}
            </p>
          </template>
        </div>
      </transition>
    </div>
  </div>
</template>

<script>
import { gsap } from "gsap";
import axios from "axios";
import AvailableCurrenciesModal from "./components/AvailableCurrenciesModal.vue";
import SelectCurrencies from "./components/SelectCurrencies.vue";

export default {
  //* передаваемые properties для child элемента AvailableCurrenciesModal
  props: {
    selectedValue: String,
  },
  components: {
    AvailableCurrenciesModal,
    SelectCurrencies,
  },
  watch: {
    currAmount: function (newValue) {
      gsap.to(this.$data, { duration: 0.5, tweenedNumber: newValue });
    },
  },
  mounted() {
    //* Наблюдение за изменением валют в Select'ах
    this.$watch(
      (vm) => [vm.firstCurrAbbr, vm.secondCurrAbbr],
      (val) => {
        if (val[0] && val[1]) {
          this.pairConversion(val[0], val[1]);
        }
      }
    );
  },
  data() {
    return {
      exchangeData: "",
      isModalVisible: false,
      showPopup: false,
      firstCurrAbbr: "",
      secondCurrAbbr: "",
      convRate: null,
      currAmount: 0,
      tweenedNumber: "",
    };
  },
  //* Проверка наличия информации от API в LocalStorage, если есть, читаем в свойство exchangeData, нету => получаем от API
  created() {
    const anyDataInLocalStorage = JSON.parse(localStorage.getItem("exchanges"));

    if (anyDataInLocalStorage) {
      this.exchangeData = anyDataInLocalStorage.supportedCodes;
    } else {
      this.writeToLS();
    }
  },
  methods: {
    //* Получение кодов(Abbreviations) для дальнейшего запроса к API с целью получить convRate(Conversion Rate)
    pushAbbrData(item) {
      if (item.id == "firstSelector") {
        this.firstCurrAbbr = item.abbr;
      } else {
        this.secondCurrAbbr = item.abbr;
      }
    },
    //* Чтение данных из LocalStorage
    readFromLS() {
      let datas = JSON.parse(localStorage.getItem("exchanges"));
      if (typeof datas != "undefined" && datas !== null) {
        console.log("Theres some data in localstorage");
        this.exchangeData = datas.supportedCodes;
      } else {
        console.log("No data");
      }
    },
    async pairConversion(firstItem, secondItem) {
      let retrievedData = await this.getData(
        `/pair/${firstItem}/${secondItem}`
      );
      this.convRate = retrievedData.conversion_rate;
    },
    //*Обращение к API
    getData(methods) {
      var getLine = `https://v6.exchangerate-api.com/v6/${process.env.VUE_APP_APIKEY}/${methods}`;
      return axios.get(getLine).then((response) => response.data);
    },
    //* Получаем массив объектов supportedCodes от API + последние conversion_rates
    async getExchangeData() {
      let dataArray = {
        supportedCodes: {},
        latest: {},
      };

      await this.getData("codes").then(async (response) => {
        response.supported_codes.forEach(
          (element, index) =>
            (dataArray.supportedCodes[index] = {
              currencyAbbr: element[0],
              currencyName: element[1],
            })
        );
      });
      await this.getData("latest/USD").then(async (response) => {
        dataArray.latest = response.conversion_rates;
      });

      return dataArray;
    },
    //* Запись данных в LocalStorage
    async writeToLS() {
      let datas = await this.getExchangeData();

      localStorage.setItem("exchanges", JSON.stringify(datas));
      let retrievedData = JSON.parse(localStorage.getItem("exchanges"));

      this.exchangeData = retrievedData.supportedCodes;

      console.log("Парсим", retrievedData);
    },
    showModal() {
      this.isModalVisible = true;
    },
    closeModal() {
      this.isModalVisible = false;
    },
    swapCurrencies() {
      let firstSel = document.getElementById("firstSelector").__vue__.$data.selected;
      let secondSel = document.getElementById("secondSelector").__vue__.$data.selected;
      let first = {
        id: "firstSelector",
        abbr: firstSel,
      }
      let second = {
        id: "secondSelector",
        abbr: secondSel,
      }
      let buff = firstSel;
      document.getElementById("firstSelector").__vue__.$data.selected = secondSel;
      document.getElementById("secondSelector").__vue__.$data.selected = buff;
      this.pushAbbrData(first);
      this.pushAbbrData(second);
      console.log(firstSel, secondSel);
    },
  },
};
</script>
<style>
.steps{
  font-size: 1.3em;
}
#converted {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.column {
  min-width: 70px;
}
#CurrAmount1 {
  margin: 0;
  font-size: 1.3em;
  margin-left: 20px;
}
#CurrAmount2 {
  margin: 0;
  font-size: 1.6em;
  margin-left: 20px;
}
.infoo {
  display: flex;
  justify-content: center;
}
.currex {
  display: flex;
  justify-content: center;
  font-size: 1.5em;
  margin: 0;
}
.currexBlock {
  font-size: 1.2em;
  font-family: Google Sans;
  font-style: initial;
  color: #424242;
  background-color: #f7e7ce;
  width: 80px;
  min-width: 80px;
  max-width: 200px;
  height: 30px;
  border-radius: 5px;
  border-style: ridge;
  border: none;
  box-shadow: rgb(0 17 51 / 30%) 0px 3px 15px;
}
.currexBlock:hover {
  cursor: pointer;
  background-color: #cfc1ab;
}
#info {
  padding: 15px;
  padding-top: 5px;
}
#header {
  padding-left: 5px;
  border-bottom-left-radius: 10px;
  border-bottom-right-radius: 10px;
  background-color: #ec8244;
  padding-top: 5px;
  min-height: 50px;
}
#parent {
  background-color: #424242;
  min-width: 600px;
  max-width: 600px;
  margin: auto;
  padding-bottom: 1px;
}
#CurrenciesExchangeWrapper {
  background-color: #ec8244;
  border-radius: 10px;
  min-height: 80px;
  padding: 10px;
  margin: 10px;
}
#currenciesInputs {
  font-size: 1.2rem;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}
.column {
  display: flex;
  flex-direction: column;
  padding: 10px;
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.appear-enter, .appear-leave-to /* .fade-leave-active до версии 2.1.8 */ {
  opacity: 0;
}
.appear-enter-active,
.appear-leave-active {
  transition: opacity 2s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
.coinExchange {
  max-width: 32px;
  padding-right: 5px;
}
</style>

<template>
  <div class="container">
    <div class="container__item">
      <div>
        <select v-model="currency.currencyTypeFrom">
          <option
            :value="c.value"
            v-for="(c, index) in currencyList"
            :key="index"
          >
            {{ c.title }}
          </option>
        </select>
      </div>
      <input @input="onCalculateFrom(currencyFrom)" v-model="currencyFrom" />
    </div>

    <div class="container__item">
      <div>
        <select v-model="currency.currencyTypeTo">
          <option
            :value="c.value"
            v-for="(c, index) in currencyList"
            :key="index"
          >
            {{ c.title }}
          </option>
        </select>
      </div>
      <input @input="onCalculateTo(currencyTo)" v-model="currencyTo" />
    </div>

    <span>Rate : ${{ parseInt(rate).toFixed(2).toLocaleString() }}</span>
  </div>
</template>


<script>
import { ref, reactive, watch } from "vue";
import axios from "axios";
export default {
  setup() {
    const currency = reactive({
      currencyTypeFrom: null,
      currencyTypeTo: null,
    });
    const currencyFrom = ref(null);
    const currencyTo = ref(null);
    const rate = ref(0);

    const currencyList = [
      {
        title: "BTC",
        value: "btc",
      },
      {
        title: "ETH",
        value: "eth",
      },
      {
        title: "USD",
        value: "usd",
      },
      {
        title: "EUR",
        value: "eur",
      },
    ];

    watch(
      () => currency,
      (newValue) => {
        if (newValue.currencyTypeFrom && newValue.currencyTypeTo) {
          onFetchCurrency();
        }
      },
      { deep: true }
    );

    const onFetchCurrency = async () => {
      const formData = new FormData();
      formData.append("currency_from", "btc");
      formData.append("currency_to", "usd");

      const response = await axios.post(
        "https://crypto-verse.info/api/calculator/exchange/calculate",
        formData,
        {
          headers: {
            "Content-Type": "multipart/form-data",
          },
          transformRequest: (formData) => formData,
        }
      );

      if (response) {
        rate.value = response.data?.data?.conversion_rate;
      }
    };

    const onCalculateFrom = (value) => {
      if (rate.value) {
        if (currencyFrom.value) {
          let rateWithPercent = parseInt(
            rate.value + 1.01 * rate.value
          ).toFixed(1);
          currencyTo.value = (rateWithPercent * parseInt(value)).toLocaleString(
            "RU"
          );
        } else {
          currencyTo.value = null;
        }
      }
    };

    const onCalculateTo = (value) => {
      if (rate.value) {
        if (currencyTo.value) {
          let rateWithPercent = parseInt(
            rate.value + 1.01 * rate.value
          ).toFixed(1);
          currencyFrom.value = parseInt(value / rateWithPercent).toLocaleString(
            "RU"
          );
        } else {
          currencyFrom.value = null;
        }
      }
    };

    return {
      currencyList,
      currencyFrom,
      currencyTo,
      currency,
      rate,
      // methods
      onFetchCurrency,
      onCalculateTo,
      onCalculateFrom,
    };
  },
};
</script>

<style>
.container__item {
  display: flex;
  gap: 20px;
  margin: 20px 0;
}
</style>

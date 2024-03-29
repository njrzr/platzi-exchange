<template>
  <div class="flex-col">
    <div class="flex justify-center">
      <bounce-loader :loading="isLoading" :color="'#68d391'" :size="100" />
    </div>
    <template v-if="!isLoading">
      <div class="flex flex-col sm:flex-row justify-around items-center">
        <div class="flex flex-col items-center">
          <img
            :src="
              `https://static.coincap.io/assets/icons/${asset.symbol.toLowerCase()}@2x.png`
            "
            class="w-20 h-20 mr-5"
            :alt="asset.name"
          />
          <h1 class="text-5xl">
            {{ asset.name }}
            <small class="sm:mr-2 text-gray-500">{{ asset.symbol }}</small>
          </h1>
        </div>

        <div class="my-10 flex flex-col">
          <ul>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Ranking</b>
              <span>#{{ asset.rank }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio actual</b>
              <span>{{ asset.priceUsd | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio más bajo</b>
              <span>{{ min | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio más alto</b>
              <span>{{ max | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio promedio</b>
              <span>{{ avg | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Variación 24hs</b>
              <span>{{ asset.changePercent24Hr | percent }}</span>
            </li>
          </ul>
        </div>

        <div class="my-10 sm:mt-0 flex flex-col justify-center text-center">
          <button
            @click="toggleConverter"
            class="bg-green-500 hover:bg-green-700 text-white font-blod py-2 px-4 rounded"
          >
            {{ fromUsd ? `USD a ${asset.symbol}` : `${asset.symbol} a USD` }}
          </button>
          <div class="flex flex-row my-5">
            <label for="convertValue" class="w-full">
              <input
                v-model="convertValue"
                type="number"
                id="convertValue"
                class="text-center bg-white focus:outline-none focus:shadow-outline border border-gray-300 rounded-lg py-2 px-4 block w-full appearance-none leading-normal"
                :placeholder="`Valor en ${fromUsd ? 'USD' : asset.symbol}`"
              />
            </label>
          </div>
          <span class="text-xl"
            >{{ convertResult }} {{ fromUsd ? asset.symbol : "USD" }}</span
          >
        </div>
      </div>
      <line-chart
        class="my-10"
        :colors="['orange']"
        :min="min"
        :max="max"
        :data="
          history.map(each => [each.date, parseFloat(each.priceUsd).toFixed(2)])
        "
      />

      <h3 class="text-xl my-10">Mejores Ofertas de Cambio</h3>
      <table>
        <tr
          v-for="mar in markets"
          :key="`${mar.exchangeId}-${mar.priceUsd}`"
          class="border-b"
        >
          <td>
            <b>{{ mar.exchangeId }}</b>
          </td>
          <td>{{ mar.priceUsd | dollar }}</td>
          <td>{{ mar.baseSymbol }} / {{ mar.quoteSymbol }}</td>
          <td>
            <Button
              :is-loading="mar.isLoading || false"
              v-if="!mar.url"
              @custom-click="getWebsite(mar)"
            >
              <slot>Obtener Link</slot>
            </Button>
            <a
              :href="mar.url"
              v-else
              class="hover:underline text-green-600"
              target="_blank"
            >
              {{ mar.url }}
            </a>
          </td>
        </tr>
      </table>
    </template>
  </div>
</template>

<script>
import api from "@/api";
import Button from "@/components/Button";

export default {
  name: "CoinDetail",
  components: { Button },
  data() {
    return {
      asset: {},
      history: [],
      markets: [],
      fromUsd: true,
      convertValue: null,
      isLoading: false
    };
  },
  computed: {
    min() {
      return Math.min(
        ...this.history.map(each => parseFloat(each.priceUsd).toFixed(2))
      );
    },
    max() {
      return Math.max(
        ...this.history.map(each => parseFloat(each.priceUsd).toFixed(2))
      );
    },
    avg() {
      return Math.abs(
        ...this.history.map(each => parseFloat(each.priceUsd).toFixed(2))
      );
    },
    convertResult() {
      if (!this.convertValue) {
        return 0;
      }

      const result = this.fromUsd
        ? this.convertValue / this.asset.priceUsd
        : this.convertValue * this.asset.priceUsd;
      return result.toFixed(4);
    }
  },
  watch: {
    $route() {
      this.getCoin();
    }
  },
  created() {
    this.getCoin();
  },
  methods: {
    getCoin() {
      const id = this.$route.params.id;
      this.isLoading = true;
      Promise.all([
        api.getAsset(id),
        api.getAssetHistory(id),
        api.getMarkets(id)
      ])
        .then(([asset, history, markets]) => {
          this.asset = asset;
          this.history = history;
          this.markets = markets;
        })
        .finally(() => (this.isLoading = false));
    },
    getWebsite(exchange) {
      this.$set(exchange, "isLoading", true);
      return api.getExchanges(exchange.exchangeId).then(res => {
        this.$set(exchange, "url", res.exchangeUrl).finally(() =>
          this.$set(exchange, "isLoading", false)
        );
      });
    },
    toggleConverter() {
      this.fromUsd = !this.fromUsd;
    }
  }
};
</script>

<style scoped>
td {
  padding: 10px;
  text-align: center;
}
</style>

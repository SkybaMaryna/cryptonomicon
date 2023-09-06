<template>
  <section>
    <div class="flex">
      <div class="max-w-xs">
        <label for="wallet" class="block text-sm font-medium text-gray-700"
          >Тикер</label
        >
        <div class="mt-1 relative rounded-md shadow-md">
          <input
            v-model="ticker"
            @keydown.enter="addTicker"
            @input="handleInput"
            type="text"
            name="wallet"
            id="wallet"
            class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
            placeholder="Например DOGE"
          />
        </div>
        <div
          v-if="ticker"
          class="flex bg-white p-1 rounded-md shadow-md flex-wrap"
        >
          <template v-for="(coin, idx) in filteredCoins" :key="coin.Id">
            <span
              v-if="idx < 4"
              @click="
                handleChoice(coin.Id);
                addTicker();
              "
              class="inline-flex items-center px-2 m-1 rounded-md text-xs font-medium bg-gray-300 text-gray-800 cursor-pointer"
            >
              {{ coin.Symbol }}
            </span>
          </template>
        </div>
        <div v-if="error" class="text-sm text-red-600">
          Такой тикер уже добавлен
        </div>
      </div>
    </div>
    <add-button
      @click="addTicker"
      type="button"
      :disabled="disabled"
      class="my-4"
    />
  </section>
</template>

<script>
import AddButton from "./AddButton.vue";

export default {
  components: {
    AddButton,
  },
  data() {
    return { ticker: "", coins: [], error: false, filteredCoins: [] };
  },
  created() {
    this.fetchData();
  },
  props: {
    disabled: {
      type: Boolean,
      required: false,
      default: false,
    },
    tickers: {
      type: Array,
      required: false,
      // default: null,
    },
  },
  emits: {
    "add-ticker": (value) => typeof value === "string" && value.length > 0,
  },
  methods: {
    addTicker() {
      const isTickerAdded = this.tickers.findIndex(
        (t) => t.name === this.ticker
      );
      if (this.tickers.length > 0 && isTickerAdded >= 0) {
        return (this.error = true);
      }
      if (this.ticker.length === 0) {
        return;
      }
      this.$emit("add-ticker", this.ticker);
      this.ticker = "";
    },
    async fetchData() {
      try {
        const res = await fetch(
          "https://min-api.cryptocompare.com/data/all/coinlist?summary=true"
        );
        const data = await res.json();
        this.coins = Object.values(data.Data);
      } catch (error) {
        console.error("Помилка при завантаженні даних:", error);
      }
    },
    handleInput() {
      this.filteredCoins = this.coins.filter((coin) =>
        coin.Symbol.toLowerCase().includes(this.ticker.toLowerCase())
      );
      if (!this.ticker) this.error = false;
    },
    handleChoice(coinId) {
      this.ticker = this.filteredCoins.find(
        (coin) => coin.Id === coinId
      ).Symbol;
    },
  },
};
</script>

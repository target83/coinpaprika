<template>
  <div>
    <h1>{{ msg }}</h1>
    <textarea v-model="text"></textarea>
    <br />
    <button @click="convert()">Przetwórz</button>
    <p v-show="isError" v-html="error" class="error"></p>
    <br /><br />
    <textarea v-model="textConverted"></textarea>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "CoinPaprika",
  data: function () {
    return {
      text: "In 1998, Wei Dai published a description of 'b-money', characterized as an anonymous, distributed electronic cash system.[Shortly thereafter, Nick Szabo described bit gold. Like {{ Name/BTC }}: {{ Valu/BTC }} and other cryptocurrencies that would follow it, bit gold (not to be confused with the later gold-based exchange, {{ Name/BITGOLD }}: {{ Value/BITGOLD }}) was described as an electronic currency system which required users to complete a proof of work function with solutions being cryptographically put together and published. A currency system based on a reusable proof of work was later created by Hal Finney who followed the work of Dai and Szabo. The first decentralized cryptocurrency, {{ Name/BTC }}: {{ Value/BTC }}, was created in 2009 by pseudonymous developer Satoshi Nakamoto. It used SHA-256, a cryptographic hash function, as its proof-of-work scheme. In April 2011, {{ Name/NMC }}: {{ Value/NMC }} was created as an attempt at forming a decentralized DNS, which would make internet censorship very difficult. Soon after, in October 2011, {{ Name/LTC }}: {{ Value/LTC }} was released. It was the first successful cryptocurrency to use scrypt as its hash function instead of SHA-256. Another notable cryptocurrency, {{ Name/PPC }}: {{ Value/PPC }} was the first to use a proof-of-work/proof-of-stake hybrid.",
      textConverted: "",
      coins: [],
      error: "",
      isError: false,
    };
  },
  props: {
    msg: String,
  },
  methods: {
    convert() {
      const prefix = "{{ ";
      const symbolPrefix = "/";
      const suffix = " }}";
      const url = "https://api.coinpaprika.com/v1/coins";
      this.isError = false;
      this.error = "";
      this.textConverted = this.text;

      this.getData(url, async (response, noCache) => {
        if (!noCache) {
          this.coins = response.data;
          this.addToStorage(url, response.data);
        } else {
          this.coins = JSON.parse(sessionStorage.getItem(url));
        }
        while (this.textConverted.indexOf(prefix) >= 0) {
          const method = this.textConverted.slice(
            this.textConverted.indexOf(prefix) + prefix.length,
            this.textConverted.indexOf(symbolPrefix)
          );
          const symbol = this.textConverted.slice(
            this.textConverted.indexOf(symbolPrefix) + 1,
            this.textConverted.indexOf(suffix)
          );
          const convertedText = await this.selectMethod(method, symbol);
          this.textConverted =
            this.textConverted.substring(
              0,
              this.textConverted.indexOf(prefix)
            ) +
            convertedText +
            this.textConverted.substring(
              this.textConverted.indexOf(suffix) + suffix.length
            );
        }
      });
    },
    selectMethod(method, symbol) {
      let value = "";
      switch (method) {
        case "Name":
          value = this.getName(symbol);
          break;
        case "Value":
          value = this.getValue(symbol);
          break;
        default:
          value = null;
          this.showError(`Niepoprawna metoda ${method}<br>`);
      }
      return value;
    },
    getName(symbol) {
      return this.coins.find((coin) => coin.symbol === symbol).name;
    },
    getValue(symbol) {
      const coin_id = this.coins.find((coin) => coin.symbol === symbol).id;
      const url = `https://api.coinpaprika.com/v1/coins/${coin_id}/ohlcv/latest`;
      return new Promise((resolve) => {
        this.getData(url, (response, noCache) => {
          if (!noCache) {
            if (response.data[0]) {
              resolve(response.data[0].close);
              this.addToStorage(url, response.data[0].close);
            } else {
              resolve(null);
              this.showError(`Brak wartości dla ${symbol}<br>`);
            }
          } else {
            resolve(JSON.parse(sessionStorage.getItem(url)));
          }
        });
      });
    },
    getData(url, callback) {
      const sessionData = sessionStorage.getItem(url);
      if (sessionData) {
        callback(JSON.parse(sessionData), true);
      } else {
        axios.get(url).then(callback);
      }
    },
    showError(msg) {
      this.isError = true;
      this.error += msg;
    },
    addToStorage(name, value) {
      sessionStorage.setItem(name, JSON.stringify(value));
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
textarea {
  width: 600px;
  height: 300px;
}
.error {
  color: red;
}
</style>

<template>
<div v-if="isPriced && fetchQueryID" style="padding-top: 10px;">
  <div>
    <b-img :src="itemImage" style="max-height:100px;"></b-img>
  </div>
  <div class="d-inline-flex p-2 bd-highlight">
    <loading loader="bars" :active.sync="isLoading" :is-full-page="false"></loading>
    <table class="table table-striped" v-if="collectionCurrency && !isLoading">
      <thead class="thead-dark">
        <tr>
          <th scope="col">前 {{ fetchResultPrice.length }} 筆價格分析
            <br>
            <b-button v-if="fetchResultPrice.length === 40" @click="morePriceAnalysis" :disabled="isCounting" size="sm" variant="outline-light">再多搜 40 筆價格</b-button>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in collectionCurrency" :key="index">
          <td :style="item.count === maxCurrencyCount ? 'color: orangered;' : ''">
            報價：{{ item.amount }} x
            <b-img :src="item.image" :alt="item.text" width=30 height=30></b-img>
            / <b>{{ item.count }}</b>筆
          </td>
        </tr>
      </tbody>
      <tfoot>
        <tr>
          <td></td>
        </tr>
      </tfoot>
    </table>
  </div>
</div>
</template>

<script>
// @ is an alias to /src
var _ = require('lodash');
var axios = require('axios');
var rateLimit = require('axios-rate-limit');
import VueLoading from 'vue-loading-overlay'

const http = rateLimit(axios.create(), {
  maxRequests: 3,
  perMilliseconds: 2400,
})

export default {
  // 接受父组件的值
  props: {
    fetchID: Array,
    fetchQueryID: String,
    fetchLength: Number,
    isPriced: Boolean,
    isCounting: Boolean,
    baseUrl: String,
  },
  components: {
    loading: VueLoading,
  },
  data() {
    return {
      fetchResult: [],
      Currency: [],
      itemImage: '',
      isLoading: false,
    }
  },
  created() {

  },
  mounted() {
    let vm = this
    this.axios.get(`https://web.poe.garena.tw/api/trade/data/static`, )
      .then((response) => { // 通貨 icon
        this.Currency = response.data.result[0].entries
      })
      .catch(function (error) {
        vm.$bvToast.toast(`error: ${error}`, {
          noCloseButton: true,
          toaster: 'toast-warning-center',
          variant: 'danger',
          autoHideDelay: 800,
          appendToast: false
        })
        console.log(error);
      })
  },
  methods: {
    hotkeyPressed() {
      this.count++
    },
    morePriceAnalysis() {
      let vm = this
      let indexLength = 8 >= this.fetchID.length ? this.fetchID.length : 8
      this.isLoading = true;
      for (let index = 4; index < indexLength; index++) {
        this.axios.get(`${this.baseUrl}/api/trade/fetch/${this.fetchID[index]}?query=${this.fetchQueryID}`)
          .then((response) => {
            let limitString = (response.headers["x-rate-limit-ip-state"]).split(",")
            let limitState = limitString[1].substring(0, limitString[1].indexOf(':'))
            this.fetchResult[index].push(response.data.result)
            this.switchLimitState(limitState)
            if (this.fetchResultLength == indexLength) {
              this.isLoading = false;
            }
          })
          .catch(function (error) {
            let limitString = (error.response.headers["x-rate-limit-ip-state"]).split(",")
            let limitState = limitString[1].slice(limitString[1].lastIndexOf(":") + 1)
            limitState = parseInt(limitState, 10)
            vm.isLoading = false;
            vm.$bvToast.toast(`Oops! 被 Server 限制發送需求了，請等待 ${limitState} 秒後再重試`, {
              noCloseButton: true,
              toaster: 'toast-warning-center',
              variant: 'danger',
              autoHideDelay: 2000,
              appendToast: false
            })
            // vm.$emit('countdown', limitState)
            console.log(error);
            return
          })
      }
    },
    switchLimitState(limitState) {
      // console.log(limitState)
      switch (limitState) {
        case '12':
          this.$emit('countdown', 4 / 1.33)
          break;
        case '13':
          this.$emit('countdown', 6 / 1.33)
          break;
        case '14':
          this.$emit('countdown', 6 / 1.33)
          break;
        case '15':
          this.$emit('countdown', 6 / 1.33)
          break;
        case '16':
          this.$emit('countdown', 6 / 1.33)
          break;
        default:
          break;
      }
    }
  },
  watch: {
    fetchID: {
      immediate: true,
      handler(val) {
        let vm = this
        let indexLength = this.fetchLength >= val.length ? val.length : this.fetchLength
        this.fetchResult = [
          [],
          [],
          [],
          [],
          [],
          [],
          [],
          []
        ]
        this.itemImage = ''
        if (this.isLoading || !this.isPriced || val.length === 0) {
          return
        }
        this.isLoading = true;
        for (let index = 0; index < indexLength; index++) {
          this.axios.get(`${this.baseUrl}/api/trade/fetch/${val[index]}?query=${this.fetchQueryID}`)
            .then((response) => {
              let limitString = (response.headers["x-rate-limit-ip-state"]).split(",")
              let limitState = limitString[1].substring(0, limitString[1].indexOf(':'))
              this.fetchResult[index].push(response.data.result)
              this.switchLimitState(limitState)
              if (this.fetchResult[0].length !== 0 && !this.itemImage) {
                this.itemImage = this.fetchResult[0][0][0].item.icon
              }
              if (this.fetchResultLength == indexLength) {
                this.isLoading = false;
              }
            })
            .catch(function (error) {
              let limitString = (error.response.headers["x-rate-limit-ip-state"]).split(",")
              let limitState = limitString[1].slice(limitString[1].lastIndexOf(":") + 1)
              limitState = parseInt(limitState, 10)
              vm.isLoading = false;
              vm.$bvToast.toast(`Oops! 被 Server 限制發送需求了，請等待 ${limitState} 秒後再重試`, {
                noCloseButton: true,
                toaster: 'toast-warning-center',
                variant: 'danger',
                autoHideDelay: 2000,
                appendToast: false
              })
              // vm.$emit('countdown', limitState)
              console.log(error);
              return
            })
        }
      }
    },
  },
  computed: {
    fetchResultPrice() { // 取出 result -> listing -> price 物件內容
      return this.fetchResult.flat(Infinity).map(item => Object.values(item)[1]).map(item => Object.values(item)[Object.values(item).length - 1]);
    },
    fetchResultLength() {
      return Math.ceil(this.fetchResultPrice.length / 10)
    },
    collectionRepeat() {
      if (!this.isPriced || !this.fetchResultPrice[0]) {
        return 0
      }
      const result = [...this.fetchResultPrice.reduce((r, e) => { // 計算相同 amount & currency 重複的次數
        let k = `${e.amount}|${e.currency}`;
        if (!r.has(k)) r.set(k, {
          ...e,
          count: 1
        })
        else r.get(k).count++
        return r;
      }, new Map).values()]

      return result
      // return [...new Set(this.fetchResultPrice.map(item => JSON.stringify(item)))].map(item => JSON.parse(item)); // 去除相同 obj
    },
    collectionCurrency() { // 增加通貨 icon
      if (!this.isPriced || !this.fetchResultPrice[0]) {
        return 0
      }
      let collectionCurrency = this.collectionRepeat
      collectionCurrency.forEach((collection, index) => {
        this.Currency.forEach(element => {
          if (collection.currency === element.id) {
            collectionCurrency[index].image = `https://web.poe.garena.tw${element.image}`
            collectionCurrency[index].text = element.text
          }
        });
      });
      return collectionCurrency
    },
    maxCurrencyCount() {
      return Math.max(...this.collectionCurrency.map(p => p.count))
    },
  },
}
</script>

<style>

</style>

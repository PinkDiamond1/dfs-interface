<template>
  <div class="notice">
    <van-notice-bar v-if="voices.length"
      class="noticeBar"
      color="#333"
      :scrollable="false"
      background="#FFF">
        <template v-slot:left-icon>
          <img class="nImg" src="https://leafy-kataifi-c6d825.netlify.app/dfs/notice_icon.png">
        </template>
        <van-swipe
          vertical
          class="notice-swipe"
          :autoplay="5000"
          :show-indicators="false"
        >
          <van-swipe-item v-for="(item, i) in voices" :key="i">{{ $t(`${item.title}`) }}</van-swipe-item>
        </van-swipe>
    </van-notice-bar>

    <div class="flexb lists">
      <van-swipe class="my-swipe" :autoplay="6000" indicator-color="#29d4b0">
        <van-swipe-item class="flexb grid" v-for="(top, index) in top3" :key="`top${index}`">
          <div class="symInfo" v-for="(v, i) in top"
            @click="handleTo(v)"
            :key="`sym-${i}`">
            <div class="coin din">
              <span>{{ v.symbol1 }}/{{ v.symbol0 }}</span>
              <span class="smallChange dinReg"
                :class="{
                  'green': parseFloat(v.priceRate) >= 0,
                  'red': parseFloat(v.priceRate) < 0,
                }">
                <span>{{ v.priceRate }}</span>
              </span>
            </div>
            <div class="num dinBold"
              :class="{
                'green': parseFloat(v.priceRate) >= 0,
                'red': parseFloat(v.priceRate) < 0,
              }">
              <span>{{ parseFloat(v.price || 0).toFixed(4) }}</span>
            </div>
            <div class="aboutPrice tip" v-if="language === 'en'">
              <span>$</span>
              <span>{{ v.aboutPriceU }}</span>
            </div>
            <div class="aboutPrice tip" v-else>
              <span>¥</span>
              <span>{{ v.aboutPriceCNY }}</span>
            </div>
          </div>
        </van-swipe-item>
      </van-swipe>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex';

export default {
  name: 'homeNotice',
  data() {
    return {
      voices: [{
        title: 'home.homeUi2',
      }, {
        title: 'home.homeUi1',
      }],
      top3: [[{}, {}, {}]],
      hotArr: [451, 17, 665]
    }
  },
  computed: {
    ...mapState({
      language: state => state.app.language,
      marketLists: state => state.config.marketLists,
      hotLists: state => state.config.hotLists, // 生产环境
    })
  },
  watch: {
    marketLists: {
      handler: function mls(newVal) {
        if (!newVal.length) {
          return
        }
        const hot = [];
        this.hotArr.forEach((mid, i) => {
          const has = newVal.filter(v => v.mid == mid && v.symbol0 === 'USDT');
          const i1 = parseInt(i / 3);
          if (!hot[i1]) {
            hot[i1] = []
          }
          hot[i1].push(...has)
        });
        this.top3 = hot;
      },
      deep: true,
      immediate: true,
    }
  },
  methods: {
    handleTo(v) {
      const query = {
        in: `${v.contract0}-${v.symbol0}`,
        out: `${v.contract1}-${v.symbol1}`,
      }
      this.$router.push({
        name: 'index',
        query,
      })
    },
  }
}
</script>

<style lang="scss" scoped>
.notice{
  padding: 0 28px 0;
  border-radius: 20px;
  background: #FFF;
  margin-bottom: 20px;
  .noticeBar{
    padding: 0 0px;
    border-bottom: 1px solid $color-border;
    font-size: 24px;
    .nImg{
      width: 32px;
      margin-right: 10px;
    }
    .notice-swipe {
      height: 32px;
      line-height: 32px;
      text-align: left;
    }
  }
  .lists{
    font-size: 24px;
    padding: 20px 0px 10px;
    text-align: left;
    .my-swipe{
      width: 100%;
      padding-bottom: 20px;
      :deep(.van-swipe__indicators){
        bottom: 5px;
      }
      :deep(.van-swipe__indicator){
        background: #999999;
      }
    }
    .symInfo{
      // max-width: 35%;
      position: relative;
      padding-bottom: 10px;
      &:last-child{
        padding-right: 0;
      }
      .smallChange{
        font-size: 20px;
        margin-left: 8px;
      }
      .num{
        font-size: 40px;
        margin: 8px 0 6px;
      }
      .red{
        color: #E54F5D;
      }
      .green{
        color: #3FB387;
      }
      .aboutPrice{
        font-size: 26px;
      }
    }
  }
  .grid{
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-gap: 30px;
  }
}
</style>

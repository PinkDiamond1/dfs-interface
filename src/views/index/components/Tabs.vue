<template>
  <div class="tabs tip flexb" :class="{
    'swap2': act === 1,
    'pddex': act === 3,
    'market': act === 2,
  }">
    <span class="" :class="{'tabAct': act === 1}" @click="handleChangeAct(1)">
      <span class="tabName flexc">{{ $t('tabbar.swap') }}</span>
    </span>
    <span class="" :class="{'tabAct': act === 3}" @click="handleChangeAct(3)">
      <span class="tabName flexc">{{ $t('pddex.pOrder') }}</span>
    </span>
    <span class="" :class="{'tabAct': act === 2}" @click="handleChangeAct(2)">
      <span class="tabName flexc">{{ $t('tab.pools') }}</span>
    </span>
  </div>
</template>

<script>
import { mapState } from 'vuex';
export default {
  name: 'tabs',
  data() {
    return {
      act: 1,
    }
  },
  computed: {
    ...mapState({
      marketLists: state => state.sys.marketLists,
    }),
  },
  mounted() {
    const routeName = this.$route.name || 'index';
    if (routeName === 'index') {
      this.act = 1;
    } else if (routeName === 'market') {
      this.act = 2;
    } else {
      this.act = 3;
    }
  },
  methods: {
    handleChangeAct(num = 1) {
      if (this.act === num) {
        return
      }
      this.act = num;
      if (this.act === 1) {
        this.$router.push({name: 'index'})
      } else if (this.act === 2) {
        const localData = localStorage.getItem('swapMarkets') ? JSON.parse(localStorage.getItem('swapMarkets')) : null;
        let mid = 17;
        if (localData) {
          const MidsLength = localData.thisMidsPath.split('-').length;
          MidsLength === 1 ? mid = localData.thisMidsPath : null;
        }
        const name = 'market'
        this.$router.push({name, params: {mid}})
      } else if (this.act === 3) {
        // usdxusdxusdx-USDC-tethertether-USDT
        let symbol = 'eosio.token-EOS-tethertether-USDT'
        const localData = localStorage.getItem('swapMarkets') ? JSON.parse(localStorage.getItem('swapMarkets')) : null;
        let mid = 17;
        if (localData) {
          const MidsLength = localData.thisMidsPath.split('-').length;
          MidsLength === 1 ? mid = localData.thisMidsPath : null;
        }
        const market = this.marketLists.find(v => v.mid == mid)
        if (market) {
          symbol = `${market.contract1}-${market.symbol1}-${market.contract0}-${market.symbol0}`
        }
        const name = 'pddexTrade'
        this.$router.push({
          name,
          params: {
            symbol,
          }
        })
      } else {
        this.$router.push({name: 'bank'})
      }
    },
  },
}
</script>

<style lang="scss" scoped>
.tabs{
  font-size: 33px;
  // width: 690px;
  height: 106px;
  &.swap2{
    background-image: url('https://leafy-kataifi-c6d825.netlify.app/icon/swap.png');
    background-size: 101%;
    background-repeat: no-repeat;
  }
  &.pddex{
    background-image: url('https://leafy-kataifi-c6d825.netlify.app/icon/pddex.png');
    background-size: 101%;
    background-repeat: no-repeat;
    background-position: 0px -1px;
  }
  &.market{
    background-image: url('https://leafy-kataifi-c6d825.netlify.app/icon/market.png');
    background-size: 101%;
    background-repeat: no-repeat;
    background-position: -1px 0;
  }
  &>span{
    flex: 1;
    height: 100%;
    text-align: center;
    color: #333333;
    .tabName{
      height: 96px;
    }
  }
  .tabAct{
    color: #29D4B0;
    font-weight: 500;
    position: relative;
    background-image: none !important;
    &::before{
      content: '';
      background-image: url('https://storied-crepe-e5e65c.netlify.app/svg/checked.svg');
      background-repeat: no-repeat;
      background-size: 100%;
      position: absolute;
      width:68px;
      height:12px;
      border-radius:0px 0px 6px 6px;
      bottom: 16px;
      left: 50%;
      transform: translateX(-50%);
    }
  }
}
</style>
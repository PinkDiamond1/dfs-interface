<template>
  <div class="markets">
    <div class="marketMain">
      <Tabs />
      <div class="infoData">
        <!-- 交易对信息 -->
        <div class="symbolInfo flexb" @click="showMarketList = true">
          <div class="flexa">
            <img class="coinImg" :onerror="$errorImg" :src="thisMarket.sym0Data.imgUrl" >
            <div>
              <div class="flexa">
                <span>{{ thisMarket.symbol0 }} <i class="el-icon-arrow-down"></i></span>
              </div>
              <div class="tip">{{ thisMarket.contract0 }}</div>
            </div>
          </div>
          <img class="addImg" src="https://storied-crepe-e5e65c.netlify.app/svg/add.svg">
          <div class="flexa">
            <img class="coinImg" :onerror="$errorImg" :src="thisMarket.sym1Data.imgUrl" >
            <div>
              <div class="flexa">
                <span>{{ thisMarket.symbol1 }} <i class="el-icon-arrow-down"></i></span>
              </div>
              <div class="tip">{{ thisMarket.contract1 }}</div>
            </div>
          </div>
        </div>
        <!-- 矿池信息 -->
        <div class="poolsInfo">
          <div class="item">
            <div class="subTitle tip">{{ $t('more.about24Apy') }}</div>
            <div class="num">
              <span class="din">{{ countApy }}%</span>
              <span class="green_p" @click.stop="showApyDetail = true">{{ $t('public.detail') }}></span>
            </div>
          </div>
          <div class="item">
            <div class="subTitle tip">{{ $t('dex.exchangePrice') }}</div>
            <div class="num flexa din" @click="exRate = !exRate">
              <span v-if="!exRate">1{{ thisMarket.symbol0 }} = {{ thisMarket.sym0Rate || '-' }}{{ thisMarket.symbol1 }}</span>
              <span v-else>1{{ thisMarket.symbol1 }} = {{ thisMarket.sym1Rate || '-' }}{{ thisMarket.symbol0 }}</span>
              <img class="iconImg" v-if="!exRate" src="https://leafy-kataifi-c6d825.netlify.app/icon/price_switch_icon_btn_left.svg" alt="">
              <img class="iconImg" v-else src="https://leafy-kataifi-c6d825.netlify.app/icon/price_switch_icon_btn_right.svg" alt="">
            </div>
          </div>
          <div class="item">
            <div class="subTitle tip">
              <span>{{ $t('dex.poolNum') }}</span>
              <span class="green_p" @click="handleToPool">{{ $t('pools.toPool') }}></span>
            </div>
            <div class="num din">
              <span>{{ thisMarket.reserve0 }} / {{ thisMarket.reserve1 }}</span>
            </div>
          </div>
          <div class="tip flexa" @click="showAboutMarket = true">
            <span>{{ $t('pools.spcTip') }}</span>
            <img class="iconImg" src="https://storied-crepe-e5e65c.netlify.app/icon/tips_icon_btn.svg" alt="">
          </div>
        </div>
        <!-- 按钮事件 -->
        <div class="btnDiv">
          <div class="btn flexc" @click="showAdd = true">{{ $t('invi.join') }}</div>
        </div>
        <!-- 跳转连接 -->
        <div class="linkTo flexb">
          <span class="flexc">
            <img src="https://storied-crepe-e5e65c.netlify.app/svg/create_icon.svg">
            <span @click="handleTo('createMarket')">{{ $t('dex.addMarket') }}</span>
            <i class="el-icon-arrow-right"></i>
          </span>
          <span class="flexc">
            <img src="https://storied-crepe-e5e65c.netlify.app/svg/record.svg">
            <span @click="handleTo('MarketHis')">{{ $t('more.mkHis') }}</span>
            <i class="el-icon-arrow-right"></i>
          </span>
        </div>
      </div>
    </div>

    <MyMarketLists :thisMarket="thisMarket"/>

    <!-- 弹窗组件 -->
    <van-popup
      class="newMarket"
      v-model="showMarketList"
      position="left">
      <market-list :marketLists="marketLists"
        @listenMarketChange="handleMarketChange"
        @listenClose="handleClose"/>
    </van-popup>
    <!-- 年化详情 -->
    <el-dialog
      class="myDialog apy"
      :visible.sync="showApyDetail">
      <MarketApy :countApy="countApy" :isActual="true"
                 :aprInfo="aprInfo"/>
    </el-dialog>
    <!-- 加入做市 -->
    <el-dialog
      class="mkListDia"
      :show-close="false"
      :visible.sync="showAdd">
      <AddMarket v-if="showAdd"
        :thisMarket="thisMarket"
        @listenClose="handleClose"/>
    </el-dialog>
    <!-- 取回做市 -->
    <el-dialog
      class="mkListDia"
      :show-close="false"
      :visible.sync="showRemove">
      <Withdraw v-if="showRemove"
        :thisMarket="thisMarket"
        @listenClose="handleClose"/>
    </el-dialog>
    <!-- 关于做市 -->
    <el-dialog
      class="myDialog apy"
      :show-close="false"
      :visible.sync="showAboutMarket">
      <AboutMarket />
    </el-dialog>
  </div>
</template>

<script>
import { mapState } from 'vuex';
import MarketList from '@/components/MarketArea';
import Tabs from '../index/components/Tabs';
import MarketApy from './popup/MarketApy'
import AddMarket from './popup/AddMarket'
import MyMarketLists from './comp/MarketLists'
import AboutMarket from './popup/AboutMarket'
import Withdraw from './comp/Withdraw'

// 公用方法
import { getYfcReward,
  toFixed, accDiv } from '@/utils/public';

export default {
  name: 'market',
  components: {
    Tabs,
    MarketList,
    MarketApy,
    AddMarket,
    MyMarketLists,
    AboutMarket,
    Withdraw,
  },
  data() {
    return {
      first: true,
      exRate: true,
      showMarketList: false,
      showApyDetail: false,
      showAdd: false,
      showRemove: false,
      showAboutMarket: false,
      thisMarket: {
        mid: 39,
        symbol0: 'EOS',
        contract0: 'eosio.token',
        symbol1: 'DFS',
        contract1: 'minedfstoken',
        sym0Data:{
          imgUrl: 'https://apps.defis.network/static/coin/eosio.token-eos.svg'
        },
        sym1Data:{
          imgUrl: 'https://apps.defis.network/static/coin/minedfstoken-dfs.png'
        }
      },
      lpApy: {},
      token: '0',
      countApy: '0.00',
      aprInfo: {},
    }
  },
  computed: {
    ...mapState({
      account: state => state.app.account,
      baseConfig: state => state.sys.baseConfig,
      marketLists: state => state.sys.marketLists,
      marketLists2: state => state.config.marketLists,
    }),
  },
  watch: {
    marketLists: {
      handler: function ml(newVal) {
        if (!newVal.length) {
          return
        }
        if (this.first) {
          this.thisMarket.mid = this.$route.params.mid || 39;
        }
        this.first = false;
        const thisMarket = newVal.find(v => v.mid === Number(this.thisMarket.mid)) || newVal[0];
        // const thisMarket = newVal[0];
        const reserve0 = thisMarket.reserve0.split(' ')[0];
        const reserve1 = thisMarket.reserve1.split(' ')[0];
        thisMarket.sym0Rate = toFixed(accDiv(reserve1, reserve0), thisMarket.decimal1)
        thisMarket.sym1Rate = toFixed(accDiv(reserve0, reserve1), thisMarket.decimal0)
        this.thisMarket = thisMarket;
        this.handleBeforeDestroy()
      },
      deep: true,
      immediate: true
    },
    thisMarket: {
      handler: function tm(newVal, oldVal) {
        if (!newVal || (oldVal && oldVal.mid === newVal.mid)) {
          return
        }
        this.handleGetApy()
      },
      deep: true,
      immediate: true
    },
    account: {
      handler: function listen(newVal) {
        if (newVal.name) {
          this.handleGetAccToken();
        }
      },
      deep: true,
      immediate: true,
    },
  },
  methods: {
    async handleGetApy() {
      const info = this.marketLists2.find(v => v.mid == this.$route.params.mid) || {}
      this.countApy = parseFloat(info.apy || 0).toFixed(2);
      this.aprInfo = info.apy_detail
      // const params = {
      //   mid: this.thisMarket.mid
      // };
      // const {status, result} = await this.$api.get_market_info(params)
      // if (!status) {
      //   return
      // }
      // console.log('handleGetApy', result)
      // this.countApy = parseFloat(result.apy || 0).toFixed(2);
      // this.aprInfo = result.apy_detail
    },
    handleClose() {
      this.showMarketList = false;
      this.showAdd = false;
      this.showRemove = false;
    },
    handleShowAdd(item) {
      this.menageMarket = item;
      this.showAdd = true;
    },
    handleShowRemove(item) {
      this.menageMarket = item;
      this.showRemove = true;
    },
    handleMarketChange(data) {
      this.thisMarket = data;
      this.showMarketList = false;
      this.token = '0';

      this.handleGetAccToken();
      this.handleBeforeDestroy();
    },
    handleToPool() {
      let name = 'dfsMinePool'
      const poolNames = this.$store.state.config.poolNames;
      Object.keys(poolNames).forEach(key => {
        const status = poolNames[key].includes(Number(this.thisMarket.mid))
        if (status) {
          name = key;
        }
      })
      this.$router.push({
        name,
        params: {
          mid: this.thisMarket.mid,
          type: 'lp',
          sym: this.thisMarket.mid,
        }
      })
    },
    handleTo(name) {
      this.$router.push({
        name,
        params: {
          mid: this.thisMarket.mid
        }
      })
    },
    handleDealApy(mid = 329, project) {
      let yfcReward = getYfcReward(this.thisMarket.mid, 'year', project);
      if (Number(yfcReward)) {
        const YfcPool = this.marketLists.find(vv => vv.mid === mid) || {};
        const price = parseFloat(YfcPool.reserve0 || 0) / parseFloat(YfcPool.reserve1 || 1)
        yfcReward = yfcReward * price / 20000 * 100;
        yfcReward = yfcReward.toFixed(2);
      }
      return yfcReward || '0.00';
    },
    handleBeforeDestroy() {
      const localData = localStorage.getItem('swapMarkets') ? JSON.parse(localStorage.getItem('swapMarkets')) : null;
      if (localData && localData.thisMidsPath == this.thisMarket.mid) {
        return
      }
      const swapMarkets = {
        thisMarket0: this.thisMarket.sym0Data,
        thisMarket1: this.thisMarket.sym1Data,
        thisCoinsPath: '',
        thisMidsPath: `${this.thisMarket.mid}`,
      }
      localStorage.setItem('swapMarkets', JSON.stringify(swapMarkets))
    },
    regInit() {
      if (this.account.name && this.marketLists.length) {
        return true;
      }
      return false;
    },
    // 获取账户当前交易对凭证数量
    async handleGetAccToken() {
      if (!this.regInit()) {
        return;
      }
      const params = {
        code: this.baseConfig.toAccountSwap,
        scope: this.thisMarket.mid,
        table: 'liquidity',
        lower_bound: ` ${this.account.name}`,
        upper_bound: ` ${this.account.name}`,
        json: true
      }
      const {status, result} = await this.$api.get_table_rows(params)
      if (!status || !result.rows.length) {
        return
      }
      const list = result.rows || [];
      !list[0] ? this.token = '0' : this.token = `${list[0].token}`;
    },
  }
}
</script>

<style lang="scss" scoped>
.markets{
  padding: 30px;
}
.marketMain{
  box-shadow: 0px 2px 6px 4px rgba(243,243,243,1);
  border-radius: 12px;
  font-size: 28px;
  .symbolInfo{
    padding: 34px 30px;
    margin: 5px 18px 20px;
    border: 1px solid #eee;
    box-sizing: border-box;
    border-radius: 12px;
    &>div{
      flex: 3;
    }
    .addImg{
      margin: 0 46px;
      width: 28px;
    }
    .coinImg{
      height: 60px;
      width: 60px;
      margin-right: 10px;
      border-radius: 60px;
      overflow: hidden;
      display: flex;
    }
    .tip{
      font-size: 24px;
      margin-top: -5px;
    }
  }
  .poolsInfo{
    border: 1px solid #eee;
    padding: 30px;
    margin: 20px 18px;
    border-radius: 12px;
    text-align: left;
    .item{
      margin-bottom: 10px;
      .subTitle{
        margin-bottom: 4px;
      }
    }
    .green_p{
      margin-left: 20px;
    }
    .iconImg{
      width: 30px;
      margin-left: 12px;
    }
  }
  .btnDiv{
    margin: 30px 20px;
    font-size: 32px;
    font-weight: 500;
    .btn{
      height: 88px;
      // background:rgba(7,215,155,1);
      background: #29D4B0;
      border-radius: 48px;
      color: #fff;
      &:active{
        background:rgba(2,198,152,1);
      }
    }
  }
  .linkTo{
    font-size: 27px;
    padding: 28px;
    margin: 0 20px;
    border-top: 1px solid #eee;
    &>span{
      flex: 1;
      height: 40px;
      &:last-child{
        border-left: 1px solid #E4E6E9;
      }
    }
    img{
      height: 36px;
      margin-right: 10px;
    }
    i{
      font-size: 28px;
    }
  }
}

.mkListDia{
  :deep(.el-dialog) {
    border-radius:12px 12px 0px 0px;
    position: relative;
    margin: auto;
    width: 690px;
    border-radius:12px;
    .el-dialog__body,
    .el-dialog__header{
      padding: 0;
    }
  }
}
.myDialog{
  :deep(.el-dialog) {
    position: relative;
    margin: auto;
    width: 570px;
    border-radius: 20px;
    .el-dialog__body,
    .el-dialog__header{
      padding: 0;
    }
  }
  &.apy{
    :deep(.el-dialog) {
      width: 620px;
    }
  }
}
</style>
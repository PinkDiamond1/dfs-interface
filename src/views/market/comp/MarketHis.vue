<template>
  <div class="history">
    <div class="title flexb">
      <span class="flexc">
        <span class="act" @click="showMarketList = true">{{ thisMarket.symbol0 }}-{{ thisMarket.symbol1 }}</span>
        <img class="iconImg" src="https://storied-crepe-e5e65c.netlify.app/dex/down.svg" alt="">
      </span>
    </div>
    <!-- <div class="tip noData">链上数据重新同步中。一天后恢复查询。</div> -->
    <div class="lists" >
      <van-list
        v-model="loadingMore"
        :finished="finished"
        :loading-text="$t('public.loading')"
        :finished-text="$t('public.noMore')"
        @load="handleCurrentChange"
      >
        <div class="list" v-for="(item, index) in pageList" :key="index"
          @click="handleToBrowser(item.trx_id)">
          <div class="flexa">
            <span class="tradeType green_p" v-if="item.isBuy">{{ $t('more.deposit') }}</span>
            <span class="tradeType red_p" v-else>{{ $t('more.withdraw') }}</span>
            <span class="flexc">
              <span>{{ thisMarket.symbol1 }}</span>/<span>{{ thisMarket.symbol0 }}</span>
            </span>
            <span class="tip time">（{{ item.time }}）</span>
          </div>
          <div class="flexb dataInfo tip">
            <div>
              <div>{{ $t('more.num') }}({{ item.sym0 }})</div>
              <div class="num">{{ item.num0 }}</div>
            </div>
            <div>
              <div>{{ $t('more.num') }}({{ item.sym1 }})</div>
              <div class="num">{{ item.num1 }}</div>
            </div>
          </div>
        </div>
      </van-list>
      <!-- <div class="noData tip" v-if="!hisList.length">{{ $t('public.noData') }}</div> -->
    </div>

    <van-popup
      class="newMarket"
      v-model="showMarketList"
      position="left">
      <market-list :marketLists="marketLists"
        @listenMarketChange="handleMarketChange"
        @listenClose="handleClose"/>
    </van-popup>
  </div>
</template>

<script>
import moment from 'moment';
import { mapState } from 'vuex';
import MarketList from '@/components/MarketArea';
import {toLocalTime, toBrowser} from '@/utils/public'
export default {
  name: 'tradeHistory',
  components: {
    MarketList,
  },
  computed: {
    ...mapState({
      account: state => state.app.account,
      marketLists: state => state.sys.marketLists,
    }),
    reward() {
      let t = this.totalSell - this.totalBuy;
      t = t.toFixed(4)
      if (t > 0) {
        t = `+${t}`
      }
      return t
    }
  },
  data() {
    return {
      loading: false,
      loadingMore: false,
      finished: false,
      exRate: false,
      showMarketList: false,
      hisList: [],
      pageList: [],
      thisMarket: {
        mid: 39,
        symbol0: 'EOS',
        symbol1: 'DFS',
      },
      // 统计
      totalBuy: 0,
      totalGetBuy: 0,
      totalSell: 0,
      totalGetSell: 0,
      pageSize: 20,
      page: 1,
    }
  },
  watch: {
    '$route': {
      handler: function rt() {
        this.page = 1;
        this.loadingMore = true;
        this.pageList = []
        this.handlerGetMarket()
      },
      deep: true,
      immediate: true,
    },
    marketLists: {
      handler: function rt(newVal, oldVal) {
        if (!newVal.length || (oldVal && oldVal.length > 0)) {
          return
        }
        this.handlerGetMarket()
      },
      deep: true,
      immediate: true,
    },
    account: {
      handler: function listen(newVal) {
        if (newVal.name) {
          this.handlerGetMarket();
        }
      },
      deep: true,
      immediate: true,
    }
  },
  methods: {
    handleToBrowser(id) {
      toBrowser(id, 'tx')
    },
    handleCurrentChange() {
      this.handlerGetMarket();
    },
    handleExchange(index) {
      this.$set(this.hisList[index], 'exRate', !this.hisList[index].exRate);
    },
    handlerGetMarket() {
      if (!this.marketLists.length || !this.account || !this.account.name || this.loading) {
        return
      }
      const mid = this.$route.params.mid || 39;
      this.thisMarket = this.marketLists.find(v => v.mid == mid) || {};
      this.handleGetHistory(mid)
    },
    async handleGetHistory(mid) {
      this.loading = true;
      const user = this.account.name;
      const params = {
        user,
        mid,
        page: this.page,
        limit: this.pageSize,
      }
      const {status, result} = await this.$api.depositlog(params);
      this.loading = false;
      if (!status) {
        this.hisList = [];
        this.pageList = [];
        // this.handleCurrentChange();
        return;
      }
      const list = result.data || [];
      list.forEach(v => {
        let t = toLocalTime(v.create_time).replace(/-/g, '/');
        t = moment(t).valueOf() + 8 * 3600 * 1000;
        const time = toLocalTime(t)
        this.$set(v, 'time', time);
        const isBuy = v.namex === 'depositlog';
        this.$set(v, 'isBuy', isBuy);
        const num0 = v.bal0.split(' ')[0].replace(/-/g, '');
        const num1 = v.bal1.split(' ')[0].replace(/-/g, '');
        const sym0 = v.bal0.split(' ')[1]
        const sym1 = v.bal1.split(' ')[1]
        this.$set(v, 'num0', num0);
        this.$set(v, 'num1', num1);
        this.$set(v, 'sym0', sym0);
        this.$set(v, 'sym1', sym1);
      })
      this.hisList = list;
      if (this.page === 1) {
        this.pageList = list;
      } else {
        this.pageList.push(...list)
      }
      this.loadingMore = false;
      this.page += 1;
      // 数据全部加载完成
      if (list.length < this.pageSize) {
        this.finished = true;
      }
    },
    handleClose() {
      this.showMarketList = false;
    },
    handleMarketChange(item) {
      this.thisMarket = item;
      this.handleClose();
      this.$router.push({
        name: 'MarketHis',
        params: {
          mid: item.mid
        }
      })
    },
  },
}
</script>

<style lang="scss" scoped>
.history{
  font-size: 27px;
  .title{
    font-size: 32px;
    text-align: left;
    margin: 0px 0 30px;
    padding: 0 0 0 40px;
    color: #000;
    &>span{
      margin-right: 60px;;
    }
    .rulesTip{
      font-size: 21px;
      margin-right: 40px;
      text-align: right;
      .tipIcon{
        margin-left: 8px;
      }
    }
    .act{
      color: $color-black;
      position: relative;
      cursor: pointer;
      &::before{
        content: '';
        position: absolute;
        width:60px;
        height:8px;
        background:rgba(2,198,152,1);
        border-radius:4px;
        bottom: -20px;
        left: 50%;
        transform: translateX(-45%);
      }
    }
  }
  .total{
    padding: 0 40px;
    font-size: 21px;
    line-height: 28px;
    // justify-content: flex-end;
    .totalNum>div{
      margin-left: 10px;
      &:first-child{
        margin-left: 0px;
      }
    }
  }
  .noData{
    padding: 50px 0;
    font-size: 30px;
  }
  .lists{
    .list{
      margin: 30px 40px;
      padding: 30px;
      border-radius: 15px;
      font-size: 30px;
      box-shadow: 0px 8px 40px 0px rgba(220,220,220,0.5);
      &>div{
        margin-top: 10px;
        &:first-child{
          margin-top: 0;
        }
      }
      .tradeType{
        font-size: 30px;
        font-weight: 500;
        margin-right: 10px;
      }
      .time{
        margin-left: -8px;
      }
      .dataInfo{
        font-size: 24px;
        text-align: left;
        &>div{
          flex: 1;
        }
        .num{
          font-size: 30px;
          color: #333;
        }
      }
    }
  }

  .iconImg{
    width: 30px;
    margin: 0 0 0 8px;
  }
}

.mkListDia{
  // animation: none;
  :deep(.el-dialog) {
    position: absolute;
    bottom: 0px;
    margin: 0px;
    width: 100%;
    border-radius:30px 30px 0px 0px;
    .el-dialog__body,
    .el-dialog__header{
      padding: 0;
    }
  }
  &.pcList{
    :deep(.el-dialog) {
      position: relative;
      margin: auto;
      width: 670px;
      border-radius:30px;
    }
  }
}

.pagination{
  text-align: right;
  margin-top: 20px;
  margin-bottom: 30px;
  font-size: 26px;
  padding-right: 30px;
  :deep(.el-pager) {
    li.active{
      color: #07D79B;
    }
    li:hover{
      color: #07D79B;
    }
    li{
      font-size: 26px;
    }
  }
  :deep(.btn-prev), :deep(.btn-next) {
    &:hover {
      color: #07D79B;
    }
    .el-icon-arrow-left, .el-icon-arrow-right{
      font-size: 26px;
    }
  }
}
</style>

<template>
  <div class="deposit">
    <img class="close" @click="handleClose(false)"
      src="https://storied-crepe-e5e65c.netlify.app/svg/sd_icon_btn.svg">
    <div class="title">{{ $t('financial.depositTitle') }}</div>
    <div class="content">
      <div>
        <div class="bal dinReg" @click="handleClickMax">{{ $t('public.balance') }}：{{ bal }} EOS</div>
        <div class="flexb">
          <div class="flexa coinInfo">
            <img class="coinImg" src="https://leafy-kataifi-c6d825.netlify.app/coin/eosio.token-eos.svg" alt="">
            <div>
              <div class="coinName">EOS</div>
              <div class="contract tip">eosio.token</div>
            </div>
          </div>
          <div class="iptDiv flexb">
            <van-field class="ipt dinBold" v-model="deposit" placeholder="0.0" />
            <span class="max" v-if="showMax" @click="handleClickMax">MAX</span>
          </div>
        </div>
      </div>
      <div class="flexb lockTime">
        <span class="tip">{{ $t('financial.depTime') }}</span>
        <span class="red">{{ $t('financial.depLock') }}</span>
      </div>
    </div>

    <!-- 按钮 -->
    <div class="btn flexc" :class="{'disabled': !regNum}"  @click="handleDeposit">
      <span v-if="regNum">{{ $t('public.confirm') }}</span>
      <span v-else>{{ $t(`financial.${regContent}`) }}</span>
    </div>
    <!-- 交易规则 -->
    <div class="rules">
      <TradeRules :showType="'dialog'"/>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex';
import { DApp } from '@/utils/wallet';
import { get_balance } from '@/utils/api'
import { toFixed } from '@/utils/public';

import TradeRules from '../comp/TradeRules';
export default {
  name: 'financialDeposit',
  components: {
    TradeRules
  },
  data() {
    return {
      deposit: '',
      bal: '0.0000',
      showMax: true,
      regContent: '',
      // 定时器
      balTimer: null,
    }
  },
  computed: {
    ...mapState({
      account: state => state.app.account,
    }),
    regNum() {
      if (!Number(this.deposit)) {
        this.regContent = 'iptNum'; // eslint-disable-line
        return false
      }
      if (Number(this.deposit) > Number(this.bal)) {
        this.regContent = 'banLow'; // eslint-disable-line
        return false
      }
      return true
    }
  },
  watch: {
    account: {
      handler: function acc(newVal) {
        if (newVal.name) {
          this.handleGetBal();
        }
      },
      deep: true,
      immediate: true
    },
    deposit(newVal) {
      if (Number(newVal) === Number(this.bal)) {
        this.showMax = false;
        return
      }
      this.showMax = true;
    },
  },
  methods: {
    handleClose(type) {
      this.$emit('listenClose', type)
    },
    // 点击MAX
    handleClickMax() {
      this.deposit = this.bal;
    },
    // 查询用户余额
    async handleGetBal() {
      clearTimeout(this.balTimer)
      this.balTimer = setTimeout(() => {
        this.handleGetBal()
      }, 5000);
      const formName = this.account.name;
      const params = {
        code: 'eosio.token',
        symbol: 'EOS',
        decimal: 4,
        account: formName,
      }
      const {status, result} = await get_balance(params);
      if (!status) {
        return
      }
      this.bal = result.split(' ')[0];
    },
    // 存入操作
    handleDeposit() {
      if (!this.regNum) {
        return
      }
      const params = {
        code: 'eosio.token',
        toAccount: 'yfcsteadyone',
        memo: `deposit`,
        quantity: `${toFixed(this.deposit, 4)} EOS`
      }
      DApp.transfer(params, (err) => {
        this.loading = false;
        if (err && err.code == 402) {
          return;
        }
        if (err) {
          this.$toast({
            type: 'fail',
            message: err.message,
          })
          return;
        }
        this.$toast({
          message: this.$t('public.success'),
          type: 'success'
        });
        this.handleClose(true)
      })
    }
  }
}
</script>

<style lang="scss" scoped>
@import '../css/dialog.scss';
</style>

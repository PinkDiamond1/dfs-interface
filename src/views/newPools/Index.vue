<template>
  <div class="pools">
    <div class="banner">
      <img width="100%" src="https://leafy-kataifi-c6d825.netlify.app/banner/mining.png">
    </div>
    <div class="main">
      <Claim :dfsTotal="dfsTotal" :tagTotal="tagTotal" :eosTotal="eosTotal"
        @listenUpdata="handleUpdate"/>

      <div class="tokenInfo">
        <TokenInfo :tokenName="'DFS'" />
        <TokenInfo :tokenName="'TAG'" />
        <!-- <TokenInfo :tokenName="'EOS'" /> -->
      </div>

      <div class="lists">
        <div class="title flexb">
          <span>{{ $t('mine.poolsList') }}</span>
          <!-- <span class="flexa">
            <span class="tip">挖矿规则</span>
            <img class="qus" src="https://storied-crepe-e5e65c.netlify.app/icon/tips_icon_btn.svg">
          </span> -->
        </div>
        <EosPools ref="EosPools" @totalClaim="handleEosClaim"/>
        <DfsPools ref="DfsPools" @totalClaim="handleDfsClaim"/>
        <TagPools ref="TagPools" @totalClaim="handleTagClaim"/>
      </div>
    </div>
  </div>
</template>

<script>
import Claim from '@/views/newPools/comp/Claim'
import TokenInfo from '@/views/newPools/comp/TokenInfo'
import DfsPools from '@/views/newPools/comp/DfsPools'
import TagPools from '@/views/newPools/comp/TagPools'
import EosPools from '@/views/newPools/comp/EosPools'
export default {
  name: 'newPools',
  components: {
    Claim, TokenInfo, DfsPools, TagPools, EosPools,
  },
  data() {
    return {
      dfsTotal: '0.0000',
      tagTotal: '0.0000',
      eosTotal: '0.0000',
    }
  },
  methods: {
    handleUpdate() {
      this.$refs.EosPools.handleGetBalPools()
      this.$refs.EosPools.handleGetLiqs()

      this.$refs.DfsPools.handleGetBalPools()
      this.$refs.DfsPools.handleGetLiqs()

      this.$refs.TagPools.handleGetBalPools()
      this.$refs.TagPools.handleGetLiqs()
    },
    handleDfsClaim(total) {
      this.dfsTotal = total + '';
    },
    handleTagClaim(total) {
      this.tagTotal = total + '';
    },
    handleEosClaim(total) {
      this.eosTotal = total + '';
    }
  }
}
</script>

<style lang="scss" scoped>
.pools{
  .banner{
    position: relative;
  }
  .main{
    z-index: 1;
    position: relative;
    padding: 40px;
    border-radius: 40px 40px 0px 0px;
    margin-top: -80px;
    background: #FFF;
    .lists{
      .title{
        font-size: 32px;
        padding-left: 20px;
        position: relative;
        margin-bottom: 30px;
        &::before{
          position: absolute;
          content: '';
          width: 4px;
          height: 30px;
          background: $color-main;
          border-radius: 4px;
          top: 50%;
          left: 2px;
          transform: translate(0, -50%);
        }
        .tip{
          font-size: 24px;
          margin-right: 6px;
        }
        .qus{
          width: 26px;
          margin-left: 10px;
        }
      }
    }
  }
}
</style>

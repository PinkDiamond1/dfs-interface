<template>
  <div class="setInfo">
    <div class="title flexb">
      <span class="back flexa" @click="$router.back()">
        <img src="https://storied-crepe-e5e65c.netlify.app/icon/back.png" alt="">
      </span>
      <span>{{ $t('my.editInfo') }}</span>
      <span class="back"></span>
    </div>

    <!-- 主要编辑内容 -->
    <div class="mainView">
      <div class="headImgItem" @click="handleShowImgCheck('head')">
        <img class="headImg" :src="avatar || 'https://leafy-kataifi-c6d825.netlify.app/coin/eosio.token-eos.svg'">
        <div class="checkBtn">
          <span class="checkHead">{{ $t('my.headImg') }}</span>
        </div>
      </div>
      <van-form>
        <div class="flexb item">
          <van-field
            v-model="nick"
            :name="$t('my.accName')"
            :label="$t('my.accName')"
            :placeholder="$t('my.accName')"
          />
          <img class="rightIcon" src="https://storied-crepe-e5e65c.netlify.app/icon/itemRight.png" alt="">
        </div>
        <div class="flexb item account">
          <van-field
            v-model="acc"
            disabled
            :name="$t('my.accAccount')"
            :label="$t('my.accAccount')"
          />
        </div>
        <div class="flexb item">
          <van-field
            readonly
            clickable
            :value="sex"
            name="picker"
            :label="$t('my.sex')"
            :placeholder="$t('my.sex')"
            @click="showPicker = true"
          />
          <img class="rightIcon" src="https://storied-crepe-e5e65c.netlify.app/icon/itemRight.png" alt="">
        </div>
        <div class="flexb item">
          <van-field
            v-model="desc"
            :name="$t('my.desc')"
            :label="$t('my.desc')"
            :placeholder="$t('my.desc')"
          />
          <img class="rightIcon" src="https://storied-crepe-e5e65c.netlify.app/icon/itemRight.png" alt="">
        </div>
        <div class="flexb item bgItem" @click="handleShowImgCheck('bg')">
          <div class="flexb">
            <span>{{ $t('my.bgImg') }}</span>
            <img class="bgImg" :src="cover || 'https://storied-crepe-e5e65c.netlify.app/accBanner/banner0.png'" alt="">
          </div>
          <img class="rightIcon" src="https://storied-crepe-e5e65c.netlify.app/icon/itemRight.png" alt="">
        </div>

        <div class="btnDiv">
          <div class="btn flexc" @click="handleSave">{{ $t('my.save') }}</div>
        </div>
      </van-form>
    </div>

    <van-popup v-model="showPicker" position="bottom">
      <van-picker
        show-toolbar
        :columns="columns"
        @confirm="onConfirm"
        @cancel="showPicker = false"
      />
    </van-popup>

    <el-dialog
      class="mydialog"
      :visible.sync="showCheckImg">
      <ImgCheck v-if="showCheckImg" :showType="type" @listenSure="handleSure"/>
    </el-dialog>
  </div>
</template>

<script>
import { mapState } from 'vuex';
import { DApp } from '@/utils/wallet';

import { get_acc_info } from '@/utils/api';
import ImgCheck from '../dialog/ImgCheck'

export default {
  name: 'setInfo',
  components: {
    ImgCheck,
  },
  data() {
    return {
      avatar: '', // 头像
      nick: '', // 昵称
      acc: '', // 账号
      sex: this.$t('my.mi'), // 性别
      desc: '', // 简介
      cover: '', // 背景

      columns: [this.$t('my.mi'), this.$t('my.man'), this.$t('my.woman')],
      showPicker: false,
      showCheckImg: false,
      type: '',
    }
  },
  mounted() {
  },
  computed: {
    ...mapState({
      account: state => state.app.account,
      accInfo: state => state.app.accInfo,
    }),
  },
  watch: {
    account: {
      handler: function sc (newVal) {
        if (newVal.name) {
          this.acc = newVal.name;
        }
      },
      deep: true,
      immediate: true
    },
    accInfo: {
      handler: function sc (newVal) {
        this.avatar = newVal.avatar
        this.cover = newVal.cover;
        this.desc = newVal.desc;
        this.nick = newVal.nick;
        if (newVal.sex === 1) {
          this.sex = this.$t('my.man')
        } else if (newVal.sex === 0) {
          this.sex = this.$t('my.woman')
        } else {
          this.sex = this.$t('my.mi')
        }
      },
      deep: true,
      immediate: true
    }
  },
  methods: {
    handleShowImgCheck(type) {
      this.type = type;
      this.showCheckImg = true;
    },
    onConfirm(value) {
      this.sex = value;
      this.showPicker = false;
    },
    handleSure(src) {
      this.showCheckImg = false;
      if (this.type === 'bg') {
        this.cover = src;
        return
      }
      this.avatar = src;
    },
    // 保存
    handleSave() {
      const formName = this.account.name;
      const permission = this.account.permissions;
      let sex = 2;
      if (this.sex === '男') {
        sex = 1
      } else if (this.sex === '女') {
        sex = 0
      }
      const params = {
        actions: [{
          account: 'dfscommunity',
          name: 'setprofiles',
          authorization: [{
            actor: formName, // 转账者
            permission,
          }],
          data: {
            user: formName,
            nick: this.nick,
            desc: this.desc,
            avatar: this.avatar,
            cover: this.cover,
            sex,
          }
        }]
      }
      DApp.toTransaction(params, (err) => {
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
        get_acc_info(this.acc)
        this.$router.replace({
          name: 'myCenter',
        })
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.setInfo{
  font-size: 32px;
  color: #333;
  .title{
    font-size: 36px;
    height: 70px;
    padding: 0 28px;
    font-weight: 500;
    .back{
      width: 50px;
      height: 50px;
      img{
        display: block;
        width: 18px;
      }
    }
  }
  .mainView{
    padding: 28px;
    .headImgItem{
      text-align: center;
      margin-bottom: 20px;
      .headImg{
        width: 204px;
        height: 204px;
        overflow: hidden;
        border-radius: 50%;
      }
      .checkBtn{
        margin-top: -20px;
      }
      .checkHead{
        background: #29D4B0;
        font-size: 26px;
        color: #fff;
        padding: 8px 30px;
        border-radius: 26px;
      }
    }
    .rightIcon{
      width: 10px;
      margin-left: 20px;
    }
    .item{
      color: #666;
      height: 100px;
      border-bottom: 1px solid rgba(#ebedf0, .4);
      &.bgItem{
        height: 178px;
        &>div{
          flex: 1;
        }
        .bgImg{
          width: 188px;
          height: 100px;
        }
      }
    }
    .btn{
      height: 90px;
      background: #29D4B0;
      border-radius: 50px;
      font-size: 32px;
      color: #FFF;
      margin-top: 72px;
      font-weight: 500;
    }
    :deep( .van-field--disabled){
      .van-field__label{
        color: #666;
      }
      .van-field__control{
        color: #999;
      }
    }
    :deep( .van-cell){
      height: 100%;
      font-size: 28px;
      padding-left: 0;
      padding-right: 0;
      padding: 0;
      display: flex;
      align-items: center;
      &::after{
        display: none;
      }
    }
    :deep( .van-field__control){
      text-align: right;
    }
  }
}
.mydialog{
  :deep(.el-dialog) {
    width: 700px;
  }
}
</style>

<template>
  <div class="update">
    <div class="title flexa">
      <img class="mail" src="https://leafy-kataifi-c6d825.netlify.app/icon/mail.png">
      <span>{{ $t('upRecord.title') }}</span>
    </div>
    <V6 />
    <V3 />
    <V2 />
    <V1 />
  </div>
</template>

<script>
import V1 from './comp/V1'
import V2 from './comp/V2'
import V3 from './comp/V3'
import V6 from './comp/V6'
export default {
  name: 'update',
  components: {
    V1, V2, V3, V6
  }
}
</script>

<style lang="scss" scoped>
.update{
  margin: 0 28px;
  padding: 28px 0;
  border-top: 1px solid $color-border;
  text-align: left;
  .title{
    color: #29D4B0;
    font-size: 32px;
    font-weight: 500;
    margin-bottom: 30px;
    .mail{
      width: 32px;
      margin-right: 24px;
    }
  }
}
</style>
<template>
  <div class="page-sd" :style="{ height: winHeight + 'px' }">
    <div class="inner">
      <h1>Stable Diffusion 绘画中心</h1>
      <h2>页面正在紧锣密鼓开发中，敬请期待！</h2>
    </div>
  </div>
</template>

<script setup>
import {ref} from "vue"

const winHeight = ref(window.innerHeight)
</script>

<style lang="stylus" scoped>
.page-sd {
  display: flex;
  justify-content: center;
  align-items center
  background-color: #282c34;

  .inner {
    text-align center

    h1 {
      color: #202020;
      font-size: 80px;
      font-weight: bold;
      letter-spacing: 0.1em;
      text-shadow: -1px -1px 1px #111111, 2px 2px 1px #363636;
    }

    h2 {
      color #ffffff;
      font-weight: bold;
    }
  }

}
</style>

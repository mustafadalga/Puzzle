<template>
  <div class="progress-bar">
    <div class="filled-area" :style="{'width':width+'%'}"></div>
    <div class="progressbar-percent">{{Math.round(width)}}%</div>
  </div>
</template>

<script>
export default {
  props:['width'],
  name: "ProgressBar"
}
</script>

<style scoped lang="scss">

</style>
<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>

export default {

}
// 根据页面宽度计算fontsize
document.addEventListener('DOMContentLoaded', () => {
  const html = document.querySelector('html')
  let fontSize = window.innerWidth / 10
  fontSize = fontSize > 50 ? 50 : fontSize
  html.style.fontSize = fontSize + 'px'
  console.log(html.style.fontSize)
})
</script>
<style lang="scss" scoped>
#app {
  width: 100%;
  height: 100%;
  background: #7d8188;
  overflow: hidden;
}
</style>

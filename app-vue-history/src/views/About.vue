<template>
  <div class="about">
    <h1>这是history的about</h1>
    <input type="text" name="" id="">
  </div>
</template>
<script>
export default {
  name: 'About',
  activated() {
    console.log('history about activated');
  },
  deactivated() {
    console.log('history about deactivated');
  },
  mounted() {
    window.a = 1;
    console.log(window.a,'app-vue-history');
  },
}
</script>
<style>
/* 测试全局css污染 */
h1{
  color: red;
}
</style>
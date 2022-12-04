<template>
  <div class="about">
    <h1>这是hash的about</h1>
    <input type="text" name="" id="">
  </div>
</template>

<script>
export default {
  name: 'About',
  activated() {
    console.log('hash about activated');
  },
  deactivated() {
    console.log('hash about deactivated');
  },
}
</script>
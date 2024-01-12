<!-- src/App.vue -->

<template>
  <div id="app">
    <Products />
  </div>
</template>

<script>
import Products from './components/Products.vue';

export default {
  components: {
    Products,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

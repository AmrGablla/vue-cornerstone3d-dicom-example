<template>
  <Viewer />
</template>

<script>
import Viewer from "./components/Viewer.vue";

export default {
  name: "App",
  components: {
    Viewer,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

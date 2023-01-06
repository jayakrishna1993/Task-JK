<template>
  <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
    <div class="container eli-text">
      <a class="navbar-brand">Eli Lilly - ABC & Co</a>
    </div>
  </nav>
  <Dashboard msg="Welcome to Assesment" />
</template>
<script>
import Dashboard from "./components/Dashboard.vue";

export default {
  name: "App",
  components: {
    Dashboard,
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
}

.bg-dark {
  background: linear-gradient(to left, #009833, #0076c8);
}
</style>

<template>
  <div id="app">
    <router-view />
  </div>
</template>

<script>
import "./assets/main.css";
export default {};
</script>

<style>
#nav {
  padding: 30px;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>

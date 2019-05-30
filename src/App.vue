<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link>
    </div>
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

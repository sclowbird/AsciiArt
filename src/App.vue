<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <HelloWorld msg="AsciiArt"/>
  </div>
</template>

<script>
import HelloWorld from "./components/HelloWorld.vue";

export default {
  name: "app",
  components: {
    HelloWorld
  }
};
</script>

<style>
@font-face {
  font-family: "Press Start 2P";
  font-style: normal;
  src: local("Press Start 2P"),
    url(https://fonts.googleapis.com/css?family=Press+Start+2p);
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin: auto;
  width: 50%;
  height: 50%;
  margin-top: 60px;
}
</style>

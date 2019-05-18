<template>
  <div id="app">
    <AsciiArt msg="AsciiArt"/>
  </div>
</template>

<script>
import AsciiArt from "./components/AsciiArt.vue";

export default {
  name: "app",
  components: {
    AsciiArt
  }
};
</script>

<style>
@font-face {
  font-family: "Press Start 2P";
  font-style: monospace;
  src: local("Press Start 2P"),
    url(https://fonts.googleapis.com/css?family=Press+Start+2p);
}

body {
  background-image: url("./assets/background.jpg");
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin: auto;
  height: 50%;
  width: 50%;
}
</style>

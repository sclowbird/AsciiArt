<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <input type="file" id="input" @change="loadTextFromFile">
    <img id="my-image">
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      pixelData: ""
    };
  },
  props: {
    msg: String
  },
  methods: {
    loadTextFromFile(e) {
      let files = e.target.files || e.dataTransfer.files;
      if (!files.length) return;
      this.createImage(files[0]);
    },
    createImage(file) {
      let fr = new FileReader();
      let eventResult;
      fr.onload = () => this.showImage(fr);
      fr.readAsDataURL(file);
    },

    showImage(fileReader) {
      let img = document.getElementById("my-image");
      img.onload = () => this.getImageData(img);
      img.src = fileReader.result;
    },

    getImageData(image) {
      let canvas = document.createElement("canvas");
      let ctx = canvas.getContext("2d");
      ctx.drawImage(image, 0, 0);
      let imageData = ctx.getImageData(0, 0, image.width, image.height).data;
      console.log(`Image size: ${image.width} x ${image.height}`);
      //    console.log(`Image loaded successfully. \n ${imageData}`);
      this.pixelData = imageData;
      let pixels = this.createPixelTupels();
    },

    createPixelTupels() {
      let pixelArray = Object.values(this.pixelData);
      let pixelTuples = [];
      let counter = 0;
      for (let i = 0; i < pixelArray.length; i++) {
        let tempArray = [];
        if (i != 0) {
          counter += 2;
        }
        for (let j = 0; j < 3; j++) {
          tempArray.push(pixelArray[i + j + counter]);
        }
        pixelTuples.push(tempArray);
      }
      console.log(pixelTuples);
      return pixelTuples;
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>

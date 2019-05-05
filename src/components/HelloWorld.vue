<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <input type="file" id="input" @change="loadImageData">
    <img id="my-image">
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {};
  },
  props: {
    msg: String
  },
  methods: {
    loadImageData(e) {
      let imageFile = e.target.files;
      if (!imageFile.length) return;
      this.createImage(imageFile[0]);
    },

    createImage(file) {
      let fr = new FileReader();
      // onload event gets triggered after content of readAsDataUrl is ready
      fr.onload = () => this.showImage(fr);
      fr.readAsDataURL(file);
    },

    showImage(fileReader) {
      let img = document.getElementById("my-image");
      img.onload = () => this.getImageData(img);
      // file reader result is necessary to display image from img source
      img.src = fileReader.result;
    },

    getImageData(image) {
      let canvas = document.createElement("canvas");
      let ctx = canvas.getContext("2d");
      ctx.drawImage(image, 0, 0);
      let imageData = ctx.getImageData(0, 0, image.width, image.height).data;
      //let imageDataWithoutAlpha = this.isNoAlpha(imageData);

      console.log(`Image size: ${image.width} x ${image.height}`);
      //console.log(`Image loaded successfully. \n ${imageData}`);
      this.createPixelTupels(imageData);
    },

    createPixelTupels(imageData) {
      let pixelArray = Object.values(imageData);
      let pixelTuples = [];
      let counter = 0;
      for (let i = 0; i < pixelArray.length; i++) {
        let tempArray = [];
        if (i != 0) {
          counter += 3;
        }
        for (let j = 0; j < 4; j++) {
          tempArray.push(pixelArray[i + j + counter]);
        }
        pixelTuples.push(tempArray);
      }
      this.removeAlphaChannel(pixelTuples);
    },

    removeAlphaChannel(pixelTuples) {
      console.log(pixelTuples.length);
      console.log(pixelTuples[0].length);
      for (let i = 0; i < pixelTuples.length; i++) {
        pixelTuples[i].splice(3, 1);
      }
      console.log(pixelTuples);
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

<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <input type="file" id="input" @change="loadImageData">
    <img hidden id="my-image">
    <div id="ascii" v-for="(item, index) in asciiObj" :key="index">
      <p class="overflow-visible">{{ item.row }}</p>
    </div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      imageWidth: 0,
      imageHeight: 0,
      resizeFactor: 0,
      asciiString: "",
      asciiObj: [{ row: " " }]
    };
  },
  props: {
    msg: String
  },
  methods: {
    loadImageData(e) {
      let imageFile = e.target.files;
      if (!imageFile.length) return;
      this.readImageData(imageFile[0]);
    },

    readImageData(imageFile) {
      let fr = new FileReader();
      // onload event gets triggered after content of readAsDataUrl is ready
      fr.onload = () => this.getImageDataResult(fr);
      fr.readAsDataURL(imageFile);
    },

    getImageDataResult(fileReader) {
      let img = document.getElementById("my-image");
      img.onload = () => this.processImage(img);
      img.src = fileReader.result;
    },

    processImage(img) {
      let ctx = this.getCanvasContext();
      this.drawImageToCanvas(img, ctx);
      let imageColors = this.getImageColors(img, ctx);
      let pixelTuples = this.createPixelTuples(imageColors);
      let pixelsWithoutAlpha = this.removeAlphaChannel(pixelTuples);
      let pixelBrightness = this.convertPixelToBrightness(pixelsWithoutAlpha);
      let asciiString = this.mapBrigthnessToAscii(pixelBrightness);
      this.addLineBreaks(asciiString);

      //can.style.border = "blue";
    },

    getCanvasContext() {
      let canvas = this.createCanvas();
      let ctx = canvas.getContext("2d");
      return ctx;
    },

    createCanvas() {
      let canvas = document.createElement("canvas");
      return canvas;
    },

    drawImageToCanvas(img, ctx) {
      const maximumWidth = 100;
      const originalPixelSize = img.height * img.width;

      this.imageWidth = img.width;
      this.imageHeight = img.height;

      let scalingFactor = maximumWidth / this.imageWidth;

      //correct resizefactor so that width and height is no float number
      this.imageWidth = Math.round(this.imageWidth * scalingFactor);
      this.imageHeight = Math.round(this.imageHeight * scalingFactor);

      ctx.drawImage(
        img,
        0,
        0,
        img.width,
        img.height,
        0,
        0,
        this.imageWidth,
        this.imageHeight
      );
    },

    getImageColors(img, ctx) {
      let imgColors = ctx.getImageData(0, 0, this.imageWidth, this.imageHeight)
        .data;

      this.imageWidth = Math.floor(this.imageWidth * 3);

      let imgColorArray = Object.values(imgColors);
      return imgColorArray;
    },

    createPixelTuples(imgColorArray) {
      let pixelTuples = [];
      let counter = 0;
      for (let i = 0; i < imgColorArray.length / 4; i++) {
        let tempArray = [];
        if (i != 0) {
          counter += 3;
        }
        for (let j = 0; j < 4; j++) {
          tempArray.push(imgColorArray[i + j + counter]);
        }
        pixelTuples.push(tempArray);
      }
      return pixelTuples;
    },

    removeAlphaChannel(pixelTuples) {
      for (let i = 0; i < pixelTuples.length; i++) {
        pixelTuples[i].splice(3, 1);
      }
      return pixelTuples;
    },

    convertPixelToBrightness(pixelTuples) {
      for (let i = 0; i < pixelTuples.length; i++) {
        let rgbAverage = 0;
        for (let j = 0; j < 1; j++) {
          rgbAverage += pixelTuples[i][0] * pixelTuples[i][0] * 0.241;
          rgbAverage += pixelTuples[i][1] * pixelTuples[i][1] * 0.691;
          rgbAverage += pixelTuples[i][2] * pixelTuples[i][2] * 0.068;
        }

        rgbAverage = Math.sqrt(rgbAverage);

        pixelTuples[i] = rgbAverage;
      }
      return pixelTuples;
    },

    createAsciiCharacters() {
      let asciiChars =
        //'.`^",:;Il!i~+_-?][}{1)(|\\/tfjrxnuvczXYUJCLQ0OZmwqpdbkhao*#MW&8%B@$';
        '$@B%8&WM#*oahkbdpqwmZO0QLCJUYXzcvunxrjft/|()1{}[]?-_+~i!lI;:,"^`.';
      return asciiChars;
    },

    mapBrigthnessToAscii(pixelBrightness) {
      const maxBrigthness = 255;
      const conversionFactor = 4;
      let asciiToString = "";
      let asciiChars = this.createAsciiCharacters();
      let counter = 0;
      for (let i = 0; i < pixelBrightness.length; i++) {
        let correspondingAsciiChar = Math.round(
          pixelBrightness[i] / conversionFactor
        );

        pixelBrightness[i] = asciiChars[correspondingAsciiChar];
        for (let j = 0; j < 3; j++) {
          asciiToString += pixelBrightness[i];
        }
      }
      return asciiToString;
    },

    addLineBreaks(asciiString) {
      let rowStrings = "";
      for (let i = 1; i < asciiString.length + 1; i++) {
        rowStrings += asciiString[i - 1];
        if (i % this.imageWidth == 0) {
          this.asciiObj.push({ row: rowStrings });
          rowStrings = "";
        }
      }
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
/*h3 {
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
*/
input {
  margin-bottom: 30px;
}

#ascii {
  min-width: 1200px;
  width: 1200px;
  resize: none;
}

.overflow-visible {
  font-family: "Press Start 2P", monospace;
  color: black;
  font-size: 3px;
  font-weight: bolder;
}
</style>

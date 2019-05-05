<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <input type="file" id="input" @change="loadImageData">
    <img id="my-image">
    <div id="test">
      <div id="ascii" v-for="(item, index) in asciiObj" :key="index">
        <p class="overflow-visible">{{ item.row }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      imageWidth: 0,
      asciiString: "",
      asciiObj: [{ row: "Placeholder" }]
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
      ctx.drawImage(img, 0, 0);
    },

    getImageColors(img, ctx) {
      let imgColors = ctx.getImageData(0, 0, img.width, img.height).data;
      this.imageWidth = img.width * 3;
      console.log(
        `Width: ${img.width}, Height: ${img.height}, pixel: ${img.width *
          img.height}`
      );

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
      console.log(pixelTuples[0]);
      for (let i = 0; i < pixelTuples.length; i++) {
        let rgbAverage = 0;
        for (let j = 0; j < pixelTuples[i].length; j++) {
          rgbAverage += pixelTuples[i][j];
        }
        rgbAverage = Math.round(rgbAverage / 3);
        pixelTuples[i] = rgbAverage;
      }
      return pixelTuples;
    },

    createAsciiCharacters() {
      let asciiChars =
        '`^",:;Il!i~+_-?][}{1)(|\\/tfjrxnuvczXYUJCLQ0OZmwqpdbkhao*#MW&8%B@$';
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
        asciiToString += pixelBrightness[i];
        asciiToString += pixelBrightness[i];
        asciiToString += pixelBrightness[i];
      }
      return asciiToString;
    },

    addLineBreaks(asciiString) {
      console.log(`Ascii String Length: ${asciiString.length}`);
      let rowStrings = "";
      for (let i = 1; i < asciiString.length + 1; i++) {
        rowStrings += asciiString[i - 1];
        if (i % this.imageWidth == 0) {
          console.log(`${rowStrings}`);
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

#test {
}

#ascii {
  text-align: justify;
  text-justify: inter-character;
}

.overflow-visible {
  font-family: "Courier New";
  font-size: 3px;
  font-weight: bold;
}
</style>
